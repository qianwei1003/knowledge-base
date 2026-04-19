---
来源: 官方文档 (Google web.dev)
原始链接: https://web.developers.google.cn/articles/vitals
---

# Core Web Vitals 与前端性能优化指南

## 概述

Web Vitals 是 Google 发起的一项计划，旨在为提供出色用户体验的关键质量信号提供统一指导。Core Web Vitals 是 Web Vitals 的子集，适用于所有网页，所有网站所有者都应衡量这些指标，并将在所有 Google 工具中展示。

每个 Core Web Vitals 代表用户体验的一个独特方面，可在现场测量，并反映了以用户为中心的关键结果的真实世界体验。

---

## Core Web Vitals 三大核心指标

### 1. Largest Contentful Paint (LCP) — 最大内容绘制

- **衡量内容**：加载性能
- **目标值**：≤ 2.5 秒
- **含义**：衡量页面主要内容可能已完成加载时的感知加载速度
- **优化建议**：
  - 优化关键资源加载路径
  - 使用 CDN 加速静态资源
  - 优化图片（使用 WebP/AVIF 格式、响应式图片、懒加载）
  - 预加载关键资源 `<link rel="preload">`
  - 服务端渲染 (SSR) 加速首屏

### 2. Interaction to Next Paint (INP) — 交互到下次绘制

- **衡量内容**：交互性
- **目标值**：≤ 200 毫秒
- **含义**：评估用户与网页交互时的响应速度（已替代 FID）
- **优化建议**：
  - 减少主线程阻塞（代码分割、延迟加载非关键 JS）
  - 使用 Web Worker 处理计算密集型任务
  - 优化事件处理器，避免频繁重排重绘
  - 使用 `requestAnimationFrame` 和 `requestIdleCallback`
  - 减少第三方脚本影响

### 3. Cumulative Layout Shift (CLS) — 累积布局偏移

- **衡量内容**：视觉稳定性
- **目标值**：≤ 0.1
- **含义**：衡量页面整个生命周期内发生的意外布局偏移
- **优化建议**：
  - 为图片和嵌入内容设置明确的宽高（`width`/`height` 或 `aspect-ratio`）
  - 预留广告位空间
  - 避免在视口上方动态注入内容
  - 使用 CSS `contain` 属性
  - 字体预加载 `font-display: optional`

---

## 衡量阈值与评估标准

为确保大多数用户达到推荐目标，衡量阈值应取第 75 百分位的页面加载数据，并按移动端和桌面端分别统计。

评估 Core Web Vitals 合规性的工具应考虑：如果页面在所有三个 Core Web Vitals 指标的第 75 百分位都达到推荐目标，则该页面通过。

---

## Core Web Vitals 生命周期

指标在 Core Web Vitals 跟踪上经历三个阶段：

| 阶段 | 说明 |
|------|------|
| **Experimental（实验性）** | 前瞻性指标，可能根据测试和社区反馈进行重大更改 |
| **Pending（待定）** | 已通过测试和反馈阶段，有明确时间表成为稳定指标，至少保持 6 个月 |
| **Stable（稳定）** | 当前 Core Web Vitals 指标集，每年变更不超过一次 |

当前状态：LCP（稳定）、CLS（稳定）、INP（稳定，已替代 FID）

---

## 测量工具

### 现场工具（Field Tools）

| 工具 | 说明 |
|------|------|
| Chrome User Experience Report (CrUX) | 收集匿名真实用户数据 |
| Chrome DevTools | Performance 面板、Lighthouse |
| PageSpeed Insights | 在线性能检测 |
| Search Console | Core Web Vitals 报告 |

### JavaScript 测量

使用 [web-vitals](https://github.com/GoogleChrome/web-vitals) 库：

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  (navigator.sendBeacon && navigator.sendBeacon('/analytics', body)) ||
    fetch('/analytics', {body, method: 'POST', keepalive: true});
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
```

### 实验室工具（Lab Tools）

- Lighthouse（Chrome DevTools 集成）
- WebPageTest
- Chrome DevTools Performance 面板

---

## 其他 Web Vitals 指标

| 指标 | 含义 | 目标值 |
|------|------|--------|
| First Contentful Paint (FCP) | 首次内容绘制 | ≤ 1.8s |
| Time to First Byte (TTFB) | 首字节时间 | ≤ 800ms |
| First Input Delay (FID) | 首次输入延迟（已废弃） | ≤ 100ms |

---

## 前端性能优化最佳实践速查

### 资源加载优化
1. 启用 Gzip/Brotli 压缩（可减少 70% 响应体积）
2. 使用 CDN 分发静态资源
3. 合并减少 HTTP 请求数
4. 图片优化：WebP/AVIF 格式、响应式 `<picture>`、懒加载
5. 关键 CSS 内联，非关键 CSS 异步加载

### 渲染优化
1. CSS 放 `<head>`，JS 放 `</body>` 前或加 `defer`/`async`
2. 避免阻塞渲染的资源
3. 减少重排重绘，使用 CSS `transform`/`opacity` 动画
4. 虚拟滚动处理大列表
5. 使用 `content-visibility: auto` 延迟渲染屏幕外内容

### 缓存策略
1. 静态资源设置长期缓存 + 内容哈希文件名
2. 合理设置 `Cache-Control` 和 `ETag`
3. 使用 Service Worker 离线缓存
4. 无 Cookie 域名分发静态资源

### 代码优化
1. 代码分割（Code Splitting）和按需加载
2. Tree Shaking 移除未使用代码
3. 预编译模板，避免运行时编译
4. 防抖（debounce）和节流（throttle）高频事件
5. 事件委托减少事件监听器数量

---

## 参考链接

- [Web Vitals 官方文档](https://web.developers.google.cn/articles/vitals)
- [LCP 优化指南](https://web.developers.google.cn/articles/lcp)
- [INP 优化指南](https://web.developers.google.cn/articles/inp)
- [CLS 优化指南](https://web.developers.google.cn/articles/cls)
- [web-vitals JavaScript 库](https://github.com/GoogleChrome/web-vitals)
- [PageSpeed Insights](https://pagespeed.web.dev)
