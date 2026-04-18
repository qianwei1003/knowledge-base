# MOC - 工具配置

---
created: 2026-04-18
source: #来源/Claude
tags: [#技术/工具, #知识库/索引]
---

> AI 编程工具的配置指南，包括 Copilot Instructions、AGENTS.md、其他 AI 工具配置等。让 AI 记住团队规范，实现一致性的开发体验。

---

## 📚 工具资源

| 工具 | 配置方式 | 相关笔记 | 状态 |
|-----|---------|---------|------|
| GitHub Copilot | copilot-instructions.md | [[Copilot配置指南]] | ✅ |
| GitHub Copilot | AGENTS.md | [[Copilot配置指南]] | ✅ |
| GitHub Copilot | *.instructions.md | [[Copilot配置指南]] | ✅ |
| Claude Code | Rules 配置 | 待补充 | ❌ |
| Cursor | .cursorrules | 待补充 | ❌ |

---

## 🔍 核心概念

### 指令文件类型

| 类型 | 位置 | 作用范围 |
|-----|------|---------|
| `.github/copilot-instructions.md` | 工作区根目录 | 所有聊天请求 |
| `*.instructions.md` | `.github/instructions/` | 按文件类型/路径 |
| `AGENTS.md` | 工作区根目录或子目录 | 所有聊天请求（多 Agent） |

### applyTo 模式

| 模式 | 说明 |
|------|------|
| `**/*.py` | 所有 Python 文件 |
| `src/**` | src 目录 |
| `api/**/*.go` | api 目录下所有 Go 文件 |

---

## 📝 详细笔记

| 笔记 | 内容 | 优先级 |
|-----|------|-------|
| [[Copilot配置指南]] | 三种指令文件详解、团队配置示例 | P0 |

---

## 🔗 关联笔记

- 大厂编程规范 - 规范来源
- SDD规范驱动开发 - SDD 框架
- prompt工程 - 提示词设计

---

## 最近更新

| 日期 | 更新内容 |
|-----|---------|
| 2026-04-18 | 初始化工具配置 MOC |
