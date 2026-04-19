# 市场调研：股票/金融相关 Skills

> 调研时间: 2026-04-19
> 调研目的: 避免重复造轮子，了解已有能力边界

---

## 市场上已有的 Skills

### 1. 金融数据层

| 插件 | Skill | 功能 | 触发场景 |
|------|-------|------|----------|
| **finance-data** | [[neodata-financial-search]] | 自然语言金融数据搜索（A股/港股/美股/基金/宏观/外汇/大宗商品） | 任何金融数据查询优先使用 |
| **finance-data** | [[finance-data-retrieval]] | 结构化 API 数据检索（209 个接口，15 大类） | 需要精确结构化数据时 |

**覆盖范围**：
- 股票数据（98 个 API）：行情、财报、资金流向、龙虎榜等
- 指数专题（19 个 API）：成分股、权重、行业分布
- 期货/债券/ETF/期权数据
- 宏观经济（13 个 API）：GDP、CPI、PPI、利率等

---

### 2. 投资分析层

| 插件 | Skill | 功能 | 特点 |
|------|-------|------|------|
| **trading-agent** | [[trading-analysis]] | 多角色辩论式投资分析 | **11 个专业 Agent，5 阶段工作流** |

**工作流程**：
```
Phase 1: 数据收集（并行）
  ├─ market-analyst（技术分析）
  ├─ fundamentals-analyst（基本面）
  ├─ news-analyst（新闻动态）
  └─ sentiment-analyst（资金流向）

Phase 2: 投资辩论（顺序）
  ├─ bull-researcher（多头论证）
  ├─ bear-researcher（空头论证）
  └─ research-manager（投资计划）

Phase 3: 交易决策
  └─ trader（交易方案）

Phase 4: 风险评估（并行+顺序）
  ├─ aggressive-risk-analyst（激进派）
  ├─ conservative-risk-analyst（保守派）
  ├─ neutral-risk-analyst（中性派）
  └─ risk-manager（风险裁决）

Phase 5: 最终报告
  └─ orchestrator 整合输出
```

**输出**：完整投资分析报告（BUY/SELL/HOLD + 仓位建议 + 风险提示）

---

### 3. 量化工具层

| 插件 | Skill | 功能 | 适用场景 |
|------|-------|------|----------|
| **quantitative-trading** | [[backtesting-frameworks]] | 回测框架 | 策略回测、绩效评估 |
| **quantitative-trading** | [[risk-metrics-calculation]] | 风险指标计算 | VaR、CVaR、Sharpe、Sortino |

**backtesting-frameworks 包含**：
- 事件驱动回测器（Event-Driven）
- 向量化回测器（Vectorized - 快速）
- Walk-Forward 优化
- 蒙特卡洛分析
- 完整的绩效指标计算

**risk-metrics-calculation 包含**：
- VaR（历史法、参数法、Cornish-Fisher）
- CVaR / Expected Shortfall
- 回撤分析（最大回撤、平均回撤、回撤持续时间）
- 风险调整收益（Sharpe、Sortino、Calmar、Omega）
- 组合风险（边际风险贡献、风险平价）
- 压力测试（历史危机情景、假设情景）

---

## 能力对比：我的 buy-signal-analyzer vs 市场现有

| 维度 | 我的 buy-signal-analyzer | trading-analysis | 差异分析 |
|------|------------------------|------------------|----------|
| **定位** | 买入信号评级系统 | 完整投资分析工作流 | 我聚焦买入时机，它覆盖全流程 |
| **角色** | 单一 Agent | 11 个专业 Agent | 它更全面专业 |
| **输出** | 信号评分（0-100）+ 简评 | 完整报告 + 操作建议 | 我简洁快速，它详尽深入 |
| **深度** | 四维度评分（趋势/资金/情绪/形态） | 五阶段辩论 + 风险评估 | 它更深更系统 |
| **速度** | 快（直接计算） | 慢（多 Agent 协作） | 我有速度优势 |
| **场景** | 快速筛选、择时参考 | 深度投资决策 | 场景不同 |

---

## 关键发现

### ✅ 已有能力非常完善

市场上的 skills 已经形成了**完整的三层架构**：

```
┌─────────────────────────────────────────────────────┐
│  应用层: trading-analysis (投资分析决策)            │
├─────────────────────────────────────────────────────┤
│  数据层: neodata-financial-search (金融数据获取)    │
├─────────────────────────────────────────────────────┤
│  工具层: backtesting-frameworks / risk-metrics      │
│          (回测验证 / 风险计算)                      │
└─────────────────────────────────────────────────────┘
```

### ❌ 我的 skill 与 trading-analysis 功能重叠

两者都是输出 BUY/SELL/HOLD 建议，但：
- `trading-analysis` 是**专业级**的，11 个 Agent 协作
- `buy-signal-analyzer` 是**轻量级**的，单 Agent 快速评分

### 🤔 可能的差异化定位

如果保留 `buy-signal-analyzer`，可以差异化定位：

| 定位 | 描述 | 触发词 |
|------|------|--------|
| **快速筛选器** | "扫描机器人板块买入信号"、"快速看一眼能不能买" | 扫描、快速筛选、量化选股 |
| **信号验证器** | 对 trading-analysis 结果进行二次验证 | 验证、交叉检查 |
| **多股对比** | 同时分析多只股票信号强度排名 | 对比、排名、选股 |

---

## 建议

### 方案 A：废弃 buy-signal-analyzer（推荐）

理由：
1. `trading-analysis` 已经覆盖了买入分析场景
2. 质量更高（11 个专业 Agent）
3. 维护成本更低（用官方 skill）

### 方案 B：重新定位 buy-signal-analyzer

改为**快速筛选工具**：
- 不输出完整分析，只输出信号评分
- 支持批量股票扫描（如"机器人板块哪些有买入信号"）
- 作为 trading-analysis 的前置筛选器

### 方案 C：整合到 trading-analysis

将自己开发的信号算法贡献为 trading-analysis 的一个 Agent 或模块。

---

## 参考资料

- `cb_teams_marketplace/plugins/finance-data/skills/neodata-financial-search/SKILL.md`
- `cb_teams_marketplace/plugins/finance-data/skills/finance-data-retrieval/`
- `cb_teams_marketplace/plugins/trading-agent/skills/trading-analysis/SKILL.md`
- `codebuddy-plugins-official/external_plugins/quantitative-trading/skills/backtesting-frameworks/SKILL.md`
- `codebuddy-plugins-official/external_plugins/quantitative-trading/skills/risk-metrics-calculation/SKILL.md`
