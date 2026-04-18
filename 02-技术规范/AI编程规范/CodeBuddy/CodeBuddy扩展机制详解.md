# CodeBuddy 扩展机制详解

## 一、核心概念概览

| 组件 | 定位 | 核心目的 | 文件位置 |
| :--- | :--- | :--- | :--- |
| **Skills** | 模块化的能力包 | 针对特定领域或任务，封装专业知识、工作流和工具，将通用 AI 转变为领域专家 | `.codebuddy/skills/<skill-name>/` |
| **Rules** | 模块化的行为规范 | 强制 AI 遵循特定的编码规范、架构设计或项目约束，实现标准化和一致性 | `.codebuddy/rules/` |
| **Agents** | 专业化的 AI 助手 | 创建专注于特定任务（如代码审查、调试）的专用 AI，提升任务处理的精准度和效率 | `.codebuddy/agents/` |

**设计原则**：三者都采用 **Markdown 文件 + YAML Frontmatter** 的配置方式，支持**渐进式披露**（按需加载），并区分**项目级**与**用户级**作用域。

---

## 二、Skills 技能系统

### 2.1 目录结构

```
skill-name/
├── SKILL.md                # 核心定义文件（必需）
├── scripts/                # 可执行代码（Python, Bash等）
├── references/             # 辅助文档（按需加载）
└── assets/                 # 输出用文件（模板、图标等）
```

### 2.2 SKILL.md 格式

```yaml
---
name: skill-name
description: 技能的简短描述，包含触发词
allowed-tools:           # 可选：工具白名单
  - read_file
  - search_content
disable: false           # 可选：禁用开关
---
```

### 2.3 主体内容规范

- 使用指令性 Markdown 描述工作流、步骤
- 可引用 `scripts/`、`references/`、`assets/` 等可选资源目录
- 遵循渐进式披露原则

### 2.4 触发机制

- **自动触发**：当用户的请求与 `description` 中的触发词匹配时自动加载
- **手动触发**：用户可通过 `/skill-name` 命令显式调用

---

## 三、Rules 规则系统

### 3.1 文件位置

- **用户级规则**：`~/.codebuddy/rules/` - 适用于所有项目
- **项目级规则**：`.codebuddy/rules/` - 仅适用于当前项目
- **优先级**：项目级规则 > 用户级规则

### 3.2 规则文件格式

```yaml
---
enabled: true              # 是否启用该规则
alwaysApply: false         # 是否始终注入上下文
paths:                     # 可选：条件触发路径
  - "**/*.ts"
  - "**/*.tsx"
---
```

### 3.3 条件规则

通过 `paths` 字段可实现条件规则，仅当操作匹配特定文件时才生效。

### 3.4 规则组织

推荐目录结构：
```
rules/
├── common/          # 语言无关原则
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   └── security.md
├── typescript/      # TypeScript 特定
├── python/          # Python 特定
└── golang/          # Go 特定
```

---

## 四、Agents 智能体系统

### 4.1 文件位置

- **用户级 Agents**：`~/.codebuddy/agents/` - 所有项目可用
- **项目级 Agents**：`.codebuddy/agents/` - 仅当前项目可用

### 4.2 Agent 文件格式

```yaml
---
name: agent-name                    # Agent 名称
description: 简短描述              # 显示在 Agent 列表中
agentMode: manual                  # 模式：manual（手动）或 agentic（自动）
enabled: true                      # 是否启用
enabledAutoRun: false              # 是否自动运行（仅 agentic 模式）
model: claude-sonnet-4-20250514    # 可选：指定模型
tools:                             # 可选：可用工具列表
  - read_file
  - search_content
systemPrompt: |                   # 系统提示词
  你是专业的...
---
```

### 4.3 模式说明

| 模式 | 说明 | 触发方式 |
|:---|:---|:---|
| `manual` | 需用户手动选择 | 用户在 Agent 列表中选择 |
| `agentic` | 由主 Agent 自动调用 | 主 Agent 根据任务类型自动调度 |

### 4.4 可用工具

- 基础工具：`read_file`、`search_content`、`search_file`、`execute_command` 等
- 集成工具：`invoke_integration`、`mcp_*` 等
- 自定义工具：可按需扩展

---

## 五、Skills 与 Agents 的协作

### 5.1 典型协作模式

```
用户请求 → Agent 分析 → 调用对应 Skill → 执行具体任务
```

### 5.2 组合示例

- `code-reviewer` Agent + `coding-standards` Skill → 代码审查
- `planner` Agent + `architecture-decision-records` Skill → 架构规划

### 5.3 配置优先级

1. 项目级 Skills/Agents > 用户级 Skills/Agents
2. 更具体的规则 > 通用规则

---

## 六、快速开始

### 6.1 创建自定义 Skill

```bash
# 创建目录结构
.codebuddy/skills/my-skill/
├── SKILL.md
└── references/
    └── guide.md
```

### 6.2 创建自定义 Agent

```bash
# 创建文件
.codebuddy/agents/my-agent.md
```

### 6.3 创建自定义 Rule

```bash
# 创建规则文件
.codebuddy/rules/coding-standards.md
```

---

## 七、相关资源

- [官方文档首页](https://www.codebuddy.ai/docs/zh/ide/Introduction)
- [Skills 文档](http://www.codebuddy.ai/docs/zh/ide/Features/Skills)
- [Rules 文档](http://www.codebuddy.ai/docs/zh/ide/User-guide/rules)
- [Subagents 文档](https://www.codebuddy.cn/docs/ide/Features/Subagents)

---

*最后更新：2026-04-18*
