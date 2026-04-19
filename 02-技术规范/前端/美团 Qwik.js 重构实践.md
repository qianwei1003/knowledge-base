---
来源: 美团技术团队官方博客
原始链接: https://tech.meituan.com/2026/03/13/qwik-practice-in-dianping.html
---

# 大众点评 M 站基于 Qwik.js 的重构实践

## 核心要点

- 传统 SSR 水合瓶颈 - Qwik Resumability：HTML 序列化状态，客户端无需水合，按需加载代码
- 首屏 JS 体积从 2MB 降至 <10KB（降幅超 99%），Qwikloader 仅 <1KB
- 混合加载策略：SSR 直出 L0 级 POI 基础数据 + CSR 渐进加载 L1 级动态数据

## Qwik vs 传统 SSR

```
传统 SSR：下载 JS -> 解析执行 -> 生成虚拟 DOM -> 对比绑定 -> 激活交互
Qwik：    HTML 已含状态 -> Qwikloader(<1KB) 监听事件 -> 交互时按需加载代码块
```

关键差异：Qwik 以 `$` 标记自动提取独立 chunk，按事件/交互级别而非组件级别拆分。

## 架构设计

### 混合加载策略
- **L0 SSR 直出**：POI 基础信息（店名、星级、地址、相册），服务端并行聚合
- **L1 CSR 渐进**：商家券、评价、问答等，通过 `useVisibleTask$` 钩子延迟加载

### 部署架构
- Serverless 平台复用请求拦截、日志监控、容灾、负载均衡
- 自定义 Adapter 适配 Express/Fastify
- undici 替代 axios，跨服务请求耗时降低 20%+
- 路由层裁剪：页面粒度部署减少匹配开销

### 容错降级
1. SSR 接口超时 -> 熔断后客户端重试
2. Serverless 渲染失败 -> 回退 SSG 静态产物
3. 低端浏览器 -> 重定向旧版 M 站

## 性能优化手段

| 手段 | 效果 |
|------|------|
| 公共 CSS 内联到 head | FCP TP90 降低 100+ms |
| LCP 图片 Preload + fetchPriority="high" | LCP 平均下降 300+ms |
| AVIF 格式图片 | 资源体积降低 80% |
| BFF 聚合层统一封装 | 后端接口 TP95 降低 31.6% |
| 首屏 JS <10KB | 弱网/低端机秒开率大幅提升 |

## Qwik 语法速览

```tsx
// Qwik 用 $ 后缀标记可序列化边界
component$(() => {
  const count = useSignal(0);
  return <button onClick$={() => count.value++}>Count: {count.value}</button>;
});
```

React 开发者零学习成本上手，核心区别：useState -> useSignal，事件加 $ 后缀。

## 关键经验

1. **Resumability 从根源消除水合**：HTML 是应用状态的完整序列化，不再是静态骨架
2. **极致代码拆分**：按交互行为级拆分，首屏仅加载必要元数据
3. **工程化适配是关键**：自定义 Adapter、部署流程重构、容灾机制缺一不可
4. **性能优化是系统工程**：CSS 内联 + 资源编排 + 接口聚合多管齐下