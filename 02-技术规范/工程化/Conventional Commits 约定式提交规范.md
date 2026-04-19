---
来源: 官方规范 (Conventional Commits v1.1.0)
原始链接: https://www.conventionalcommits.org
---

# Conventional Commits 约定式提交规范

> Conventional Commits 是一套轻量级的提交信息约定，为创建清晰的提交历史提供简单规则，使自动化工具更容易基于提交信息工作。

## 提交信息结构

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 组成部分

| 部分 | 必填 | 说明 |
|------|------|------|
| **type** | ✅ | 提交类型，如 feat、fix |
| **scope** | ❌ | 影响范围，用括号包围，如 `(parser)` |
| **description** | ✅ | 简短描述变更内容 |
| **body** | ❌ | 详细说明变更原因和内容，与 description 之间空一行 |
| **footer(s)** | ❌ | 破坏性变更说明或 Issue 引用，与 body 之间空一行 |

## 核心提交类型

| 类型 | 说明 | SemVer 影响 |
|------|------|------------|
| `feat` | 新增功能 | MINOR（次版本） |
| `fix` | 修复 Bug | PATCH（修订号） |
| `BREAKING CHANGE` | 破坏性 API 变更 | MAJOR（主版本） |

## 扩展提交类型

| 类型 | 说明 |
|------|------|
| `docs` | 文档更新 |
| `style` | 代码格式调整（不影响逻辑） |
| `refactor` | 代码重构（非功能新增或修复） |
| `perf` | 性能优化 |
| `test` | 添加或修改测试 |
| `build` | 构建系统或外部依赖变更 |
| `ci` | CI/CD 配置变更 |
| `chore` | 其他不涉及 src 或 test 的变动 |
| `revert` | 回滚之前的提交 |

## 提交示例

### 带描述和破坏性变更

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

### 使用 ! 标记破坏性变更

```
feat!: send an email to the customer when a product is shipped
```

### 带作用域

```
feat(lang): add Polish language
```

### 简单文档更新

```
docs: correct spelling of CHANGELOG
```

### 多段落正文 + 脚注

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

## 描述规则

- 使用**祈使语气**（如 "add feature" 而非 "added feature"）
- 首字母**小写**，结尾**不加句号**
- 长度建议**不超过 50 个字符**
- 说明"做了什么"而非"怎么做的"（代码差异已通过 Git 展示）

## Body 规则

- 与 description 之间**空一行**
- 解释**为什么**修改（动机、背景）
- 每行建议不超过 **72 个字符**
- 可多段落，段落间空行分隔

## Footer 规则

- 与 body 之间**空一行**
- 关联 Issue：`Closes #123`、`Refs #456`
- 破坏性变更：`BREAKING CHANGE: <描述>` 或在 type 后加 `!`
- 审阅人：`Reviewed-by: <name>`

## 与 SemVer 的关系

| 提交类型 | 版本号变更 |
|---------|----------|
| `feat` | 1.x.y → 1.(x+1).0 |
| `fix` | 1.x.y → 1.x.(y+1) |
| `BREAKING CHANGE` | 1.x.y → 2.0.0 |

## 工具推荐

| 工具 | 用途 |
|------|------|
| `commitizen` | 交互式生成规范提交信息 |
| `commitlint` | 提交信息格式校验（配合 husky） |
| `standard-version` | 自动生成 CHANGELOG 和版本号 |
| `husky` | Git hooks 管理 |

## 最佳实践

1. **团队统一约定**：确定团队使用的 type 和 scope 列表
2. **配合 CI 校验**：通过 commitlint + husky 在提交时自动检查
3. **自动生成 CHANGELOG**：使用 standard-version 根据提交记录自动生成
4. **善用 scope**：标注修改的模块/组件，如 `feat(auth):`、`fix(api):`
5. **逐步采用**：不必一步到位，从核心类型（feat/fix）开始

## 常见问题

**Q: 初始开发阶段需要遵循吗？**  
A: 建议遵循，即使是团队内部使用，也能帮助追踪变更内容。

**Q: 提交类型区分大小写吗？**  
A: 规范不要求，但建议团队内部统一使用小写。

**Q: 一个提交符合多种类型怎么办？**  
A: 尽可能拆分为多个原子提交，每个提交只做一件事。

**Q: 错误使用了提交类型怎么办？**  
A: 发布前可用 `git rebase -i` 修改；已发布的不合规提交只会被工具忽略，不会导致严重问题。
