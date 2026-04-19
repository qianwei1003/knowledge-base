# AI-Skill创建工作流-反思与总结

> 创建时间: 2026-04-19
> 关联: [[Agent设计模式]], [[skill-creator]]
> 状态: ✅ 已修复

---

## 产出背景

在创建 `buy-signal-analyzer` skill 时，违反了官方规范和标准工作流。经过用户指正，发现了系统性失误：不知道如何正确创建 skill。

---

## 问题总结

### ❌ 犯的错误

| 问题 | 详情 |
|------|------|
| **未使用标准工作流** | 不知道有 `skill-creator` 工作流程，直接凭直觉写代码和文档 |
| **未遵循 SKILL.md 标准格式** | `description` 多行冗长、缺少 `allowed-tools`、内容过于详细 |
| **未创建测试用例** | 缺少 `evals/evals.json` 进行验证 |
| **未参考已有 skill** | 没有先阅读 2-3 个标准 skill 作为参考 |

### 📋 正确的 skill-creator 工作流（官方）

```
1. Capture Intent     → 理解用户意图、触发场景、期望输出
2. Interview          → 提问研究、边缘情况、依赖关系  
3. Write SKILL.md     → 遵循标准格式（name/description/allowed-tools）
4. Create Test Cases  → 创建 2-3 个真实测试用例，保存到 evals/evals.json
5. Run & Evaluate     → 并行运行 with-skill 和 baseline 对比
6. Review             → 用 eval-viewer 生成可视化报告
7. Iterate            → 根据反馈改进 skill
8. Description Optimization → 优化触发词
```

### 📋 官方 SKILL.md 标准格式

```yaml
---
name: skill-name
description: 一句话简短描述
allowed-tools: [Read, Write, Bash, ExecuteCommand]
---
```

**必须字段**：description（一句话）
**可选字段**：name, allowed-tools

### 📁 标准 Skill 目录结构

```
skill-name/
├── SKILL.md          # 必需：简洁主文件（≈60行以内）
├── reference.md      # 可选：详细参考文档（代码实现细节）
├── scripts/          # 可选：辅助脚本
└── tests/            # 可选：测试用例
```

---

## 关键教训

### 1. 创建 Skill 前必须先做

- [ ] 加载 `skill-creator` skill，使用官方工作流
- [ ] 阅读 `skills-marketplace/skills-standard.md` 规范
- [ ] 参考 2-3 个已有标准 skill（如 `personal-knowledge-base`、`akshare-stock`）

### 2. SKILL.md 主文件应该

- [ ] `description` 必须是**一句话**
- [ ] `allowed-tools` 声明允许的工具
- [ ] 保持简洁（≈60行以内），代码细节移至 `reference.md`
- [ ] 包含「何时激活」触发场景

### 3. 创建后必须做

- [ ] 创建测试用例 `evals/evals.json`
- [ ] 运行评估对比
- [ ] 根据反馈迭代改进

---

## 相关文档

- 官方规范：`skills-marketplace/skills-standard.md`
- 官方工作流：`plugins/marketplaces/codebuddy-plugins-official/external_plugins/skill-creator/SKILL.md`
- 标准 skill 参考：`personal-knowledge-base`、`akshare-stock`

---

## 反思

**问题根源**：AI 在创建 skill 时没有主动查找和使用标准工作流和规范文档，而是凭直觉先写代码、后补文档。

**正确做法**：
1. 开始任何 skill 相关工作前，先加载 `skill-creator` skill
2. 遵循 `skills-standard.md` 的格式要求
3. 参考已有标准 skill 的结构

---

*来源: #来源/Claude | 日期: 2026-04-19*
