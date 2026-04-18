---
name: cross-platform-skill-discovery
description: 跨平台AI助手技能发现工具。触发词：查看所有技能、技能目录、已安装技能、本地技能清单、AI工具有哪些、skill list、available skills、installed skills、查看skills、跨平台技能、技能生态、多平台技能、CodeBuddy技能、OpenClaw技能、QClaw技能、Claude技能
allowed-tools:
  - read_file
  - list_dir
  - search_file
  - web_search
  - web_fetch
disable: false
---

# 跨平台AI助手技能发现工具

## 🎯 功能概述

扫描并发现所有主流AI助手平台（CodeBuddy、OpenClaw、QClaw、Claude Desktop等）上安装的技能，以**位置**和**功能**双维度展示，帮助用户了解本地技能生态全貌，避免重复创建。

## 🔍 支持的平台

| 平台 | 技能目录路径 | 说明 |
|:---|:---|:---|
| **CodeBuddy** | `~/.codebuddy/skills/`、`./.codebuddy/skills/` | 用户级 + 项目级 |
| **OpenClaw** | `~/.openclaw/skills/`、`~/.agents/skills/`、`<workspace>/skills/` | 6层优先级 |
| **QClaw** | `~/.qclaw/skills/`、`~/.agents/skills/` | 基于OpenClaw |
| **Claude Desktop** | `~/.claude/skills/` | 官方Claude技能 |
| **自定义** | 用户指定的任意路径 | 支持自定义扩展 |

## 📊 扫描策略

### 1. 多路径自动探测
```
扫描顺序（按优先级）：
1. 当前项目目录 (.codebuddy/skills/, .openclaw/skills/)
2. 用户主目录 (~/.codebuddy/, ~/.openclaw/, ~/.qclaw/, ~/.claude/)
3. 工作区上级目录（向上搜索多层）
4. 环境变量配置的路径
5. 用户自定义输入路径
```

### 2. 配置文件读取
- `~/.openclaw/openclaw.json`：OpenClaw配置
- `~/.codebuddy/config.json`：CodeBuddy配置（如有）

### 3. 智能去重
- 按技能名称去重，显示所有安装位置
- 标记重复技能，提示冗余风险

## 📋 报告格式

### 按位置分类
```
📁 用户主目录 (~/.codebuddy/skills/)
├── kb-doc-writer       ✍️ 知识库文档写入
├── kb-doc-reviewer     🔍 知识库文档审查
└── ...

📁 项目目录 (./.codebuddy/skills/)
├── skill-discovery     🔎 跨平台技能发现
└── ...

📁 OpenClaw目录 (~/.openclaw/skills/)
├── web-search          🌐 网页搜索
├── file-manager        📁 文件管理
└── ...
```

### 按功能分类
```
🎯 质量保证类
├── kb-doc-writer       ✍️ 知识库文档写入
├── kb-doc-reviewer     🔍 知识库文档审查
├── kb-doc-quality       📋 文档质量检查
└── code-review         👁️ 代码审查

📥 文档工具类
├── codebuddy-doc-fetcher 📥 文档获取
└── ...

📊 数据分析类
├── finance-data-retrieval 📈 金融数据获取
└── ...

🔧 系统工具类
├── skill-discovery     🔎 跨平台技能发现
└── ...
```

### 冲突提醒
```
⚠️ 重复技能检测
- "file-manager" 存在于 OpenClaw 和 QClaw（功能可能相同）
- "code-review" 存在于 CodeBuddy 和 OpenClaw
建议：确认使用哪个版本，或合并功能
```

## 🔧 使用流程

### 交互式扫描
```
1. 问候 + 确认需求
   → "你好！要扫描本地的AI助手技能吗？"

2. 平台选择
   → "你想扫描哪些平台？"
   → 选项：[全部平台] [仅CodeBuddy] [仅OpenClaw] [自定义路径]

3. 执行扫描
   → 逐个探测各平台的标准路径
   → 读取找到的SKILL.md文件

4. 生成报告
   → 按位置分类展示
   → 按功能分类展示
   → 冲突检测和提醒

5. 后续建议
   → 如有重复，提供合并/选择建议
   → 如需创建新技能，提示先检查现有技能
```

### 报告输出示例
```
🌐 跨平台技能生态报告
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
扫描时间: 2026-04-18
扫描路径: 全部平台

📊 统计概览
├── 总技能数：28 个
├── 唯一技能：24 个
├── 重复技能：4 个
└── 覆盖平台：4 个

📁 按位置分类
├── CodeBuddy (8个)
│   ├── kb-doc-writer ✍️
│   ├── kb-doc-reviewer 🔍
│   └── ...
├── OpenClaw (12个)
│   ├── web-search 🌐
│   └── ...
├── QClaw (6个)
│   ├── wechat-integration 💬
│   └── ...
└── Claude Desktop (2个)

🎯 按功能分类
├── 文档类 (4个)
├── 质量类 (3个)
├── 数据类 (2个)
└── 工具类 (5个)

⚠️ 冲突提醒
├── file-manager: OpenClaw + QClaw
└── code-review: CodeBuddy + OpenClaw

💡 建议
如需创建新技能，建议先检查上述分类，
确保不与现有技能功能重复。
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 📚 参考文档

- [[MOC-CodeBuddy]]：CodeBuddy技能规范索引
- [[kb-doc-writer]]：知识库文档写入规范
- [[kb-doc-reviewer]]：知识库文档审查规范
