# CodeBuddy 技术规范索引

## 📚 文档概览

本目录收录 CodeBuddy IDE 扩展机制的完整技术规范，包括 Skills、Rules、Agents 三大核心系统的文件格式、配置方法和最佳实践。

## 📁 目录结构

```
CodeBuddy/
├── CodeBuddy扩展机制详解.md    # 📖 三大扩展机制的完整说明
├── MOC-CodeBuddy.md            # 📑 本索引文件
├── rules/                      # 📋 自定义 Rules 示例
│   └── ts-vue-ai-coding.md     # 🎯 TypeScript+Vue AI编程规范（基于实践迭代）
├── skills/                     # 💡 自定义 Skills 示例
│   ├── codebuddy-doc-fetcher/  # 📥 文档获取 Skill
│   │   └── SKILL.md
│   ├── kb-doc-writer/          # ✍️ 知识库文档规范写入指导
│   │   └── SKILL.md
│   ├── kb-doc-reviewer/        # 🔍 知识库文档规范审查
│   │   └── SKILL.md
│   ├── kb-doc-quality/         # 📋 知识库文档质量检查
│   │   └── SKILL.md
│   └── cross-platform-skill-discovery/  # 🔎 跨平台技能发现
│       └── SKILL.md
└── agents/                     # 🤖 自定义 Agents 示例
    ├── codebuddy-doc-fetcher.md # 📥 文档获取 Agent
    ├── kb-doc-writer.md         # ✍️ 知识库文档规范写入 Agent
    ├── kb-doc-reviewer.md       # 🔍 知识库文档规范审查 Agent
    ├── kb-doc-quality.md        # 📋 知识库文档质量检查 Agent
    └── skill-ecosystem-explorer.md  # 🌍 技能生态探索 Agent
```

---

## 📄 核心文档

### [CodeBuddy扩展机制详解](./CodeBuddy扩展机制详解.md)

**内容摘要**：
- Skills 技能系统：模块化能力包
- Rules 规则系统：行为规范配置
- Agents 智能体：专业化 AI 助手
- 文件格式规范与示例
- 快速开始指南

---

## 🛠️ 使用指南

### 复制 Skills 到本地

```bash
# 复制 Skill 到用户目录
cp -r skills/codebuddy-doc-fetcher ~/.codebuddy/skills/
cp -r skills/kb-doc-writer ~/.codebuddy/skills/
cp -r skills/kb-doc-reviewer ~/.codebuddy/skills/
cp -r skills/kb-doc-quality ~/.codebuddy/skills/
cp -r skills/cross-platform-skill-discovery ~/.codebuddy/skills/

# 或复制到项目目录
cp -r skills/codebuddy-doc-fetcher <your-project>/.codebuddy/skills/
cp -r skills/kb-doc-writer <your-project>/.codebuddy/skills/
cp -r skills/kb-doc-reviewer <your-project>/.codebuddy/skills/
cp -r skills/kb-doc-quality <your-project>/.codebuddy/skills/
cp -r skills/cross-platform-skill-discovery <your-project>/.codebuddy/skills/
```

### 复制 Agents 到本地

```bash
# 复制 Agent 到用户目录
cp agents/codebuddy-doc-fetcher.md ~/.codebuddy/agents/
cp agents/kb-doc-writer.md ~/.codebuddy/agents/
cp agents/kb-doc-reviewer.md ~/.codebuddy/agents/
cp agents/kb-doc-quality.md ~/.codebuddy/agents/
cp agents/skill-ecosystem-explorer.md ~/.codebuddy/agents/

# 或复制到项目目录
cp agents/codebuddy-doc-fetcher.md <your-project>/.codebuddy/agents/
cp agents/kb-doc-writer.md <your-project>/.codebuddy/agents/
cp agents/kb-doc-reviewer.md <your-project>/.codebuddy/agents/
cp agents/kb-doc-quality.md <your-project>/.codebuddy/agents/
cp agents/skill-ecosystem-explorer.md <your-project>/.codebuddy/agents/
```

---

## 🔍 知识库文档质量管理体系

为知识库文档的全生命周期提供质量保障，包含三大核心工具：

### 1. [[kb-doc-writer|Skill - 文档规范写入指导]]

**功能**：指导用户按照知识库规范创建新文档

**适用场景**：
- 创建新文档时不确定放在哪里
- 不清楚文件命名格式
- 需要完整的元信息头模板
- 不知道如何更新MOC索引

### 2. [[kb-doc-reviewer|Skill - 文档规范审查]]

**功能**：审查现有文档是否符合规范，生成评分报告

**适用场景**：
- 检查文档是否符合规范
- 发现文档格式问题
- 生成合规性审查报告

### 3. [[kb-doc-quality|Skill - 文档质量检查]]

**功能**：检查文档歧义、错误、可读性等质量问题

**适用场景**：
- 检查文档是否有歧义表达
- 验证技术事实准确性
- 评估文档可读性
- 检查内容完整性

### 4. [[cross-platform-skill-discovery|Skill - 跨平台技能发现]]

**功能**：扫描并发现本地所有AI助手平台（CodeBuddy、OpenClaw、QClaw、Claude Desktop等）上安装的技能

**适用场景**：
- 想了解本地有哪些可用技能
- 创建新技能前检查是否重复
- 跨平台技能管理
- 了解AI助手生态全貌

**支持平台**：
| 平台 | 技能路径 |
|:---|:---|
| CodeBuddy | `~/.codebuddy/skills/`、`./.codebuddy/skills/` |
| OpenClaw | `~/.openclaw/skills/`、`~/.agents/skills/` |
| QClaw | `~/.qclaw/skills/` |
| Claude Desktop | `~/.claude/skills/` |

**报告维度**：
- 📁 按位置分类（技能安装在哪个目录）
- 🎯 按功能分类（文档类、质量类、工具类等）
- ⚠️ 重复检测（同名技能的不同安装位置）

---

## 🔗 相关资源

### 官方文档

| 文档 | 链接 |
|:---|:---|
| 官方首页 | https://www.codebuddy.ai/docs/zh/ide/Introduction |
| Skills 文档 | http://www.codebuddy.ai/docs/zh/ide/Features/Skills |
| Rules 文档 | http://www.codebuddy.ai/docs/zh/ide/User-guide/rules |
| Subagents 文档 | https://www.codebuddy.cn/docs/ide/Features/Subagents |

---

*索引更新：2026-04-18*
