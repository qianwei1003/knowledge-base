---
来源: 博客/CSDN
原始链接: https://blog.csdn.net/weixin_43107715/article/details/159618922
---

# React 20 新特性：自动批处理、use() 与编译时优化

## 核心要点

- **全场景自动批处理**：Promise / setTimeout / 原生事件中的多次 setState 自动合并为一次渲染
- **use() Hook**：顶层直接读取 Promise 和 Context，消除 useEffect 嵌套地狱
- **编译时优化**：内置 Compiler，自动跳过无变化子组件渲染，无需 memo/useMemo/useCallback
- **flushSync 可选禁用**：需要立即渲染的特殊场景可用 `flushSync` 强制同步更新

## 自动批处理（Automatic Batching）

React 20 实现全场景自动批处理，不再区分事件上下文：

```jsx
// ❌ React 18：setTimeout 中触发 3 次渲染
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  setText("Updated");
}, 1000);

// ✅ React 20：合并为 1 次渲染
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  setText("Updated");
}, 1000);
```

需要立即渲染时使用 `flushSync`：

```jsx
import { flushSync } from 'react-dom';

setTimeout(() => {
  flushSync(() => {
    setCount(c => c + 1); // 立即触发渲染
  });
  setFlag(f => !f); // 和后续更新合并
}, 1000);
```

## use() Hook

消除 useEffect + state 组合，直接在组件顶层处理异步和上下文：

```jsx
import { use, createContext } from 'react';

// 直接处理异步数据
function UserProfile({ userId }) {
  const user = use(fetchUser(userId));
  return <div><p>{user.name}</p><p>{user.email}</p></div>;
}

// 简化上下文调用
const ThemeContext = createContext('light');
function ThemedButton() {
  const theme = use(ThemeContext);
  return <button>Click me</button>;
}
```

## 编译时优化（Compiler）

无需手动 memoization，编译器自动分析依赖：

```jsx
// ❌ React 18：需要手动 memo + useCallback + useMemo
const ChildComponent = memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});

// ✅ React 20：直接写，Compiler 自动优化
const ChildComponent = ({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
};

function ParentComponent() {
  const [count, setCount] = useState(0);
  const handleClick = () => setCount(c => c + 1);
  return (
    <div>
      <p>Count: {count}</p>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}
```

对比：

| 场景 | React 18 | React 20 |
|------|----------|----------|
| 列表渲染 | 手动 memo 包裹子组件 | Compiler 自动跳过 |
| 回调传递 | useCallback 缓存 | 自动稳定引用 |
| 计算属性 | useMemo 缓存 | 自动缓存纯计算 |

## 升级步骤

```bash
# 1. 升级依赖
npm install react@latest react-dom@latest

# 2. 兼容性扫描
npx react-compat scan ./src

# 3. 逐步迁移：先非核心组件验证，再全面推广
```
