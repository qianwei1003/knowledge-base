---
来源: CSDN博客
领域: AI编程规范
关键词: AI Coding Agent, Agent Loop, MCP协议, 工具系统, 安全模型, Context Window, Claude Code, Amazon Q
质量: ⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://blog.csdn.net/Android_XG/article/details/159791110
---

# AI Coding Agent 核心设计拆解：Agent Loop、安全模型与MCP协议

> 基于 Amazon Q Developer CLI（Rust 实现）和 Claude Code（TypeScript + Python）两个开源项目源码分析

## 1. AI Coding ≠ 聊天机器人

聊天机器人只能「说」，AI Coding Agent 能「做」。区别在于 **Tool Use（工具调用）**。

当让 Claude Code 修改 bug 时，背后流程：

```
👤 用户：「帮我修一下 src/app.js 里的 bug」
⬇️
🤖 LLM 思考：我需要先看看这个文件
📤 LLM 输出工具调用：fs_read("src/app.js")
⬇️
⚙️ Agent 执行：读取文件内容 → 返回给 LLM
⬇️
🤖 LLM 思考：找到 bug 了，第 42 行有问题
📤 LLM 输出工具调用：fs_write("src/app.js", ...)
⬇️
⚙️ Agent 执行：修改文件 → 返回结果
⬇️
🤖 LLM：「搞定了，问题出在…」
```

这个过程叫 **Agent Loop（智能体循环）**，是所有 AI Coding 工具最核心的设计模式。

> 关键概念：LLM 不直接操作系统，通过结构化的 JSON 请求告诉 Agent「我想做什么」，Agent 验证权限后代为执行。所有操作可审计、可拦截、可回滚。

## 2. Agent Loop：一个精巧的状态机

Amazon Q CLI 在 Rust 里实现了显式的有限状态机：

```
6 种状态：Idle → ExecutingRequest → ExecutingHooks → WaitingForApproval → ExecutingTools → 循环
```

精妙之处：

- **🔄 循环是自动的** —— LLM 调用工具后，结果自动注入对话并再次调用 LLM，直到 LLM 决定停下来
- **⚡ 工具可以并行执行** —— LLM 一次可返回多个 tool_use，Agent 用 Tokio 的 FuturesUnordered 并行执行
- **🛡️ 每次工具调用都过权限检查** —— Allow 直接执行，Ask 弹出确认，Deny 直接拒绝

## 3. 工具系统：AI 的「手和脚」

AI Coding Agent 的能力上限完全取决于它有哪些工具。

### 工具描述的艺术

工具的 description 不是给人看的文档，而是给 LLM 看的**行为指令**。它的质量直接决定 Agent 的表现。

> 这是 Prompt Engineering 最被低估的领域：工具描述的措辞差异，可以让 Agent 行为从「靠谱」变成「灾难」。一个写得好的 description 比调整模型温度有用 10 倍。

### tool_use_purpose：让 AI 「三思后行」

Amazon Q CLI 的精妙设计——每次工具调用都强制 LLM 填写 purpose 字段：

1. 用户能看到 AI 为什么做这个操作
2. 迫使 LLM「思考」后再行动，减少无意义的调用
3. 所有操作都有审计记录

## 4. 安全模型：四层纵深防御

Amazon Q CLI 实现了四层安全架构：

```
Hook → 用户确认 → 路径权限 → 工具白名单
```

- **路径权限**：所有路径先做 `canonicalize` 规范化处理，防止 `../` 路径穿越攻击
- **Claude Code** 用 Hook 实现声明式安全策略，自动检测 9 种常见安全风险：命令注入、XSS、eval() 滥用、SQL 注入、硬编码凭证等

## 5. Context Window 管理：最核心的稀缺资源

传统软件瓶颈是 CPU、内存、IO。AI Coding 的瓶颈是 **Context Window**——LLM 一次能处理的 token 上限。

五层 Prompt 架构：
```
System Prompt → Context Entries → Tool Specs → 对话历史 → 用户消息
```

Amazon Q CLI 四个管理策略：

1. **自动压缩（Compact）** —— 上下文溢出时，调用 LLM 对历史对话生成摘要。200K tokens → 2K tokens 摘要 + 最近 20 条消息
2. **消息截断** —— 读取大文件时只保留前 10000 字符
3. **历史裁剪** —— 保留最近消息，删除最早的，但维护结构完整性（不破坏 tool_use/tool_result 配对）
4. **资源文件限制** —— 自动包含的资源文件不超过 10KB

> 终端提示符实时显示 Context Window 使用率：42% 正常，85% 需要压缩

## 6. MCP 协议：工具扩展的事实标准

MCP（Model Context Protocol）正在成为 AI Coding 领域的事实标准。

简单说，MCP 就是 AI Agent 调用外部工具的「USB 接口」：

```
AI Agent → JSON-RPC → MCP Server → 工具执行
```

Amazon Q CLI 用 **Actor 模型**管理多个 MCP Server——每个 Server 是独立 Actor，支持独立的连接管理、错误恢复和工具发现。

## 7. Claude Code 插件体系：五维扩展

5 种正交的扩展点，每种解决不同需求。最有意思的是 **feature-dev 插件的 7 阶段工作流**，Phase 2 并行启动多个子 Agent 去探索不同的代码路径——这就是 Multi-Agent 协作在实际产品中的落地。

## 8. 两个项目架构对比

| 维度 | Amazon Q CLI | Claude Code |
|------|-------------|-------------|
| 语言 | Rust（系统级性能） | TypeScript + Python |
| 架构 | 单体 Agent + Actor 并发 | 核心引擎 + 插件生态 |
| 并发 | Tokio async + Actor | Node.js 事件循环 + 子进程 |
| 扩展 | MCP + Hook 脚本 | 5 维插件体系 |
| 状态 | SQLite 持久化 | 文件系统 + 会话状态 |
| 优势 | 性能、类型安全、编译时保证 | 开发效率、生态丰富、灵活性 |

核心范式完全一致：**LLM Agent + Tool Use + Streaming + Safety + MCP**

## 9. 七大设计原则总结

1. **LLM 是大脑，工具是手脚** —— LLM 不直接操作系统，通过结构化 Tool Use 间接操作，所有操作可审计、可拦截、可回滚
2. **流式处理优先** —— 不等完整响应，增量解析 + 实时渲染
3. **安全是架构级关注点** —— 权限不是事后补丁，而是从工具定义到用户确认的完整链路
4. **Context Window 是稀缺资源** —— 所有设计围绕「如何在有限窗口内塞入最有价值的信息」
5. **工具描述即 Prompt** —— 工具的 description 直接决定 Agent 选择和使用工具的方式
6. **MCP 标准化工具生态** —— 统一的工具接口协议，支持跨 Agent 复用
7. **状态机驱动对话管理** —— 不是简单的一问一答，而是带有明确状态转换的有限状态机

## 实践启示

- Agent「失忆」→ Context Window 满了
- Agent 不用最好的方法 → 工具描述引导不够好
- 不同工具「手感」差很多 → 状态机和错误处理的精细度不同
- MCP 如此重要 → 它是工具生态的 USB 接口
