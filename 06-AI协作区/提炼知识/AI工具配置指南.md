# AI 工具配置指南

> 基于 Copilot 配置指南提炼的 AI 编程工具配置最佳实践，实现团队规范的自动化执行。

---

## 一、Copilot 指令文件体系

### 1.1 三层指令架构

```
┌─────────────────────────────────────────────────────────────┐
│                    AI 工具指令体系                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   .github/                    .github/              AGENTS.md │
│   copilot-instructions.md    instructions/                    │
│   ┌─────────────────┐       ┌─────────────────┐  ┌─────────────────┐│
│   │ 全局通用规范     │       │ 按场景分类规范   │  │ 多 Agent 指南   ││
│   │                 │       │                 │  │                 ││
│   │ - 命名规范       │       │ - api/*.go      │  │ - 代码生成指南   ││
│   │ - 注释规范       │       │ - src/**/*.ts   │  │ - 代码审查指南   ││
│   │ - 安全规范       │       │ - test/**       │  │ - 重构指南       ││
│   │ - Git 规范       │       │                 │  │                 ││
│   └────────┬────────┘       └────────┬────────┘  └────────┬────────┘│
│            │                         │                    │          │
│            └─────────────────────────┴────────────────────┘          │
│                              ↓                                        │
│                    所有聊天请求自动应用                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 文件对比

| 文件 | 位置 | 作用范围 | 适用场景 |
|-----|------|---------|---------|
| `.github/copilot-instructions.md` | 根目录 | 所有请求 | 团队通用规范 |
| `.github/instructions/*.instructions.md` | instructions/ | 按 applyTo | 特定语言/框架 |
| `AGENTS.md` | 根目录或子目录 | 所有请求 | 多 Agent 协作 |

---

## 二、配置模板

### 2.1 全局通用规范

> 文件：`.github/copilot-instructions.md`

```markdown
# {{项目名称}} - 团队编码规范

## 规范来源
本配置整合了以下规范：
- 大厂编程规范
- SDD规范驱动开发
- AI行为规范

## 核心原则

### 1. SDD 工作流
- 新功能遵循：Proposal → Spec → Design → Tasks
- 变更必须文档化，不口头传达
- 每个产出标注对应的规范条款

### 2. 代码风格
- 命名：{{命名规范}}
- 注释：所有导出函数添加 JSDoc
- 格式化：通过 ESLint/Prettier 检查

### 3. 安全规范
- 禁止硬编码密钥和凭证
- 用户输入必须验证
- 遵循 OWASP Top 10

### 4. 可维护性
- 函数不超过 20 行
- 单一职责原则
- 禁止魔法数字
```

### 2.2 按语言配置

> 文件：`.github/instructions/typescript.instructions.md`

```markdown
---
name: "TypeScript 编码规范"
description: "TypeScript/React 代码规范"
applyTo: "**/*.{ts,tsx}"
---

# TypeScript 编码规范

## 类型规范
- 禁止使用 any，必须指定具体类型
- 优先使用 interface 而非 type
- 导出类型而非实现

## React 规范
- 使用函数式组件
- Props 必须有类型定义
- 状态使用 useState/useReducer

## 命名规范
- 组件：PascalCase（如 UserCard）
- Hooks：use 前缀（如 useAuth）
- 类型：PascalCase（如 UserProps）
```

### 2.3 API 层配置

> 文件：`.github/instructions/api.instructions.md`

```markdown
---
name: "API 服务规范"
description: "后端 API 代码规范"
applyTo: "api/**/*.{ts,go,java}"
---

# API 服务规范

## 接口设计
- RESTful 命名规范
- 统一的错误响应格式
- 请求参数必须验证

## 错误处理
- 区分业务错误和系统错误
- 错误信息包含错误码
- 关键错误必须记录日志

## 安全
- 认证/授权检查不遗漏
- 敏感数据加密传输
- SQL 参数化查询
```

---

## 三、团队协作配置

### 3.1 多语言项目结构

```
.github/
├── copilot-instructions.md          # 全局规范（所有文件）
└── instructions/
    ├── frontend.instructions.md     # 前端规范（applyTo: "frontend/**"）
    ├── backend.instructions.md       # 后端规范（applyTo: "backend/**"）
    ├── mobile.instructions.md        # 移动端规范（applyTo: "mobile/**"）
    ├── shared-test.instructions.md   # 测试规范（applyTo: "**/*.{spec,test}.*"}）
    └── shared-docs.instructions.md   # 文档规范（applyTo: "docs/**"）
```

### 3.2 职责分离

| 配置 | 责任人 | 更新频率 | 版本控制 |
|-----|--------|---------|---------|
| 全局规范 | 技术负责人 | 季度 | 必须 |
| 语言规范 | 各语言owner | 按需 | 必须 |
| 项目规范 | 项目经理 | 按需 | 必须 |

### 3.3 CI/CD 集成

```yaml
# .github/workflows/copilot-check.yml
name: Copilot Config Check

on:
  push:
    paths:
      - '.github/**/*.md'
      - '.github/instructions/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Check instructions syntax
        run: |
          # 验证 frontmatter 格式
          for file in .github/instructions/*.instructions.md; do
            if ! grep -q "^---" "$file"; then
              echo "Missing frontmatter in $file"
              exit 1
            fi
          done
```

---

## 四、与 codebuddy-craft 的集成

### 4.1 从知识库到配置

```
提炼知识/                       .github/
┌─────────────────┐             ┌─────────────────┐
│ AI行为规范      │  ──►       │ copilot-        │
│                 │             │ instructions.md  │
│ - SDD原则       │             │                 │
│ - 一致性规则     │             │ - 命名规范       │
│ - 安全要求       │             │ - 注释规范       │
└─────────────────┘             └─────────────────┘

┌─────────────────┐             ┌─────────────────┐
│ 前端规范-综合   │  ──►       │ instructions/   │
│                 │             │ frontend.*.md    │
│ - 组件规范      │             │                 │
│ - 状态管理      │             │ - React规范     │
│ - 性能要求      │             │ - TypeScript规范│
└─────────────────┘             └─────────────────┘
```

### 4.2 自动生成脚本

```bash
#!/bin/bash
# generate-copilot-config.sh

# 从提炼知识生成 copilot-instructions.md
cat > .github/copilot-instructions.md << EOF
# $(basename $(pwd)) - 团队编码规范

## 自动生成
本文件由 $(date +%Y-%m-%d) 自动生成

$(cat ../AI行为规范.md | grep -A 100 "## 核心原则")

EOF
```

---

## 五、配置检查清单

### 5.1 新项目检查

- [ ] 创建 `.github/` 目录
- [ ] 创建 `copilot-instructions.md`
- [ ] 为每种语言创建 `instructions/*.instructions.md`
- [ ] 配置 `applyTo` 模式
- [ ] 纳入 Git 版本控制

### 5.2 配置验证

```markdown
## 配置验证清单

### 格式检查
- [ ] 所有 .instructions.md 有 frontmatter
- [ ] frontmatter 包含 name 和 description
- [ ] applyTo 模式格式正确

### 内容检查
- [ ] 命名规范与项目一致
- [ ] 安全要求完整
- [ ] SDD 工作流说明清晰

### 协作检查
- [ ] 所有成员可见
- [ ] 变更经过评审
- [ ] 有更新记录
```

---

## 六、相关链接

- Copilot配置指南 - 详细配置来源
- 大厂编程规范 - 规范来源
- AI行为规范 - AI 行为准则

---

## 更新记录

| 日期 | 更新内容 |
|-----|---------|
| 2026-04-18 | 提炼 Copilot 配置最佳实践 |
