---
来源: 美团技术团队官方博客
领域: 前端
关键词: Qwik.js, SSR, 性能优化, Resumability, 首屏优化, 大众点评
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://tech.meituan.com/2026/03/13/qwik-practice-in-dianping.html
---

# 重塑站外体验：大众点评 M 站基于 Qwik.js 的重构实践

## 一、背景与挑战：流量转化与用户体验的困境

### M 站定位
M 站（Mobile 站）是面向公域的流量引流入口，定位为"信息展示 + 点击唤端"的极简触点。核心功能：
- 商户详情页
- 内容详情页
- 首页

**核心目标**：让最差的设备加最差的网络也能流畅打开页面，最大化挖掘公域流量的转化价值。

### 传统架构的性能瓶颈
旧技术栈基于小程序 DSL 编译生成 Vue 产物，存在三大问题：
1. **首屏渲染效率低下**：无缓存首次访问页面白屏时间长
2. **运行时性能瓶颈**：跨端编译产物冗余，首屏资源体积大
3. **开发维护成本高**：框架停止维护，与 AI Coding 体系脱节

## 二、技术选型：为何选择 Qwik？

### 站外场景的四个约束
1. 无预优化能力，首屏资源必须"从零加载"
2. POI 数据需要保持实时性
3. 大众点评 POI 数量超千万级，预生成成本极高
4. 搜索引擎带来巨量流量，有强 SEO 诉求

**结论**：服务端渲染是唯一解法。

### Qwik 的核心优势

#### 1. Resumability（可恢复性）
- **传统 SSR**：服务端返回静态 HTML → 客户端下载全量 JS → 水合激活
- **Qwik**：服务端序列化状态和事件绑定到 HTML → 客户端无需水合，按需加载

```
传统 SSR 水合过程：
下载 JS → 解析执行 → 生成虚拟 DOM → 对比绑定 → 激活交互

Qwik Resumability：
HTML 已包含状态 → Qwikloader（<1KB）监听事件 → 用户交互时按需加载对应代码块
```

#### 2. 极致细粒度代码拆分
- 所有内容可延迟加载：组件、方法、监听器、样式
- 按事件/交互拆分，而非按组件拆分
- 每个 `$` 标记函数自动提取为独立 chunk

#### 3. 编译期优化
- 预编译响应式依赖
- 规避虚拟 DOM，直接操作真实 DOM
- 极致 Tree Shaking

### Qwik vs React 语法对比

```tsx
// React
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// Qwik
component$(() => {
  const count = useSignal(0);
  return (
    <button onClick$={() => count.value++}>
      Count: {count.value}
    </button>
  );
});
```

**学习成本**：React 开发者几乎零学习成本上手。

## 三、架构设计：面向首屏最优

### 混合加载策略

#### SSR 首屏直出层（L0 级数据）
- POI 基础信息（店名、星级、地址、相册）
- 服务端接口聚合，并行拉取
- 直接渲染进 HTML

#### CSR 渐进加载层（L1 级数据）
- 商家券、团购等依赖到餐链路数据
- 评价列表、问答等首屏外数据
- useVisibleTask$ 钩子渐进加载

### 部署架构优化
- **Serverless 平台**：复用请求拦截、日志监控、容灾、负载均衡
- **自定义 Adapter**：适配 Express、Fastify、Node Server
- **路由层裁剪**：页面粒度部署，减少路由匹配开销
- **连接池复用**：undici 替代 axios，跨服务请求耗时降低 20%+

### 容错与降级机制
1. **SSR 接口超时降级**：熔断后客户端重试
2. **Serverless 渲染失败降级**：回退到 SSG 静态产物
3. **兼容性兜底**：低端浏览器重定向到旧版 M 站

## 四、性能优化实践

### CSS 内联策略
- 公共样式内联到 `<head>` 的 `<style>` 标签
- 消除 CSS 网络往返时延
- FCP 时间 TP90 降低 100+ms

### 资源编排优化
- LCP 图片 Preload + fetchPriority="high"
- AVIF 格式资源体积降低 80%，LCP 平均下降 300+ms
- DNS 预解析和预连接
- 首屏 JS 体积从 2MB 降至 <10KB

### 接口聚合优化
- BFF 聚合层统一封装业务模块
- 登录态校验、Token 续期、实验分流下沉到主接口
- 后端聚合主接口响应 TP95 降低 31.6%

## 五、成果总结

### 性能提升
- 首屏秒开率大幅提升
- 首屏 JS 加载体积降幅超 99%
- 弱网、低端机型表现突出

### 业务价值
- 性能优化带动业务指标稳步增长
- 高效释放公域流量价值

### 技术沉淀
- Qwik 落地经验可复用
- 站外渲染架构可沉淀

## 六、关键要点

1. **Resumability 从根源消除水合**：HTML 不再只是静态骨架，而是应用状态的完整序列化
2. **极致的代码拆分**：按交互行为级拆分，首屏仅加载必要元数据
3. **工程化适配是关键**：自定义 Adapter、部署流程重构、容灾机制设计
4. **性能优化是系统工程**：CSS 内联、资源编排、接口聚合多管齐下

## 参考链接

- [Qwik 官方文档](https://qwik.dev/)
- [React Cheat Sheet](https://qwik.dev/docs/guides/react-cheat-sheet/)
- 原文：https://tech.meituan.com/2026/03/13/qwik-practice-in-dianping.html
