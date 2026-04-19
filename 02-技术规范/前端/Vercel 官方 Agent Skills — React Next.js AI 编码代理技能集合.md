---
来源: GitHub (vercel-labs/agent-skills)
原始链接: https://github.com/vercel-labs/agent-skills
---

# Vercel 官方 Agent Skills — AI 编码代理技能集合

## 核心要点

- Vercel 官方维护的 AI Agent 技能包，包含性能优化 40+ 规则、UI 审查 100+ 规则
- 安装：`npx skills add vercel-labs/agent-skills`，每个 Skill 由 SKILL.md + 可选脚本/参考文档组成
- 六大技能模块：React/Next.js 性能优化、UI 最佳实践、React Native、View Transition API、组合模式、一键部署

## 技能模块

### 1. React & Next.js 性能优化 (react-best-practices)

按影响程度分级：
- **Critical** — 消除瀑布流（并行化数据获取）、包体积优化
- **High** — 服务端性能（减少服务端计算）
- **Medium-High** — 客户端数据获取策略
- **Medium** — 重渲染优化（React.memo/useMemo）、渲染性能、JS 微优化

### 2. UI 最佳实践审查 (ui-best-practices)

100+ 规则覆盖 10 个类别：

| 类别 | 关键检查项 |
|------|-----------|
| 可访问性 | aria-labels、语义化 HTML、键盘交互 |
| 焦点状态 | 可见焦点、focus-visible 模式 |
| 表单 | autocomplete、验证、错误处理 |
| 动画 | prefers-reduced-motion、合成器友好变换 |
| 排版 | 弯引号、省略号、等宽数字(tabular-nums) |
| 图片 | 尺寸、懒加载、alt 文本 |
| 性能 | 虚拟化、避免 layout thrashing、preconnect |
| 导航 | URL 反映状态、深链接 |
| 暗色模式 | color-scheme、theme-color meta |
| 触摸交互 | touch-action、tap-highlight |

触发方式：`"Review my UI"` / `"Check accessibility"` / `"Audit design"`

### 3. React Native 最佳实践 (16 条规则)

- 性能(Critical)：FlashList、memoization、重计算
- 布局(High)：flex 模式、安全区域、键盘处理
- 动画(High)：Reanimated、手势处理
- 图片(Medium)：expo-image、缓存、懒加载
- 状态(Medium)：Zustand 模式、React Compiler
- 架构(Medium)：monorepo、导入管理

### 4. View Transition API

- `<ViewTransition>` 组件支持 enter/exit/update/share 四种触发器
- `addTransitionType` 实现方向性导航动画（前进/后退）
- 内置 CSS 配方：fade、slide、scale、flip
- Next.js `next/link` 支持 `transitionTypes` prop

### 5. React 组合模式

- 提取复合组件、状态提升减少 props、组合内部增加灵活性
- 避免布尔属性膨胀（boolean props 过多）

### 6. Vercel 部署

- 自动检测 40+ 框架（通过 package.json）
- 返回预览 URL（实时站点）和认领 URL（转移所有权）
- 上传时排除 node_modules 和 .git