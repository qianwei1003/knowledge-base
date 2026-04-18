# Copilot 配置指南

> 收集整理 GitHub Copilot 和类似 AI 编程工具的配置方法，让 AI 记住团队规范，实现一致性的开发体验。

---

## 一、三种指令文件类型

### 1.1 `.github/copilot-instructions.md`

| 属性 | 说明 |
|------|------|
| **位置** | 工作区根目录的 `.github/` 文件夹下 |
| **作用范围** | 自动应用于工作区内的所有聊天请求 |
| **兼容性** | VS Code、Visual Studio 和 GitHub.com 的 Copilot |
| **启用设置** | `github.copilot.chat.codeGeneration.useInstructionFiles`（默认已启用）|

**示例结构：**

```markdown
# 项目编码规范

- 遵循Go官方编码风格指南
- 遵循项目中定义的gofmt和golint规则
- 为所有导出函数添加文档注释
- 使用context进行超时和取消控制
```

### 1.2 `*.instructions.md`

| 属性 | 说明 |
|------|------|
| **默认位置** | `.github/instructions/` 文件夹下 |
| **作用范围** | 可通过 `applyTo` 属性指定特定文件类型或路径 |
| **灵活性** | 可创建多个指令文件，每个针对不同场景 |
| **配置方式** | 支持 YAML frontmatter 配置 |

**示例结构：**

```markdown
---
name: "Go编码规范"
description: "Go文件的编码规范"
applyTo: "**/*.go"
---

# Go编码规范

- 遵循Go官方编码风格指南
- 使用gofmt格式化代码
- 为所有导出的函数和类型添加文档注释
- 使用testing包编写单元测试
```

### 1.3 `AGENTS.md`

| 属性 | 说明 |
|------|------|
| **位置** | 工作区根目录或子目录 |
| **作用范围** | 自动应用于所有聊天请求 |
| **适用场景** | 为多 AI Agent 环境定义指令 |
| **实验性功能** | 支持嵌套的 AGENTS.md 文件 |
| **启用设置** | `chat.useAgentsMdFile`（默认已启用）|

**示例结构：**

```markdown
# 开发代理指南

## 代码生成
- 始终包含错误处理
- 为重要操作添加日志记录
- 使用环境变量进行配置
- 永远不要提交密钥或凭证
```

---

## 二、applyTo 模式详解

`applyTo` 属性用于指定指令自动应用的文件范围，使用标准的 glob 模式。

### 2.1 支持的模式

| 模式 | 说明 |
|------|------|
| `**/*.py` | 所有 Python 文件 |
| `src/**` | src 目录下的所有文件 |
| `**/test/**` | 所有测试目录 |
| `**` | 所有文件 |
| `api/**/*.go` | api 目录下所有 Go 文件 |

### 2.2 配置示例

```yaml
---
name: "API服务编码规范"
description: "API服务Go代码规范"
applyTo: "api/**/*.go"
---
```

### 2.3 注意事项

> ⚠️ **重要**：如果工作区下包含多个项目，且项目下有独立的 `.github/instructions/` 指令配置，这些指令无法被编辑器自动识别和应用，需要手动 **Add Context** 才能使用。

---

## 三、作用域管理

### 3.1 工作区级别（Workspace Level）

| 属性 | 说明 |
|------|------|
| **存储位置** | `.github/instructions/` 或其他配置的工作区路径 |
| **作用范围** | 仅在当前工作区生效 |
| **适用场景** | 项目特定的规范和准则 |
| **版本控制** | 应纳入 Git 管理，供团队共享 |

### 3.2 用户级别（User Level）

| 属性 | 说明 |
|------|------|
| **存储位置** | 用户配置文件目录 |
| **作用范围** | 在所有工作区生效 |
| **适用场景** | 个人偏好的编码风格 |
| **跨设备同步** | 可通过 Settings Sync 同步 |

---

## 四、团队协作配置

### 4.1 创建团队共享指令

在工作区根目录创建 `.github/copilot-instructions.md`，确保纳入 Git 版本控制：

```markdown
# 团队编码规范

## 命名规范
- 变量：驼峰命名法（camelCase）
- 常量：全大写下划线命名法（UPPER_SNAKE_CASE）
- 类型：大驼峰命名法（PascalCase）

## Git工作流
- 功能分支命名：feature/TICKET-123-description
- 提交信息格式：type(scope): message
- 合并前压缩提交

## 安全规范
- 永远不要硬编码密钥或凭证
- 所有敏感配置使用环境变量
- 用户输入必须验证
```

### 4.2 多语言项目配置结构

```
.github/instructions/
├── api-service.instructions.md     (applyTo: "api/**")
├── data-layer.instructions.md      (applyTo: "dao/**")
├── business-logic.instructions.md  (applyTo: "service/**")
├── shared-testing.instructions.md  (applyTo: "**/*_test.go")
└── shared-docs.instructions.md     (applyTo: "docs/**")
```

### 4.3 跨设备同步

用户级别的指令文件可以通过 **Settings Sync** 同步：

1. 确保已启用 Settings Sync
2. 命令面板 → Settings Sync: Configure
3. 确保选中 "Prompts and Instructions"

---

## 五、frontmatter 配置项

| 配置项 | 必填 | 说明 |
|--------|------|------|
| `name` | 否 | 指令文件的显示名称，在 UI 中展示 |
| `description` | 否 | 指令文件的简短描述，说明其用途 |
| `applyTo` | 否 | Glob 模式，定义指令自动应用的文件范围 |

### 配置提示

- `applyTo` 支持标准的 glob 模式
- 如果不设置 `applyTo`，指令不会自动应用，只能手动附加
- 多个指令文件可以匹配同一个文件，VS Code 会合并所有匹配的指令

---

## 六、设置集成

可在 VS Code 设置中定义特定任务的指令：

```json
{
  "github.copilot.chat.reviewSelection.instructions": [
    { "text": "检查安全漏洞和数据验证" },
    { "text": "验证错误处理和边界情况" },
    { "file": "guidelines/code-review-checklist.md" }
  ],
  "github.copilot.chat.commitMessageGeneration.instructions": [
    { "text": "遵循Conventional Commits格式" },
    { "text": "包含工单号，格式：[PROJ-123]" },
    { "text": "保持主题行在50字符以内" }
  ],
  "github.copilot.chat.pullRequestDescriptionGeneration.instructions": [
    { "text": "包含：做什么、为什么、怎么做" },
    { "text": "列出所有破坏性变更" },
    { "text": "添加测试说明" }
  ]
}
```

---

## 七、快速开始

### 7.1 自动生成指令

VS Code 支持自动生成指令文件：

1. 打开聊天视图
2. 点击配置按钮 → "Generate Chat Instructions"
3. VS Code 会分析项目结构、代码风格等
4. 生成 `.github/copilot-instructions.md` 文件
5. 审查并根据需要调整内容

### 7.2 手动创建指令

```markdown
# .github/copilot-instructions.md

# 项目编码规范

## 通用规范
- 遵循本项目的编码风格
- 代码必须通过 ESLint/Prettier 检查
- 为所有导出的函数添加 JSDoc 注释

## 安全规范
- 永远不要硬编码密钥
- 所有用户输入必须验证
- 遵循 OWASP Top 10 防护

## 性能规范
- 避免不必要的重渲染
- 使用懒加载优化资源
- 避免 N+1 查询问题
```

---

## 八、codebuddy-craft 集成

### 8.1 生成 Copilot 配置

基于大厂编程规范和SDD规范驱动开发生成 Copilot 配置：

```markdown
# .github/copilot-instructions.md

# codebuddy-craft 项目规范

## 规范来源
本配置整合了以下规范：
- 大厂编程规范
- SDD规范驱动开发

## 核心原则

### 1. 一致性原则
- 遵循团队统一的命名规范
- 使用 Prettier 格式化代码
- 添加必要的注释说明

### 2. SDD 工作流
- 新功能遵循 Proposal → Spec → Design → Tasks 流程
- 变更必须文档化
- 验收必须可追溯

### 3. 安全原则
- 用户输入必须验证
- 敏感数据不硬编码
- 遵循 OWASP Top 10

### 4. 可维护性原则
- 函数保持简短（<20行）
- 单一职责原则
- 避免魔法数字
```

### 8.2 多语言配置示例

```markdown
# .github/instructions/typescript.instructions.md
---
name: "TypeScript 编码规范"
description: "TypeScript/React 代码规范"
applyTo: "**/*.ts"
---

# TypeScript 编码规范

## 类型规范
- 禁止使用 any，必须指定具体类型
- 优先使用 interface 而非 type
- 导出类型而非实现

## React 规范
- 组件使用函数式组件
- 状态使用 useState 或 useReducer
- 使用 TypeScript 定义 props 类型
```

---

## 九、相关链接

- 大厂编程规范 - 规范来源
- [[../ai-programming/SDD规范驱动开发]] - SDD 框架
- [[../ai-programming/prompt工程]] - 提示词设计
- [[./MOC]] - 工具配置总览

---

## 参考资料

| 来源 | 链接 | 核心价值 |
|-----|------|---------|
| John's Blog | [Copilot Instructions 指南](https://johng.cn/ai/copilot-instructions-guide) | 配置详解 |
| CSDN | [copilot-instructions.md vs AGENTS.md](https://blog.csdn.net/Lvyizhuo/article/details/158382558) | 对比分析 |

---

## 更新记录

| 日期 | 更新内容 |
|-----|---------|
| 2026-04-18 | 初始化 Copilot 配置指南文档 |
