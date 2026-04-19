# 市场调研：股票/金融相关 Skills（完整版）

> 调研时间: 2026-04-19
> 调研范围: CodeBuddy 官方插件市场 + cb_teams_marketplace
> 调研目的: 全面了解已有能力，避免重复造轮子

---

## 一、数据层 Skills

### 1. A股/中国金融数据

| 插件 | Skill | 功能 | 覆盖范围 |
|------|-------|------|----------|
| **finance-data** | [[neodata-financial-search]] | 自然语言金融数据搜索 | A股/港股/美股/基金/宏观/外汇/大宗商品 |
| **finance-data** | [[finance-data-retrieval]] | 结构化 API 数据检索 | **209 个接口，15 大类** |

**finance-data-retrieval 详细覆盖**（98 个股票相关 API）：

| 类别 | API 数量 | 示例 |
|------|----------|------|
| 基础数据 | 10+ | 股票列表、曾用名、ST 列表、沪深港通 |
| 行情数据 | 20+ | 日线/周线/月线、分钟线、复权因子 |
| 财务数据 | 15+ | 利润表、资产负债表、现金流量表、财务指标 |
| 市场数据 | 15+ | 涨停股、龙虎榜、资金流向、板块异动 |
| 参考数据 | 10+ | 股票回购、分红送股、股权激励 |
| 打板专题 | 10+ | 连板天梯、板块统计、涨停原因 |

---

## 二、投资分析层 Skills

### 1. 多角色辩论式投资分析

| 插件 | Skill | 功能 | 特点 |
|------|-------|------|------|
| **trading-agent** | [[trading-analysis]] | **11 个 Agent 协作投资分析** | 5 阶段工作流，输出 BUY/SELL/HOLD |

**工作流程**：
```
Phase 1: 数据收集（并行）
  ├─ market-analyst（技术分析）
  ├─ fundamentals-analyst（基本面分析 - 见下方）
  ├─ news-analyst（新闻动态）
  └─ sentiment-analyst（资金流向）

Phase 2: 投资辩论（顺序）
  ├─ bull-researcher（多头论证）
  ├─ bear-researcher（空头论证）
  └─ research-manager（投资计划裁决）

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

### 2. 股票研究（Equity Research）- 专业研报级别

| 插件 | Skill | 功能 | 输出 |
|------|-------|------|------|
| **equity-research** | [[earnings-analysis]] | 季度财报分析报告 | 8-12 页 DOCX 研报 |
| **equity-research** | [[earnings-preview]] | 财报前瞻/预览 | 简短前瞻性分析 |
| **equity-research** | [[initiating-coverage]] | 首次覆盖报告 | 30-50 页深度研报 |
| **equity-research** | [[sector-overview]] | 行业/板块研究报告 | 5-30 页行业分析 |
| **equity-research** | [[idea-generation]] | 系统化选股/idea 挖掘 | 5-10 个 idea 清单 |
| **equity-research** | [[catalyst-calendar]] | 催化剂日历管理 | Excel 事件追踪表 |
| **equity-research** | [[thesis-tracker]] | 投资 thesis 追踪 | - |
| **equity-research** | [[morning-note]] | 晨会笔记/早报 | - |
| **equity-research** | [[model-update]] | 财务模型更新 | - |

---

### 3. 卖方/机构级分析（LSEG - 伦敦证券交易所集团）

| 插件 | Skill | 功能 | 适用场景 |
|------|-------|------|----------|
| **lseg** | [[equity-research]] | IBES 一致预期 + 基本面 + 估值 | 机构级股票研究 |
| **lseg** | [[option-vol-analysis]] | 期权波动率分析（Greeks、隐波 vs 实波）| 期权定价、波动率交易 |
| **lseg** | [[fx-carry-trade]] | 外汇套利交易分析（Carry-to-Vol）| 外汇套利策略 |
| **lseg** | [[bond-relative-value]] | 债券相对价值分析（Spread 分解）| 债券投资 |
| **lseg** | [[swap-curve-strategy]] | 利率互换曲线策略 | 利率交易 |
| **lseg** | [[macro-rates-monitor]] | 宏观利率监控面板 | 宏观策略 |
| **lseg** | [[fixed-income-portfolio]] | 固收组合管理 | 债券组合 |
| **lseg** | [[bond-futures-basis]] | 国债期货基差分析 | 期货套利 |

---

## 三、量化工具层 Skills

### 1. 回测与策略验证

| 插件 | Skill | 功能 | 特点 |
|------|-------|------|------|
| **quantitative-trading** | [[backtesting-frameworks]] | **回测框架** | 事件驱动/向量化/Walk-Forward/蒙特卡洛 |
| **quantitative-trading** | [[risk-metrics-calculation]] | **风险指标计算** | VaR、CVaR、Sharpe、Sortino、压力测试 |

**backtesting-frameworks 包含**：
- 事件驱动回测器（Event-Driven）
- 向量化回测器（Vectorized - 快速）
- Walk-Forward 优化（避免过拟合）
- 蒙特卡洛分析（Bootstrap 模拟）
- 完整绩效指标计算

**risk-metrics-calculation 包含**：
- VaR（历史法、参数法、Cornish-Fisher）
- CVaR / Expected Shortfall
- 回撤分析（最大回撤、平均回撤、回撤持续时间）
- 风险调整收益（Sharpe、Sortino、Calmar、Omega）
- 组合风险（边际风险贡献、风险平价）
- 压力测试（历史危机情景、假设情景）

---

### 2. 量化分析师 Agent

| 插件 | Agent | 功能 |
|------|-------|------|
| **agents-business-finance** | [[quant-analyst]] | 量化策略开发、回测、风险分析、组合优化、统计套利 |

**能力范围**：
- 开发和回测量化交易策略
- 实现风险指标（VaR、Sharpe、最大回撤）
- 组合优化模型（Markowitz、Black-Litterman）
- 时间序列分析和预测模型
- 期权定价和 Greeks 计算
- 统计套利和配对交易系统

---

## 四、财富管理 Skills

| 插件 | Skill | 功能 | 适用场景 |
|------|-------|------|----------|
| **wealth-management** | [[portfolio-rebalance]] | 投资组合再平衡 | 资产配置偏离调整 |
| **wealth-management** | [[tax-loss-harvesting]] | 税务亏损收割 | 税务优化 |
| **wealth-management** | [[investment-proposal]] | 投资方案建议书 | 客户提案 |
| **wealth-management** | [[financial-plan]] | 财务规划 | 长期财富规划 |
| **wealth-management** | [[client-report]] | 客户报告生成 | 定期报告 |
| **wealth-management** | [[client-review]] | 客户回顾会议 | 定期回顾 |

---

## 五、私募股权（Private Equity）Skills

| 插件 | Skill | 功能 | 适用场景 |
|------|-------|------|----------|
| **private-equity** | [[deal-screening]] | 交易初筛 | 快速筛选投资项目 |
| **private-equity** | [[deal-sourcing]] | 交易 sourcing | 寻找投资机会 |
| **private-equity** | [[returns-analysis]] | 收益分析 | IRR/MOIC 敏感性分析 |
| **private-equity** | [[dd-checklist]] | 尽职调查清单 | DD 流程管理 |
| **private-equity** | [[dd-meeting-prep]] | DD 会议准备 | 专家访谈准备 |
| **private-equity** | [[ic-memo]] | 投委会备忘录 | IC 材料撰写 |
| **private-equity** | [[portfolio-monitoring]] | 投后监控 | 已投项目管理 |
| **private-equity** | [[unit-economics]] | 单位经济模型 | SaaS 等商业模式分析 |
| **private-equity** | [[value-creation-plan]] | 价值创造计划 | 投后增值服务 |

---

## 六、加密数字货币

| 插件 | Agent | 功能 |
|------|-------|------|
| **agents-crypto-trading** | [[crypto-analyst]] | 加密资产分析 |
| **agents-crypto-trading** | [[crypto-trader]] | 加密交易执行 |
| **agents-crypto-trading** | [[crypto-risk-manager]] | 加密风险管理 |

---

## 七、能力对比与定位建议

### 已有 Skills 覆盖矩阵

| 需求场景 | 推荐 Skill | 理由 |
|----------|-----------|------|
| "能不能买这只股票" | `trading-analysis` | 最全面，11 个 Agent 协作 |
| "快速看一下基本面" | `fundamentals-analyst` | 专门的基本面分析 Agent |
| "季度财报分析" | `earnings-analysis` | 专业研报格式 |
| "找投资机会" | `idea-generation` | 系统化选股 |
| "回测策略" | `backtesting-frameworks` | 专业回测框架 |
| "计算风险指标" | `risk-metrics-calculation` | 完整风险指标库 |
| "期权分析" | `option-vol-analysis` | 隐波/实波/Greeks |
| "宏观利率分析" | `macro-rates-monitor` | 宏观面板 |
| "投资组合再平衡" | `portfolio-rebalance` | 税感知再平衡 |
| "PE 投资分析" | `deal-screening` + `returns-analysis` | PE 专用工具 |

---

### buy-signal-analyzer 的差异化定位（如果保留）

| 维度 | trading-analysis | buy-signal-analyzer（建议定位） |
|------|------------------|--------------------------------|
| **深度** | 深度（11 Agent 辩论） | 快速（单 Agent 评分） |
| **速度** | 慢（多轮协作） | 快（直接计算） |
| **场景** | 重大投资决策 | 快速筛选、批量扫描 |
| **输出** | 完整报告 | 信号评分 + 简评 |
| **批量** | 单股深度 | 多股并行扫描 |

**建议触发词**：
- "扫描机器人板块买入信号"
- "快速筛选值得买的股票"
- "这几只股票信号强度排名"
- "量化选股"

---

## 八、参考资料索引

### trading-agent
```
cb_teams_marketplace/plugins/trading-agent/
├── skills/trading-analysis/SKILL.md
└── agents/
    ├── market-analyst.md
    ├── fundamentals-analyst.md
    ├── news-analyst.md
    ├── sentiment-analyst.md
    ├── bull-researcher.md
    ├── bear-researcher.md
    ├── research-manager.md
    ├── trader.md
    ├── aggressive-risk-analyst.md
    ├── conservative-risk-analyst.md
    ├── neutral-risk-analyst.md
    └── risk-manager.md
```

### equity-research
```
cb_teams_marketplace/plugins/equity-research/skills/
├── earnings-analysis/SKILL.md
├── earnings-preview/SKILL.md
├── initiating-coverage/SKILL.md
├── sector-overview/SKILL.md
├── idea-generation/SKILL.md
├── catalyst-calendar/SKILL.md
├── thesis-tracker/SKILL.md
├── morning-note/SKILL.md
└── model-update/SKILL.md
```

### lseg
```
cb_teams_marketplace/plugins/lseg/skills/
├── equity-research/SKILL.md
├── option-vol-analysis/SKILL.md
├── fx-carry-trade/SKILL.md
├── bond-relative-value/SKILL.md
├── swap-curve-strategy/SKILL.md
├── macro-rates-monitor/SKILL.md
├── fixed-income-portfolio/SKILL.md
└── bond-futures-basis/SKILL.md
```

### quantitative-trading
```
codebuddy-plugins-official/external_plugins/quantitative-trading/skills/
├── backtesting-frameworks/SKILL.md
└── risk-metrics-calculation/SKILL.md
```

---

## 九、总结

### 市场覆盖情况

| 领域 | 覆盖度 | 评价 |
|------|--------|------|
| A股数据 | ✅ 完整 | finance-data 插件 209 个 API |
| 投资分析 | ✅ 完整 | trading-analysis 11 Agent 协作 |
| 股票研究 | ✅ 完整 | equity-research 9 个 skills |
| 量化回测 | ✅ 完整 | backtesting-frameworks |
| 风险管理 | ✅ 完整 | risk-metrics-calculation |
| 财富管理 | ✅ 完整 | wealth-management 6 个 skills |
| 私募股权 | ✅ 完整 | private-equity 9 个 skills |
| 固定收益 | ✅ 完整 | lseg 8 个 skills |
| 期权衍生品 | ✅ 完整 | option-vol-analysis |
| 加密货币 | ✅ 完整 | agents-crypto-trading |

### 建议

1. **不要创建新的买入信号 skill** —— `trading-analysis` 已经覆盖
2. **如需保留 buy-signal-analyzer**，重新定位为**快速批量筛选工具**
3. **直接使用官方 skills** —— 质量高、维护好、生态完整
4. **如需扩展**，考虑：
   - A股特有的技术分析指标（如筹码分布、龙虎榜游资分析）
   - 涨停板战法专用工具
   - 个股与板块联动分析

---

> 文档版本: 1.0
> 最后更新: 2026-04-19
