# CodeBuddy Plugins 经验沉淀

> 本文档记录 CodeBuddy 插件使用的关键经验，避免重复踩坑。

## finance-data 插件

### token 获取方式

**直接运行脚本即可自动获取 token，无需手动传入。**

```bash
python "C:\Users\60597\.codebuddy\plugins\marketplaces\cb_teams_marketplace\plugins\finance-data\skills\neodata-financial-search\scripts\query.py" --query "查询内容"
```

- `neodata-financial-search`：自然语言金融搜索，支持股票、基金、宏观等数据
- `finance-data-retrieval`：无独立脚本，需通过 neodata 脚本调用

### 常见数据查询示例

| 查询内容 | 命令 |
|---------|------|
| 股票行情 | `--query "贵州茅台股价"` |
| 历史日线 | `--query "贵州茅台2025年4月日线数据"` |
| 资金流向 | `--query "贵州茅台资金流向"` |
| 指数成分 | `--query "沪深300指数成分股权重"` |
| 基金净值 | `--query "沪深300ETF基金净值"` |
| 宏观利率 | `--query "最新Shibor利率"` |
| 涨停榜 | `--query "今日涨停股票"` |

---

## 更新记录

| 日期 | 内容 |
|------|------|
| 2026-04-19 | 记录 finance-data token 获取方式 |
