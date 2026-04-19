---
来源: CSDN博客
原始链接: https://blog.csdn.net/TrisighT0/article/details/160079961
---

# React 19 新特性深度解析与迁移指南

## 核心要点

- **use() Hook**：可在条件语句中调用，读取 Promise/Context，配合 Suspense 使用
- **Server Components**：生产可用，服务端直连数据库，客户端组件用 `'use client'` 标记
- **Form Actions**：原生集成 Server Actions，`useActionState` 管理表单提交状态
- **useOptimistic / useFormStatus**：乐观更新和表单状态 Hook，提升交互体验
- **React Compiler**：自动 memoization，无需手动 `useMemo`/`useCallback`

## use() Hook

打破传统 Hooks 不能条件调用的限制，读取 Promise 和 Context：

```jsx
import { use, Suspense } from 'react';

// 读取 Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}

// 可条件调用（Hooks 不允许）
function Comment({ commentPromise, isExpanded }) {
  if (isExpanded) {
    return <ExpandedComment comment={use(commentPromise)} />;
  }
  return <SummaryComment comment={use(commentPromise)} />;
}
```

## Server Components

```jsx
// 服务端组件 — 直接访问数据库
async function UserList() {
  const users = await db.query('SELECT * FROM users LIMIT 10');
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}

// 客户端组件
'use client';
function LikeButton({ initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  return <button onClick={() => setLikes(l + 1)}>👍 {likes}</button>;
}
```

## Form Actions + useActionState

```jsx
// actions/userActions.ts
'use server';
export async function createUser(formData) {
  const name = formData.get('name');
  await db.users.create({ name });
  return { success: true };
}

// 表单组件
function UserForm() {
  const [state, formAction, isPending] = useActionState(createUser, null);
  return (
    <form action={formAction}>
      <input name="name" placeholder="用户名" />
      <button type="submit" disabled={isPending}>
        {isPending ? '提交中...' : '注册'}
      </button>
    </form>
  );
}
```

## useOptimistic — 乐观更新

```jsx
function ChatRoom({ messages, sendMessage }) {
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (state, newMsg) => [...state, { id: Date.now(), text: newMsg, pending: true }]
  );
  async function handleSend(formData) {
    addOptimisticMessage(formData.get('message'));
    await sendMessage(text);
  }
}
```

## Asset Loading API

```jsx
import { preload, preconnect, prefetchDNS } from 'react-dom';
preconnect('https://api.example.com');
preload('/fonts/custom.woff2', { as: 'font', type: 'font/woff2' });
```

## React Compiler 自动优化

无需手动 memoization，编译器自动处理：

```jsx
// ❌ React 18：需手动优化
const processed = useMemo(() => expensiveCalculation(data), [data]);
const handleClick = useCallback(() => setCount(c => c + 1), []);

// ✅ React 19：直接写，编译器自动缓存
function ExpensiveComponent({ data, onClick }) {
  const processed = expensiveCalculation(data);
  return <div onClick={onClick}>{processed}</div>;
}
```

## 迁移指南

```bash
npm install react@19 react-dom@19
npm install -D @types/react@19 @types/react-dom@19
```

关键变更：

| 变更项 | React 18 | React 19 |
|--------|----------|----------|
| ref 回调 | 必须用 callback ref | 支持返回 null |
| Context | Context.Provider | 兼容，新增 `use()` |
| Strict Mode | 双重渲染 | 行为优化 |

SSR 应用使用 `hydrateRoot`：

```jsx
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document.getElementById('root'), <App />, {
  onRecoverableError: (error) => console.error('Recoverable:', error)
});
```
