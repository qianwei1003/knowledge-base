---
来源: GitHub/CSDN深度解读
领域: AI编程
关键词: superpowers, AI Coding Agent, 工程化工作流, TDD, 系统调试, 代码审查, 子代理协作, skill系统, Claude Code, Cursor
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://github.com/obra/superpowers
---

# Superpowers — AI Coding Agent 工程化工作流系统

> 摘要：superpowers 不是一个普通的 Prompt 仓库，而是一套面向 AI Coding Agent 的工程化工作流系统。它通过 brainstorming、planning、TDD、debugging、review、verification 和多代理协作，把 AI 从"会写代码"进一步约束为"按流程做开发"的协作伙伴。

项目地址：https://github.com/obra/superpowers

## 一、项目定位

Superpowers is a complete software development workflow for your coding agents.

它不是单点能力增强，而是一套完整的软件开发工作流。

### 1. 服务对象是 Coding Agent

面向 Claude Code、Cursor、Codex、OpenCode、Gemini CLI 等 AI 编码平台，给这些"会写代码的 AI 代理"提供一套可调用的技能系统。

### 2. 核心是技能与规则

仓库最重要的目录：
- `skills/` — 结构化 Markdown 技能文件
- `agents/` — 角色定义
- `commands/` — 快捷命令入口
- `hooks/` — 会话启动钩子

每个 skill 包含：技能名、触发条件、使用场景、核心原则、执行流程、红线与反模式、示例与注意事项。

## 二、解决的核心问题

AI Coding Agent 的典型问题：
1. **一上来就写代码** — 还没搞清需求就直接实现
2. **调 bug 时喜欢拍脑袋** — 没有真正定位根因
3. **跳过测试与验证** — 改完代码就说"应该可以了"
4. **没有计划意识** — 复杂需求容易中途跑偏
5. **多代理协作缺乏纪律** — 角色混乱、上下文污染

## 三、完整工作流

```
会话开始
 ↓
using-superpowers：先判断该用哪些 skill
 ↓
需求澄清与设计
 ├─ brainstorming：理解上下文、提问、比较方案、形成 spec
 ↓
准备实施
 ├─ using-git-worktrees：创建隔离工作区，验证基线
 ├─ writing-plans：把 spec 写成详细实施计划
 ↓
选择执行方式
 ├─ subagent-driven-development / executing-plans
 ↓
实现中的纪律约束
 ├─ test-driven-development：先测后写
 ├─ systematic-debugging：遇到问题先查根因
 ├─ requesting-code-review：完成后请求审查
 ├─ receiving-code-review：理性处理反馈
 ├─ dispatching-parallel-agents：独立问题可并行派发
 └─ verification-before-completion：没有验证就不能宣称完成
 ↓
收尾阶段
 └─ finishing-a-development-branch：测试、合并、PR
 ↓
能力演进
 └─ writing-skills：把新的有效方法沉淀成 skill
```

## 四、核心 Skill 解读

### 1. using-superpowers — 总控开关

核心要求：
- 只要有 1% 的可能某个 skill 适用，就必须先检查并调用
- 禁止靠"直觉"直接进入执行状态
- 压制 AI 最典型的坏习惯：先动手再说

### 2. brainstorming — 先设计再实现

明确要求：
- 先探索项目上下文
- 一次只问一个问题
- 理解目标、约束和成功标准
- 提供 2~3 个方案与权衡
- 最终输出 spec 文档

### 3. writing-plans — 把 spec 写成可执行施工图

要求产出极度具体的实施计划：
- 每个任务对应哪些文件
- 每一步的代码长什么样
- 运行什么命令验证
- 每个动作要小到 2~5 分钟级别

### 4. test-driven-development — 铁律

**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST**

- 先写了实现代码，就删掉重来
- 没有亲眼看到测试失败，就不算完成

### 5. systematic-debugging — 先查根因

**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST**

四阶段：根因调查 → 模式分析 → 假设与验证 → 实施修复

### 6. verification-before-completion — 证据先于结论

**NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE**

不能说"应该可以了""已经修复"——必须跑验证命令、查看输出和退出码。

### 7. receiving-code-review — 反对表演式认同

收到 review 后：
- 先完整阅读，先理解反馈
- 如果 reviewer 对，就修改
- 如果 reviewer 不对，要做技术性反驳
- 明确反对"你说得对""非常好的建议"这类情绪化表达

### 8. dispatching-parallel-agents — 独立问题才并发

判断标准：
- 问题是否独立
- 是否存在共享状态
- 是否会互相影响

## 五、Skill 设计哲学

1. **description 只描述"何时使用"** — 不能把完整流程写在 description 里，否则模型可能只看 description 不读正文
2. **skill 是"可复用行为模块"** — 是工程规范 + 行为控制器 + 流程模板 + 角色指令集
3. **写 skill 也应遵守 TDD** — 先设计压力场景，看没有 skill 时模型怎么失败，再写 skill 去修复
4. **Prompt/Skill 当成"软件资产"** — 需要测试、维护、性能优化（token）、检索优化

## 六、跨平台适配

支持的平台配置文件：
- `.claude-plugin/plugin.json`
- `.cursor-plugin/plugin.json`
- `gemini-extension.json`
- `.codex/INSTALL.md`
- `.opencode/INSTALL.md`

## 七、关键启示

1. **AI 最大的问题不是不会写代码，而是缺少稳定的工程纪律**
2. **不要期待模型天然具备工程纪律，要把纪律写成规则、技能和流程**
3. **Prompt Engineering 不仅是"怎么让模型更会答题"，更是"怎么让模型进入工程团队的工作方式"**
4. **完成实现 ≠ 完成研发流程** — 还要决定怎么集成、是否开 PR、是否保留现场
