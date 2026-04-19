---
来源: 博客/腾讯新闻
领域: AI编程
关键词: Claude Code, 源码架构, Prompt Cache, Agent Fork, Swarm并发, Dream记忆, AI Agent工程架构
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://new.qq.com/rain/a/20260331A07EUY00
---

# 从 Claude Code 泄漏源码看第一梯队 AI Agent 工程架构

2026 年 3 月 31 日，Anthropic 因打包流程失误，将 Claude Code（v2.1.88）的完整前端与客户端源码暴露在 npm 仓库中。约 1900 个文件、超过 51 万行的原生 TypeScript 代码。这份源码是生产级 AI 编程助手架构的极佳教材。

---

## PART.01 不仅仅是一个 CLI 工具

从目录结构（`src/` 下约 40 个一级模块）可以看出，Claude Code 的复杂度远超常规单体 Agent。

**技术栈选型：**
- 语言：TypeScript
- 运行时：Bun（性能更激进）
- CLI 框架：Commander
- 终端渲染层：React + Ink

**为什么 CLI 工具用 React？**

源码中的 `screens/REPL.tsx`（高达 5005 行）给出了答案。在大模型流式输出（Streaming）和多工具并发执行的场景下，终端 UI 的状态管理极其复杂（同时渲染思考过程、工具调用进度条、代码 Diff 预览等）。采用声明式的 React 配合极简的 Zustand 风格自定义 Store（`state/store.ts`），是应对高频局部刷新的最佳工程实践。

**两种运行模式：**
- **交互式 REPL 模式**：通过 Ink 驱动前端终端 UI，面向人类开发者
- **无头/SDK 模式**（QueryEngine 类）：完全剥离 UI，支持 JSON 流式输出，为嵌入 IDE 或 CI/CD 流程埋下伏笔

**启动优化：** 配置读取（MDM Settings）和 Keychain 密钥预取等 I/O 密集型操作放在子进程中，与主模块 ~135ms 的加载过程并行执行。

---

## PART.02 Prompt Cache（提示词缓存）工程学

这是整份源码中最具技术含量的部分，也是拉开 Claude Code 与普通套壳应用体验差距的核心壁垒。

为了最大化缓存命中率，Claude Code 设计了严密的**分段缓存架构**：

1. **静态段（全局可缓存）**：通过 `systemPromptSection()` 生成，包含模型身份介绍、系统级安全规则、代码风格限制、工具使用基础指南等。在整个会话生命周期内几乎不变。

2. **动态分界线**：源码中硬编码特殊标记 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY`。

3. **动态段（会话级缓存/不缓存）**：包含当前工作目录信息（CWD）、Git 状态、MCP 指令、用户配置等高频变化的数据。

**防止缓存穿透的兜底工作：**
- **确定性排序**：传给大模型的工具描述严格按照内置工具前缀 + MCP 工具后缀进行字母表排序
- **哈希路径映射**：配置文件路径使用基于内容的哈希值，避免每次注入路径不同破坏缓存
- **状态外置**：当前可用的 Agent 列表从工具描述中剥离，转移到消息附件（Attachments）中。仅此一项改动就减少了约 10.2% 的 Cache Creation Tokens 消耗

**核心洞察：** 现阶段优秀的 AI 应用层开发，本质上就是在贪婪且精细地压榨 API 缓存系统的价值。

---

## PART.03 Tools 与流式并发执行

Claude Code 内置超过 40 种工具，采用高度模块化的工厂模式（Factory Pattern）。每个工具继承自基础 `Tool` 接口，必须实现 `checkPermissions()`、`validateInput()` 和 `isConcurrencySafe()` 等方法。

**按需加载的 ToolSearch 机制：**

非核心工具被标记为 `defer_loading: true`。模型在当前 Prompt 中看不到这些工具的具体定义，只知道有一个 `ToolSearch` 工具。当模型认为自己需要额外能力时，必须先调用 `ToolSearch` 去动态加载对应的工具配置。

**StreamingToolExecutor（流式工具执行器）：**

协调器（`toolOrchestration.ts`）将大模型返回的工具调用请求分区为并发批次和串行批次：
- 并发安全的工具（如同时读取多个不相关的文件）被并行触发
- 非并发安全的工具（如先后修改同一个代码文件）严格串行
- 大结果集的工具设有 `maxResultSizeChars` 预算，超过预算的内容被截断并持久化到本地临时文件中，只给 LLM 返回预览摘要

---

## PART.04 解决上下文污染的 Fork 机制

单体 Agent 的致命缺陷：执行复杂任务时，模型可能反复读取错误文件、尝试错误命令，产生大量垃圾上下文，迅速污染主对话。

**协调器模式（Coordinator Mode）+ Fork Subagent 机制：**

- **Coordinator（协调者）**：被剥夺了直接操作文件的权限，只保留 `Agent`（派生子代理）、`SendMessage` 和 `TaskStop` 三个工具。唯一工作是规划工作流（Research → Synthesis → Implementation → Verification）
- **Workers（执行者）**：携带具体的工具描述被派生出来

**Fork 继承机制：**

Coordinator Fork 出 Explore Agent 时，子 Agent 继承父对话的缓存（共享 Prompt Cache 以节约成本），但后续探索动作完全在其隔离的上下文中进行。探索结束后，子 Agent 只需要通过 XML 格式 `<task-notification>` 将提炼好的关键结论传回 Coordinator 的主上下文。

**设计精髓：** 用完即毁，只留结论——这是目前业界处理复杂多 Agent 长文本协同的最佳实践之一。

---

## PART.05 Agent Swarm 并发机制

源码还展示了更具野心的并发多 Agent 架构——**Swarm（Teammate）集群**。

系统支持 `in_process_teammate` 任务类型，主进程可以平行唤醒多个 Agent 同时执行不同的任务。

**终端 CLI 环境的工程挑战与解法：**
- **Leader 权限桥接**（`permissionSync.ts`）：所有 Teammate 子进程不允许直接弹窗请求权限，通过内部通道桥接给主进程的 Leader Agent，由 Leader 统一进行安全拦截和用户确认
- **终端布局自动化**：源码集成了 iTerm2 和 Terminal.app 的 AppleScript 控制指令，派生新的 Teammate 时自动在终端中切分窗格（Split Pane），为每个子 Agent 分配独立的输出视窗

**标志着 AI 从"单体思考"正式向"集群并发协作"演进。**

---

## PART.06 Dream（梦境）记忆架构

Claude Code 的记忆系统（`memdir/` 模块）**完全基于本地文件系统**，没有使用向量数据库。

**架构组成：**
- 核心的 `MEMORY.md`（限制在最多 200 行/25KB 以内）作为高层索引
- 多个基于 Frontmatter 格式的主题文件
- 记忆被划分为 User、Feedback、Project、Reference 四大类

**KAIROS 助手模式（未正式发布）：**

在 KAIROS 模式下，记忆系统采用类似人类日志的追加模式（写入 `logs/YYYY/MM/YYYY-MM-DD.md`）。到夜间或闲置时间，后台唤醒名为 **Dream（做梦）** 的离线任务 Agent，对白天的流水账日志进行总结、蒸馏，然后提取固化到结构化的长期主题文件中。

**核心意义：** 从短期日志到长期记忆的异步整合机制，绕开了向量检索的召回率痛点，代表了端侧 AI 助理向永远在线、持续学习演进的明确方向。

---

## PART.07 权限收敛与安全

Claude Code 采用多层权限收敛架构：从底层沙箱到特定危险操作的硬编码拦截，再到工具级别的校验。

**最引人注目：Auto Mode Classifier（`yoloClassifier.ts`）**

当用户开启自动模式时，系统使用**侧查询（Side Query）机制**：
- 在后台静默调用一个更小、更便宜的 LLM
- 将当前对话的精简转录和即将执行的 Bash 命令抛给它
- 由侧边模型输出 Allow 或 Deny 的决策
- 内部还有基于阈值的 Denial Tracking，当自动工具被频繁拒绝时，系统优雅降级，退回到 Prompting 模式请求人类介入

**用小 AI 监管大 AI 的动态权限系统，比传统静态拦截规则灵活得多。**

---

## PART.08 我们能学到什么？

从 Claude Code 的底层设计可以看出，大模型应用层创业，单纯依靠拼凑 Prompt、堆砌向量数据库、套一个简单循环外壳的时代已经结束了。

**真正的壁垒建立在对以下方面的极致追求：**
1. Token 成本的极致抠门（Prompt Cache 优化）
2. 多状态机协同的流式调度（Coordinator 与 Fork 机制）
3. 用户意图容错与安全干预的平衡（YOLO Classifier）
4. 对宿主操作系统深度的文件流集成

Claude Code 展示出的工程化水平，已经为 2026 年的 AI 助理产品树立了全新的技术标杆。
