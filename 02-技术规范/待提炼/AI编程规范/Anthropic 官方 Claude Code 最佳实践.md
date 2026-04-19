---
来源: Anthropic官方博客/GitHub
领域: AI编程
关键词: Claude Code,AI编程,最佳实践,CLAUDE.md,智能体编程,MCP,子代理,工作流,prompt工程
质量: ⭐⭐⭐⭐⭐
提炼状态: 初提炼完成
提炼日期: 2026-04-19
提炼人: Claude
原始链接: https://www.anthropic.com/engineering/claude-code-best-practices
---

# Anthropic 官方 Claude Code 最佳实践

> 来源：Anthropic 官方博客（2025年4月发布）
> 原文：https://www.anthropic.com/engineering/claude-code-best-practices
> 中文翻译：GitHub coderPerseus/blog Issue #63

---

## 核心要点速查

| 类别 | 规则 | 来源 |
|------|------|------|
| **CLAUDE.md** | 记录常用命令、代码风格、工作流，用 `#` 键迭代优化 | 章节1.a, 1.b |
| **CLAUDE.md 位置** | 仓库根目录/父目录/子目录/全局 `~/.claude/CLAUDE.md` | 章节1.a |
| **指令优化** | 使用 "IMPORTANT" 或 "YOU MUST" 强调关键指令 | 章节1.b |
| **工具管理** | 使用 `/allowed-tools` 或 `--allowedTools` 管理权限 | 章节1.c |
| **工作流程** | 探索→计划(think)→编码→提交，先计划后编码 | 章节3.a |
| **TDD** | 先写测试→运行失败→提交测试→写代码通过测试 | 章节3.b |
| **视觉迭代** | 提供截图→Claude实现→截图反馈→2-3轮迭代 | 章节3.c |
| **GitHub集成** | 安装 gh CLI，Claude 可创建PR、修复审查评论 | 章节1.d, 3.g |
| **MCP扩展** | 项目配置/全局配置/`.mcp.json` 三种连接方式 | 章节2.b |
| **自定义命令** | 放在 `.claude/commands/`，支持 `$ARGUMENTS` 参数 | 章节2.c |
| **多Claude工作流** | git worktree + 多终端，或使用子代理 | 章节6 |
| **上下文管理** | 常用 `/clear` 重置上下文，保持聚焦 | 章节4.f |

---

## 概述

Claude Code 是 Anthropic 推出的命令行智能体编程工具。它刻意设计得底层且无预设偏好（unopinionated），提供接近原始模型的访问能力。这种灵活性创造了学习曲线，本文总结了 Anthropic 内部和社区验证的最佳实践。

---

## 1. 定制你的设置

> 来源：章节1 - 定制你的设置

### a. 创建 CLAUDE.md 文件

> 来源：章节1.a - 创建 CLAUDE.md 文件

CLAUDE.md 是 Claude 在对话开始时自动拉取到上下文的特殊文件，适合记录：

- 常用的 bash 命令
- 核心文件和工具函数
- 代码风格指南
- 测试说明
- 代码仓库规范（分支命名、merge vs rebase 等）
- 开发者环境设置
- 项目特有的意外行为或警告

**示例 CLAUDE.md：**

```markdown
# Bash commands
- npm run build: Build the project
- npm run typecheck: Run the typechecker

# Code style
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible (eg. import { foo } from 'bar')

# Workflow
- Be sure to typecheck when you're done making a series of code changes
- Prefer running single tests, and not the whole test suite, for performance
```

**CLAUDE.md 放置位置：**
- 仓库根目录（最常见，命名为 `CLAUDE.md` 提交到 git，或 `CLAUDE.local.md` 加入 .gitignore）
- 父目录（适用于 monorepo）
- 子目录（按需拉取）
- 主文件夹 `~/.claude/CLAUDE.md`（全局）

运行 `/init` 命令可自动生成 CLAUDE.md。

### b. 优化你的 CLAUDE.md 文件

> 来源：章节1.b - 优化你的 CLAUDE.md 文件

CLAUDE.md 是提示的一部分，应像提示一样迭代优化：

- 使用 `#` 键给 Claude 指令，它会自动整合到 CLAUDE.md
- 将 CLAUDE.md 更改包含在 commit 中，让团队受益
- 使用 Anthropic 的提示词优化器（prompt improver）处理 CLAUDE.md
- 使用 "IMPORTANT" 或 "YOU MUST" 强调关键指令

### c. 管理 Claude 的允许工具列表

四种方法：
1. 会话中选择 "Always allow"
2. 使用 `/allowed-tools` 命令
3. 手动编辑 `.claude/settings.json` 或 `~/.claude.json`
4. 使用 `--allowedTools` 命令行标志

### d. 安装 gh CLI

Claude 知道如何使用 gh CLI 与 GitHub 交互（创建 issue、打开 PR、读取评论等）。

---

## 2. 扩展 Claude 的能力

> 来源：章节2 - 扩展 Claude 的能力

### a. 将 Claude 与 bash 工具结合使用

> 来源：章节2.a - 将 Claude 与 bash 工具结合使用

- 告诉 Claude 工具名称和使用示例
- 告诉 Claude 运行 `--help` 查看文档
- 在 CLAUDE.md 中记录常用工具

### b. 将 Claude 与 MCP 结合使用

Claude Code 同时是 MCP 服务器和客户端。三种连接方式：
- 项目配置
- 全局配置
- 签入的 `.mcp.json` 文件

使用 `--mcp-debug` 标志排查配置问题。

### c. 使用自定义斜杠命令

在 `.claude/commands/` 文件夹存储提示模板，输入 `/` 即可调用。支持 `$ARGUMENTS` 关键字传递参数。

**示例：fix-github-issue.md**

```markdown
Please analyze and fix the GitHub issue: $ARGUMENTS.

Follow these steps:
1. Use `gh issue view` to get the issue details
2. Understand the problem described in the issue
3. Search the codebase for relevant files
4. Implement the necessary changes to fix the issue
5. Write and run tests to verify the fix
6. Ensure code passes linting and type checking
7. Create a descriptive commit message
8. Push and create a PR
```

---

## 3. 常见工作流程

> 来源：章节3 - 常见工作流程

### a. 探索、计划、编码、提交 (Explore, Plan, Code, Commit)

> 来源：章节3.a - 探索、计划、编码、提交

1. 让 Claude 阅读相关文件（明确告诉它暂不编码）
2. 使用 "think" 触发扩展思考模式制定计划
3. 让 Claude 实现方案并验证
4. 提交结果并创建 PR

> 步骤 1-2 至关重要——没有它们，Claude 往往直接跳到编码。

### b. 测试驱动开发 (TDD)

> 来源：章节3.b - 测试驱动开发

1. 让 Claude 编写测试（明确说明 TDD，避免 mock 实现）
2. 运行测试确认失败
3. 提交测试
4. 让 Claude 编写通过测试的代码（指示不修改测试）
5. 提交代码

> 使用子代理验证实现是否过度拟合（overfitting）测试。

### c. 视觉迭代 (Write code, screenshot, iterate)

1. 提供视觉模型（截图、设计稿）
2. Claude 实现设计 → 截图 → 迭代
3. 满意后提交

> 2-3 次迭代后结果通常好得多。

### d. 安全 YOLO 模式

使用 `claude --dangerously-skip-permissions` 绕过权限检查，适合修复 lint 错误等。**务必在无互联网访问的容器中使用。**

### e. 代码库问答 (Codebase Q&A)

直接向 Claude 提问代码库相关问题——日志如何工作、如何创建新 API 端点等。这是 Anthropic 的核心入职工作流程。

### f. 使用 Claude 与 git 交互

- 搜索 git 历史记录
- 编写 commit 消息
- 处理复杂 git 操作（还原文件、解决 rebase 冲突、比较和嫁接补丁）

> Anthropic 工程师用 Claude 处理 90% 以上的 git 交互。

### g. 使用 Claude 与 GitHub 交互

- 创建 PR
- 一键修复代码审查评论
- 修复失败构建
- 分类和分流 issue

### h. 处理 Jupyter Notebook

读取和编写 notebook，解释输出（包括图像），清理和美化 notebook。

---

## 4. 优化工作流程

> 来源：章节4 - 优化工作流程

### a. 在指令中要具体

> 来源：章节4.a - 在指令中要具体

| 欠佳的指令 | 良好的指令 |
|-----------|-----------|
| 为 foo.py 添加测试 | 为 foo.py 编写测试，覆盖用户未登录的边缘情况，避免使用 mock |
| 为什么 ExecutionFactory 的 API 这么奇怪？ | 查看 ExecutionFactory 的 git 历史记录，总结其 API 是如何演变的 |
| 添加一个日历小部件 | 查看现有小部件实现方式，遵循模式实现日历小部件，允许选择月份和年份 |

### b. 给 Claude 图片

- 粘贴截图（macOS: cmd+ctrl+shift+4 → ctrl+v）
- 拖放图像
- 提供图像文件路径

### c. 提及文件

使用 tab 键补全快速引用文件。

### d. 给 Claude URL

粘贴 URL 让 Claude 获取和阅读。使用 `/allowed-tools` 避免重复权限提示。

### e. 尽早且经常地修正方向

四个修正工具：
1. 编码前要求 Claude 制定计划
2. 按 Escape 中断（保留上下文）
3. 双击 Escape 跳回历史编辑提示
4. 要求 Claude 撤销更改

### f. 使用 /clear 保持上下文聚焦

任务之间经常使用 /clear 重置上下文窗口。

### g. 对复杂工作流使用清单和草稿板

让 Claude 使用 Markdown 文件或 GitHub issue 作为清单，逐项解决。

### h. 向 Claude 传递数据

- 直接复制粘贴
- 管道输入（`cat foo.txt | claude`）
- 通过 bash/MCP 工具拉取
- 读取文件或获取 URL

---

## 5. 无头模式自动化

使用 `-p` 标志启用无头模式，`--output-format stream-json` 获取流式 JSON 输出。

### a. Issue 分流

GitHub 事件触发自动化。

### b. 代码检查器 (Linter)

提供主观代码审查，识别拼写错误、过时注释、误导性命名等。

---

## 6. 多 Claude 工作流

> 来源：章节6 - 多 Claude 工作流

### a. 一个编写，另一个验证

> 来源：章节6.a - 一个编写，另一个验证

- Claude A 编写代码
- `/clear` 或新终端启动 Claude B 审查
- Claude C 根据反馈修改代码

### b. 多个代码库检出

创建 3-4 个 git 检出，每个运行独立 Claude 任务，轮流查看进度。

### c. 使用 git worktrees

```bash
git worktree add ../project-feature-a feature-a
```

每个 worktree 运行独立 Claude 会话，互不干扰。

### d. 子代理 (Subagents)

Claude 可以将子任务委托给子代理。使用子代理可以：
- 保持主代理上下文窗口干净
- 并行处理独立子任务
- 让 Claude 花更多计算力解决复杂问题

---

## 核心要点总结

> 来源：章节7 - 核心要点总结

1. **CLAUDE.md 是关键** — 把它当提示工程来迭代优化
2. **先计划后编码** — "think" 触发扩展思考
3. **给具体指令** — 细节决定质量
4. **迭代优于一次性** — 2-3 轮迭代结果远超第一版
5. **多实例并行** — worktree + 多终端是效率倍增器
6. **工具链扩展** — MCP + 自定义命令 + gh CLI
7. **上下文管理** — /clear 保持聚焦，子代理分担负载
8. **错误驱动优化** — 每次纠错后更新 CLAUDE.md

---

## 信息来源追溯

| 本文件内容 | 主要来源 | 辅助来源 |
|-----------|---------|---------|
| CLAUDE.md 编写与优化 | Anthropic 官方博客 | - |
| 工具管理与 MCP | Anthropic 官方博客 | - |
| 常见工作流程 | Anthropic 官方博客 | - |
| 多 Claude 工作流 | Anthropic 官方博客 | - |
| 优化技巧 | Anthropic 官方博客 | - |

---

## 原始来源

- [Anthropic Engineering Blog - Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [GitHub coderPerseus/blog - 中文翻译](https://github.com/coderPerseus/blog/issues/63)

---
*初提炼完成时间: 2026-04-19 | 提炼人: Claude*
*L1 检查清单: ✅ frontmatter完整 | ✅ 核心要点速查(12条) | ✅ 关键段落标注来源 | ✅ 来源追溯表*


---

# 附录：补充内容（来自其他来源合并）

> 以下内容来自 Claude Code 最佳实践中文翻译版、Memory/Skills 指南、Boris Cherny 内部技巧等来源

---

## 附录A: Subagents 完整 Frontmatter 字段（16个）

> 来源：claude-code-best-practice-zh

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 唯一标识符，小写字母+连字符 |
| `description` | string | 是 | 何时调用，用 PROACTIVELY 实现自动调用 |
| `tools` | string/list | 否 | 工具白名单，省略则继承全部 |
| `disallowedTools` | string/list | 否 | 禁用的工具 |
| `model` | string | 否 | sonnet/opus/haiku/完整模型ID/inherit |
| `permissionMode` | string | 否 | default/acceptEdits/auto/dontAsk/bypassPermissions/plan |
| `maxTurns` | integer | 否 | 最大交互轮次 |
| `skills` | list | 否 | 预加载的技能名（完整内容注入） |
| `mcpServers` | list | 否 | MCP 服务器配置 |
| `hooks` | object | 否 | 生命周期钩子（PreToolUse/PostToolUse/Stop 最常用） |
| `memory` | string | 否 | user/project/local |
| `background` | boolean | 否 | true 则始终后台运行 |
| `effort` | string | 否 | low/medium/high/max（仅 Opus 4.6） |
| `isolation` | string | 否 | worktree 则在临时 git worktree 中运行 |
| `initialPrompt` | string | 否 | 作为主会话代理时的首条自动提交提示 |
| `color` | string | 否 | 显示颜色：red/blue/green/yellow/purple/orange/pink/cyan |

### 官方内置 Agent（5个）

| 代理 | 模型 | 工具 | 说明 |
|------|------|------|------|
| `general-purpose` | inherit | 全部 | 复杂多步任务默认代理 |
| `Explore` | haiku | 只读 | 快速代码库搜索和探索 |
| `Plan` | inherit | 只读 | 编码前预规划研究 |
| `statusline-setup` | sonnet | Read, Edit | 配置状态栏 |
| `claude-code-guide` | haiku | Glob,Grep,Read,WebFetch,WebSearch | Claude Code 功能问答 |

---

## 附录B: Skills 完整 Frontmatter 字段（14个）

> 来源：claude-code-best-practice-zh

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 否 | 显示名称和 slash-command 标识符 |
| `description` | string | 推荐 | 功能描述，显示在自动补全中 |
| `when_to_use` | string | 否 | 触发时机和示例请求 |
| `argument-hint` | string | 否 | 自动补全提示 |
| `disable-model-invocation` | boolean | 否 | true 阻止自动调用 |
| `user-invocable` | boolean | 否 | false 从菜单隐藏 |
| `allowed-tools` | string | 否 | 无需权限提示的工具 |
| `model` | string | 否 | 运行时模型 |
| `effort` | string | 否 | 覆盖努力级别 |
| `context` | string | 否 | fork 在隔离子代理中运行 |
| `agent` | string | 否 | context:fork 时的子代理类型 |
| `hooks` | object | 否 | 生命周期钩子 |
| `paths` | string/list | 否 | 限制自动激活的 glob 模式 |
| `shell` | string | 否 | bash（默认）或 powershell |

### 官方内置 Skills（5个）

| Skill | 说明 |
|-------|------|
| `simplify` | 审查代码复用性和质量，重构消除重复 |
| `batch` | 批量跨多文件运行命令 |
| `debug` | 调试失败命令或代码问题 |
| `loop` | 按固定间隔运行提示（最长3天） |
| `claude-api` | 使用 Claude API/SDK 构建应用 |

---

## 附录C: CLAUDE.md Monorepo 加载机制

> 来源：claude-code-best-practice-zh

- **祖先加载**：启动时从当前目录向上遍历加载所有 CLAUDE.md
- **后代懒加载**：子目录 CLAUDE.md 仅在读取该子目录文件时加载
- **兄弟互不加载**：在 frontend/ 工作时不会加载 backend/CLAUDE.md
- 共享约定放根 CLAUDE.md，组件专用指令放组件 CLAUDE.md
- 个人偏好用 CLAUDE.local.md，加入 .gitignore

---

## 附录D: Boris Cherny 内部团队10条实战技巧

> 来源：Claude Code 创始人 Boris Cherny 2026年1月推特分享

1. **多开 Worktree** - 同时跑3-5个 git worktree，公认最提效方式
2. **复杂任务先规划** - 让一个 Claude 写计划，另一个审查
3. **错误后更新 CLAUDE.md** - 构建复合增长系统，每次纠错让 Claude 更聪明
4. **自建 Skills 库** - 一天做两次以上的事就该做成 Skills
5. **让 Claude 自己修 Bug** - 接入 Slack MCP，说"修它"
6. **提高提示词质量** - "严格审查这些改动，测试不过不准建 PR"
7. **追求更优方案** - "废掉这个方案，实现更优雅的"
8. **终端配置** - Ghostty终端 + /statusline + 颜色编码标签页
9. **语音输入** - 比打字快三倍，提示词自然更详细（macOS 双击fn）
10. **用子代理** - "use subagents"，保持主代理上下文干净

### 实战组合模式

**并行开发**：3个worktree各自跑独立Claude会话
**规划-执行-审查**：Claude A规划 - B审查 - A修改 - C验证
**纠错自迭代**：犯错-纠正-更新CLAUDE.md-重复直到错误率下降
