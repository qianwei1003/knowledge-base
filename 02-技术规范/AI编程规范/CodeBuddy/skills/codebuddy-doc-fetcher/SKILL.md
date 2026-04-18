---
name: codebuddy-doc-fetcher
description: CodeBuddy 官方文档获取工具。触发词：查 CodeBuddy 文档、获取官方文档、codebuddy docs、文档格式要求、skills 规范、rules 规范、agents 规范、subagent 配置
allowed-tools:
  - web_fetch
  - web_search
disable: false
---

# CodeBuddy 文档获取工具

这是一个专门用于获取 CodeBuddy 官方文档的 Skill。

## 快速使用

当您需要了解 CodeBuddy 的以下内容时，此 Skill 将自动激活：

### 可获取的文档类型

| 文档类型 | 包含内容 | 官方链接 |
|:---|:---|:---|
| **Skills 技能** | SKILL.md 格式、frontmatter 字段、目录结构、触发机制 | [Skills 文档](http://www.codebuddy.ai/docs/zh/ide/Features/Skills) |
| **Rules 规则** | 规则文件格式、条件触发、paths 配置、层次结构 | [Rules 文档](http://www.codebuddy.ai/docs/zh/ide/User-guide/rules) |
| **Agents 智能体** | Agent 文件格式、agentMode、可用工具、配置文件 | [Subagents 文档](https://www.codebuddy.cn/docs/ide/Features/Subagents) |
| **完整指南** | 三大扩展机制的完整说明和最佳实践 | [官方首页](https://www.codebuddy.ai/docs/zh/ide/Introduction) |

## 常用查询示例

### 示例 1：获取 Skills 格式规范
```
用户：查看 skills 文件的 frontmatter 有哪些字段
Skill：自动调用 web_fetch 获取 Skills 文档
```

### 示例 2：了解 Agent 配置
```
用户：subagent 的 agentMode 有哪些模式？
Skill：自动抓取 Subagents 文档并提取相关信息
```

### 示例 3：创建自定义规则
```
用户：怎么配置 paths 条件规则？
Skill：返回 Rules 文档中关于 paths 配置的说明
```

## 核心功能

1. **智能识别**：解析用户对 CodeBuddy 文档的模糊需求
2. **自动定位**：通过关键词匹配找到对应的官方文档页面
3. **内容抓取**：使用 web_fetch 获取最新文档内容
4. **格式整理**：将文档整理为可读性强的 Markdown 格式

## 文档 URL 映射

| 文档需求 | URL |
|:---|:---|
| 官方首页/概览 | https://www.codebuddy.ai/docs/zh/ide/Introduction |
| Skills 详细文档 | http://www.codebuddy.ai/docs/zh/ide/Features/Skills |
| Rules 使用指南 | http://www.codebuddy.ai/docs/zh/ide/User-guide/rules |
| Subagents 文档 | https://www.codebuddy.cn/docs/ide/Features/Subagents |
| Memory 管理 | https://www.codebuddy.ai/docs/cli/memory |

## 注意事项

- 此 Skill 使用 `web_fetch` 工具实时获取最新文档
- 如需更详细的示例代码，建议同时查看知识库中的 `CodeBuddy/CodeBuddy扩展机制详解.md`
- 文档内容以 CodeBuddy 官方为准，此 Skill 仅为便捷访问工具
