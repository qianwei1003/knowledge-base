---
来源: GitHub
领域: 前端
关键词: Vercel, Agent Skills, React, Next.js, 性能优化, 可访问性, 部署
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://github.com/vercel-labs/agent-skills
---

# Vercel 官方 Agent Skills — AI 编码代理技能集合

> Vercel 官方推出的 AI 编码代理技能包，遵循 [Agent Skills](https://agentskills.io/) 格式，为 Claude、Copilot 等 AI Agent 提供结构化的 React/Next.js 最佳实践指南。包含 40+ 条性能优化规则、100+ 条 UI 审查规则。

## 概述

vercel-labs/agent-skills 是 Vercel 工程团队官方维护的 AI 编码代理技能集合。每个 Skill 包含：
- **SKILL.md** — 代理指令文件
- **scripts/** — 辅助自动化脚本（可选）
- **references/** — 参考文档（可选）

安装方式：
```bash
npx skills add vercel-labs/agent-skills
```

## 技能列表

### 1. React & Next.js 性能优化 (react-best-practices)

**优先级分类：**
- 🔴 **Eliminating Waterfalls（消除瀑布流）** — Critical
- 🔴 **Bundle Size Optimization（包体积优化）** — Critical
- 🟠 **Server-side Performance（服务端性能）** — High
- 🟡 **Client-side Data Fetching（客户端数据获取）** — Medium-High
- 🟢 **Re-render Optimization（重渲染优化）** — Medium
- 🟢 **Rendering Performance（渲染性能）** — Medium
- 🟢 **JavaScript Micro-optimizations（JS 微优化）** — Low-Medium

**使用场景：**
- 编写新 React 组件或 Next.js 页面
- 实现客户端/服务端数据获取
- 代码审查中的性能问题检查
- 优化包体积或加载时间

### 2. UI 最佳实践审查 (ui-best-practices)

覆盖 **100+ 规则**，按类别划分：

| 类别 | 检查内容 |
|------|---------|
| **可访问性** | aria-labels、语义化 HTML、键盘交互 |
| **焦点状态** | 可见焦点、focus-visible 模式 |
| **表单** | autocomplete、验证、错误处理 |
| **动画** | prefers-reduced-motion、合成器友好变换 |
| **排版** | 弯引号、省略号、等宽数字(tabular-nums) |
| **图片** | 尺寸、懒加载、alt 文本 |
| **性能** | 虚拟化、布局抖动(layout thrashing)、preconnect |
| **导航与状态** | URL 反映状态、深链接 |
| **暗色模式与主题** | color-scheme、theme-color meta |
| **触摸与交互** | touch-action、tap-highlight |
| **国际化** | Intl.DateTimeFormat、Intl.NumberFormat |

**使用场景：**
- "Review my UI"
- "Check accessibility"
- "Audit design"
- "Check my site against best practices"

### 3. React Native 最佳实践 (react-native-best-practices)

包含 **16 条规则**，覆盖 7 个方面：

- 🏎️ **性能（Critical）** — FlashList、memoization、重计算
- 📐 **布局（High）** — flex 模式、安全区域、键盘处理
- 🎬 **动画（High）** — Reanimated、手势处理
- 🖼️ **图片（Medium）** — expo-image、缓存、懒加载
- 📦 **状态管理（Medium）** — Zustand 模式、React Compiler
- 🏗️ **架构（Medium）** — monorepo 结构、导入管理
- 📱 **平台（Medium）** — iOS/Android 特定模式

### 4. View Transition API (view-transitions)

React 的视图过渡动画 API 指南：

**核心概念：**
- `<ViewTransition>` 组件（enter、exit、update、share 触发器）
- `addTransitionType` 用于方向性/上下文特定动画
- View Transition 类和 CSS 伪元素
- 共享元素过渡（列表到详情的形变动画）
- Next.js 中 `next/link` 的 `transitionTypes` prop

**覆盖场景：**
- 页面过渡或路由动画
- 组件进入/退出动画
- 共享元素过渡（列表到详情变形）
- 方向性（前进/后退）导航动画
- 列表重排或 Suspense fallback 显示动画
- 无障碍访问（prefers-reduced-motion）

**内置 CSS 动画配方：** fade、slide、scale、flip

### 5. React 组合模式 (composition-patterns)

帮助避免布尔属性膨胀：

**核心模式：**
- 提取复合组件
- 状态提升以减少 props
- 组合内部以增加灵活性
- 避免 prop drilling

**使用场景：**
- 重构有大量布尔属性（boolean props）的组件
- 构建可复用的组件库
- 设计灵活的 API
- 审查组件架构

### 6. Vercel 部署 (deploy-to-vercel)

一键部署到 Vercel 的技能：

**特性：**
- 自动检测 40+ 框架（通过 package.json）
- 返回预览 URL（实时站点）和认领 URL（转移所有权）
- 自动处理静态 HTML 项目
- 上传时排除 node_modules 和 .git

**工作流程：**
1. 将项目打包为 tarball
2. 检测框架（Next.js、Vite、Astro 等）
3. 上传到部署服务
4. 返回预览 URL 和认领 URL

## 总结

Vercel Agent Skills 的核心价值在于将 **工程团队多年积累的最佳实践** 结构化为 AI Agent 可消化的格式。这不是简单的 prompt，而是经过分类、优先级排序、可执行的规则体系。

- **性能优化** — 40+ 规则按影响程度分级，从消除瀑布流到 JS 微优化
- **UI 审查** — 100+ 规则覆盖可访问性、性能、UX 三大维度
- **跨平台** — 同时覆盖 Web（React/Next.js）和 Mobile（React Native）
- **部署集成** — 与 Vercel 平台深度集成，支持一键部署

## 参考链接

- 仓库地址: https://github.com/vercel-labs/agent-skills
- Agent Skills 格式规范: https://agentskills.io/
- 安装: `npx skills add vercel-labs/agent-skills`
- 许可证: MIT
