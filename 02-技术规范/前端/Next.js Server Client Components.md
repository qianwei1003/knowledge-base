---
来源: 官方文档
原始链接: https://nextjs.org/docs/app/getting-started/server-and-client-components
---

# Next.js App Router: Server 与 Client Components

## 核心要点

- **Server Components（默认）**：适合数据获取、使用密钥、减少JS bundle、改善FCP
- **Client Components（`"use client"`）**：需要状态、事件处理、生命周期、浏览器API时使用
- **`"use client"` 定义边界**：标记后，所有导入和子组件都属于客户端bundle
- **children模式**：在Client Component中嵌入Server Component渲染内容，避免全部客户端化
- **Context需包裹**：React Context不支持Server Components，需创建Client Component包裹

---

## 什么时候用 Server vs Client Components？

### ✅ 使用 Client Components

- **State** 和事件处理（`onClick`, `onChange`）
- **生命周期逻辑**（`useEffect`）
- **浏览器专属 API**（`localStorage`, `window`, `Navigator.geolocation`）
- **自定义 Hooks**

### ✅ 使用 Server Components（默认）

- 从数据库或 API **获取数据**
- 使用 **API 密钥、令牌**等密钥
- **减少**发送到浏览器的 JavaScript 量
- 改善 **First Contentful Paint (FCP)**
- **流式传输**内容到客户端

## 核心模式

### 1. 声明 Client Component
```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

`"use client"` 声明了 **Server 和 Client 模块图之间的边界**。

### 2. 减少 JS bundle 大小
```tsx
// app/layout.tsx - Server Component（默认）
import Search from './search'    // Client Component
import Logo from './logo'        // Server Component

export default function Layout({ children }) {
  return (
    <>
      <nav>
        <Logo />
        <Search />   {/* 只有 Search 部分需要客户端 JS */}
      </nav>
      <main>{children}</main>
    </>
  )
}
```

> 💡 只对交互组件加 `'use client'`，而非大面积标记。

### 3. Server → Client 数据传递（Props）
```tsx
// Server Component
export default async function Page({ params }) {
  const post = await getPost(params.id)
  return <LikeButton likes={post.likes} />
}

// Client Component
'use client'
export default function LikeButton({ likes }) {
  const [count, setCount] = useState(likes)
  return <button onClick={() => setCount(count + 1)}>{count} likes</button>
}
```

### 4. 交错使用 Server/Client Components（children 模式）
```tsx
// Client Component - 用 children 创建"插槽"
'use client'
export default function Modal({ children }) {
  const [isOpen, setIsOpen] = useState(false)
  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open</button>
      {isOpen && <div>{children}</div>}
    </div>
  )
}

// Server Component - 传入 Server Component 作为 children
export default function Page() {
  return (
    <Modal>
      <Cart />  {/* Cart 是 Server Component，在服务端预先渲染 */}
    </Modal>
  )
}
```

### 5. Context Providers
React Context 不支持 Server Components。需要创建 Client Component 包裹：

```tsx
'use client'
import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({ children }) {
  return (
    <ThemeContext.Provider value={{ theme: 'dark' }}>
      {children}
    </ThemeContext.Provider>
  )
}
```

## 关键概念

| 概念 | 说明 |
|------|------|
| RSC Payload | Server Component 渲染结果的紧凑二进制表示 |
| Hydration | React 将事件处理器附加到 DOM，使静态 HTML 可交互 |
| `"use client"` | Server/Client 模块图的边界声明 |
| children 模式 | 在 Client Component 中嵌入 Server Component 渲染内容 |
