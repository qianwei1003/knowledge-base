# ECC 前端开发模式

本文档综合 ECC 前端技能库中的核心开发模式，按模块分层，供日常开发参考。

---

## 1. React/Next.js 组件模式（frontend-patterns）

**何时激活：** 构建 React/Next.js 项目时，优先使用此模式。

### 组件设计

- **组合优于继承**：Card/CardHeader/CardBody，语义清晰，灵活组合。
- **复合组件（Compound Components）**：Tabs/TabList/Tab + Context 共享状态，子组件通过 Context 获取父组件状态，无需层层 prop 传递。
- **渲染属性模式（Render Props）**：DataLoader 组件将数据获取逻辑封装，接受 children 作为渲染函数。

```tsx
// 复合组件示例
const Tabs = ({ children }) => (
  <TabContext.Provider value={useTabs()}>{children}</TabContext.Provider>
);

// 渲染属性模式
<DataLoader url="/api/data">
  {({ data, loading }) => loading ? <Spinner /> : <List data={data} />}
</DataLoader>
```

### 自定义 Hooks

- `useToggle`：布尔值切换，返回 `[value, toggle]`。
- `useQuery`：封装异步数据获取，支持 loading/error/data 三态。
- `useDebounce`：防抖，常用于搜索输入。

```tsx
const [keyword, setKeyword] = useState('');
const debouncedKeyword = useDebounce(keyword, 300);
```

### 状态管理

**Context + Reducer**：适合全局状态（如 MarketContext），通过 `useContext` + `useReducer` 管理复杂状态。

```tsx
const MarketContext = createContext(null);
const [state, dispatch] = useReducer(marketReducer, initialState);
```

### 性能优化

| 工具 | 场景 |
|------|------|
| `useMemo` | 昂贵计算、依赖稳定的派生数据 |
| `useCallback` | 回调函数作为 prop，避免子组件重渲染 |
| `React.memo` | 纯组件 props 未变化时跳过重渲染 |
| `lazy + Suspense` | 代码分割，按需加载 |
| `@tanstack/react-virtual` | 长列表虚拟化，DOM 节点数量固定 |

### 表单处理

受控表单 + validate 函数，验证 name/description/endDate 等字段，返回错误信息对象。

```tsx
const validate = (values) => {
  const errors = {};
  if (!values.name) errors.name = '必填';
  if (!values.endDate) errors.endDate = '必填';
  return errors;
};
```

### 错误边界

Class Component 实现 `getDerivedStateFromError`，捕获子树渲染错误，显示兜底 UI。

### 动画与无障碍

- **Framer Motion**：`AnimatePresence` 退出动画、`motion.div` 过渡、`Modal` 组件。
- **无障碍**：Dropdown 支持 ArrowUp/Down/Enter/Escape 键盘导航；Modal 使用焦点陷阱（Focus Trap）。

---

## 2. HTML 演示文稿（frontend-slides）

**何时激活：** 需要用纯 HTML 实现幻灯片演示时使用。

### 不可妥协原则

- **零依赖**：单 HTML 文件，不依赖任何外部 CDN/CSS 框架。
- **必须适配视口**：每张幻灯片 `height: 100vh`（或 `100dvh`）+ `overflow: hidden`。
- **独特设计**：避免通用紫色渐变，追求独特视觉风格。

### 视口安全

- 每张幻灯片独立 `height: 100vh`，确保内容不溢出。
- 字体用 `clamp()` 自动缩放：`font-size: clamp(1.5rem, 3vw, 2.5rem)`。

### 内容密度限制

| 类型 | 限制 |
|------|------|
| 标题 | 1标题 + 1副标题 |
| 内容 | 4-6 要点或 2 短段 |
| 功能网格 | 最多 6 张卡片 |
| 代码 | 8-10 行 |
| 引用 | 1 条 + 出处 |

### 必需 JS 功能

- 键盘导航（←/→/Space）
- 触摸滑动（touchstart/touchend）
- 滚轮翻页（wheel）
- 进度指示器
- Intersection Observer 动画

### PPT 转换

使用 `python3 + python-pptx` 提取文本、图片、备注，自动生成 HTML 结构。

---

## 3. iOS 26 液态玻璃设计（liquid-glass-design）

**何时激活：** 开发 iOS 26 及以上版本应用，引入液态玻璃效果时使用。

### SwiftUI

```swift
.glassEffect()
.glassEffect(.regular.tint(.orange).interactive(), in: .rect())
```

- **GlassEffectContainer**：包装组件，通过 `spacing` 控制相邻玻璃元素的合并距离。
- **glassEffectUnion**：多个玻璃元素在视觉上合并为一个整体。
- **变形过渡**：配合 `@Namespace` 和 `glassEffectID` 实现元素变形动画。

### UIKit

- `UIGlassEffect`：基础玻璃模糊效果。
- `UIGlassContainerEffect`：容器级别的玻璃效果，管理子元素合并逻辑。

### WidgetKit

```swift
widgetRenderingMode
widgetAccentable
containerBackground
```

### 核心设计原则

- GlassEffectContainer 包装性能优化组件。
- `spacing` 控制元素合并距离，间距越小越容易合并。
- 使用 `withAnimation` 实现流畅的变形过渡。

---

## 4. Next.js 16+ 与 Turbopack（nextjs-turbopack）

**何时激活：** 使用 Next.js 16 及以上版本时。

### 核心变化

- **Turbopack 默认启用**：开发环境替代 Webpack，构建速度显著提升。
- **文件系统缓存**：增量编译，修改文件只重新构建受影响部分。
- **`next build` 生产构建**：保持稳定，生产环境行为与开发一致。

### 实验性功能（Next.js 16.1+）

- **捆绑包分析器**（Bundle Analyzer）：可视化产物大小，定位体积膨胀来源。

---

## 5. Nuxt 4 SSR 模式（nuxt4-patterns）

**何时激活：** 构建 Nuxt 4 SSR 应用时。

### 水合安全（Hydration Safety）

- **禁止在 SSR 中使用** `Date.now()` / `Math.random()`，会导致水合不匹配。
- 使用 `onMounted` 或 `<ClientOnly>` 隔离客户端专属逻辑。

```vue
<ClientOnly>
  <DatePicker />
</ClientOnly>
```

### 数据获取

| 方法 | 特点 |
|------|------|
| `useFetch` | SSR 安全，自动 payload 转发，最常用 |
| `useAsyncData` | 自定义键，支持更灵活的缓存控制 |
| `pick` | 裁剪 payload，减少客户端数据量 |
| `lazy: true` | 非阻塞，数据异步加载 |

### 路由规则

```ts
routeRules: {
  '/': { prerender: true },
  '/blog/**': { swr: 3600 },
  '/dashboard': { ssr: false }
}
```

- `prerender`：静态预渲染
- `swr`：ISR 增量静态再生
- `ssr: false`：纯客户端渲染

### 懒加载

- **Lazy 前缀**：`<LazyHeavyComponent />`，自动代码分割。
- **`hydrate-on-visible`**：元素进入视口时才水合。
- **`defineLazyHydrationComponent`**：细粒度控制懒水合。

### NuxtLink 预取

`prefetch` 属性在链接进入视口时预取页面数据。

---

## 6. Node.js/Express/Next.js API 模式（backend-patterns）

**何时激活：** 构建后端 API 时，作为架构设计参考。

### 架构分层

```
API Route → Middleware → Service → Repository → Database
```

### Repository 模式

接口与实现分离，便于测试和替换数据源。

```ts
interface UserRepository {
  findById(id: string): Promise<User | null>;
  create(data: CreateUserDto): Promise<User>;
}

// Supabase 实现
class SupabaseUserRepository implements UserRepository {
  async findById(id: string) {
    return supabase.from('users').select('*').eq('id', id).single();
  }
}
```

### 中间件：withAuth

高阶函数模式，封装认证逻辑。

```ts
const withAuth = (handler) => async (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  req.user = jwt.verify(token, process.env.JWT_SECRET);
  return handler(req, res);
};
```

### 数据库

- **N+1 预防**：使用批量 fetch（如 `IN` 查询）替代循环查询。
- **事务模式**：多个写操作原子提交。

### 缓存：Redis 旁路缓存（Cache-Aside）

```ts
async function getUser(id: string) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  const user = await db.user.findUnique({ where: { id } });
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  return user;
}
```

### 错误处理

```ts
class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

app.use((err, req, res, next) => {
  const status = err instanceof ApiError ? err.statusCode : 500;
  res.status(status).json({ error: err.message });
});
```

### 指数退避

```ts
async function fetchWithRetry(url: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (e) {
      if (i === retries - 1) throw e;
      await new Promise(r => setTimeout(r, 2 ** i * 1000));
    }
  }
}
```

### 认证与权限

- **JWT verify**：验证 Token 有效性，提取用户信息。
- **RBAC**：基于角色的权限矩阵，`admin > editor > viewer`。

### 速率限制

滑动窗口 RateLimiter，防止滥用 API。

### 后台队列

JobQueue 简单实现，异步处理耗时任务（如发送邮件、图像处理）。

### 结构化日志

```ts
class Logger {
  log(level: string, message: string, context?: object) {
    console.log(JSON.stringify({ timestamp: new Date().toISOString(), level, message, ...context }));
  }
}
```

---

*文档版本：2026-04-20 | 来源：ECC 前端技能库综合整理*
