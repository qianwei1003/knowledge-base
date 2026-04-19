---
来源: CSDN博客 (FrontAI)
原始链接: https://blog.csdn.net/FrontAI/article/details/159997493
---

# Next.js 数据获取与缓存策略完全指南

## 核心要点

- **Server Components 默认**：App Router 组件默认为服务端组件，可直接 `async/await` 获取数据，凭证不暴露给浏览器
- **Fetch 缓存三模式**：`force-cache`（静态）、`no-store`（实时）、`revalidate: N`（定时刷新）
- **四层缓存体系**：Request Memoization → Data Cache → Full Route Cache → Router Cache
- **缓存精确失效**：`revalidateTag` 按标签批量失效，`revalidatePath` 按路径失效
- **Server Actions 优先**：数据写入优先用 Server Actions，Route Handlers 仅用于 webhook/第三方调用/SSE

## 一、服务端数据获取

```tsx
export default async function BlogPage() {
  const posts = await prisma.post.findMany({
    where: { published: true },
    orderBy: { createdAt: 'desc' },
  })
  return posts.map(post => <article key={post.id}><h2>{post.title}</h2></article>)
}
```

优势：数据库凭证仅服务端存在；服务器与数据库同机房，延迟毫秒级。

## 二、Fetch 缓存策略

```tsx
// 永久缓存（构建时获取一次）
const res = await fetch('https://api.example.com/config', { cache: 'force-cache' })

// 禁用缓存（每次实时获取）
const res = await fetch('https://api.example.com/live-data', { cache: 'no-store' })

// 定时重新验证
const res = await fetch('https://api.example.com/posts', { next: { revalidate: 3600 } })
```

| 数据类型 | 推荐策略 | 场景 |
|---------|---------|------|
| 静态配置 | `force-cache` | 导航配置、国家列表 |
| 定期更新 | `revalidate: N` | 博客（小时级）、天气（分钟级） |
| 高实时性 | `no-store` | 通知、股票、购物车 |

## 三、四层缓存架构

| 层 | 位置 | 作用 |
|----|------|------|
| Request Memoization | 请求内存 | 同一请求内相同 fetch 自动去重 |
| Data Cache | 服务端持久化 | 多用户共享 fetch 结果 |
| Full Route Cache | 服务端持久化 | 静态页面的完整 HTML/RSC payload |
| Router Cache | 浏览器端 | 前进/后退时复用已访问页面 |

## 四、缓存标签：精确失效

```tsx
// 写入时标记
async function getBlogPosts() {
  return fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] },
  }).then(r => r.json())
}

// 更新时失效
'use server'
import { revalidateTag } from 'next/cache'
export async function publishPost(postId: string) {
  await prisma.post.update({ where: { id: postId }, data: { published: true } })
  revalidateTag('posts')
}
```

## 五、流式渲染（Suspense）

```tsx
export default async function ProductPage({ params }) {
  return (
    <div>
      <ProductInfo id={params.id} />
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews id={params.id} />
      </Suspense>
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations id={params.id} />
      </Suspense>
    </div>
  )
}
```

> 适用场景：相对独立且加载较慢的内容区域。不要过度使用。

## 六、数据新鲜度决策框架

```
数据更新频率？
├── 几乎不变 → force-cache / SSG
├── 规律性变化
│   ├── 新鲜度要求不高 → revalidate: 3600
│   └── 新鲜度要求较高 → revalidate: 60 + revalidateTag
└── 每次不同 / 必须实时
    ├── 与用户身份无关 → no-store
    └── 与用户身份相关 → no-store（严禁缓存用户私有数据！）
```

## 七、并行请求

```tsx
// ✅ 并行（推荐）
const [user, posts, followers] = await Promise.all([
  getUser(id), getUserPosts(id), getFollowers(id),
])
// 耗时 = max(500, 300, 400) = 500ms

// ❌ 串行（不推荐）
// 耗时 = 500 + 300 + 400 = 1200ms
```

## 八、Server Actions vs Route Handlers

| 场景 | 推荐方案 |
|------|---------|
| 表单提交 / 数据增删改 | ✅ Server Actions |
| Webhook 接收 / 第三方调用 | ✅ Route Handlers |
| SSE / AI 流式输出 | ✅ Route Handlers |
| 服务端组件 fetch 自己的 Route Handler | ❌ 反模式，直接调用数据库 |

```tsx
// Server Action 示例
'use server'
import { revalidatePath } from 'next/cache'
export async function createComment(postId: string, content: string) {
  await prisma.comment.create({ data: { postId, content, authorId: getCurrentUserId() } })
  revalidatePath(`/blog/${postId}`)
}
```
