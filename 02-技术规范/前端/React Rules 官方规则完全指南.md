---
来源: React 官方文档
原始链接: https://react.dev/reference/rules
---

# React Rules 官方规则完全指南

> 版本：React 19+
> React 的规则不是建议——违反它们会导致 bug 和不可预期的行为。

## 核心要点
- 组件和 Hook 是纯函数：相同输入始终产生相同输出
- 副作用（fetch、DOM 操作、订阅）必须在渲染之外运行
- Props 和 State 不可变，永远不要直接修改它们
- Hooks 只在组件顶层调用，禁止在条件/循环/嵌套函数中调用
- 永远不要将 Hook 作为普通值传递，或直接调用组件函数

---

## 一、纯函数：组件和 Hooks 必须保持纯净

### 1.1 什么是纯函数？

- **幂等性**：相同输入 -> 相同输出
- **渲染时无副作用**：副作用在渲染外运行
- **不修改非局部值**：不修改渲染中未创建的引用

### 1.2 非纯函数（禁止在渲染中使用）

```
new Date() / Date.now()
Math.random()
fetch()
```

### 1.3 代码示例

```jsx
// Bad - 每次渲染结果不同，非幂等
function Clock() {
  const time = new Date();
  return <span>{time.toLocaleString()}</span>
}

// Good - 用 state + Effect 保持幂等
function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

---

## 二、副作用必须在渲染之外运行

| 场景 | 解决方案 |
|------|----------|
| 用户交互 | 事件处理器 |
| 外部系统同步 | useEffect |
| 数据订阅 | useEffect |
| DOM 操作 | useEffect |

```jsx
// Bad - 渲染中发起网络请求
function UserProfile({ userId }) {
  const user = fetchUser(userId);
  return <div>{user.name}</div>;
}

// Good - useEffect 处理副作用
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser(userId).then(data => setUser(data));
  }, [userId]);
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

---

## 三、Props 和 State 是不可变的

```jsx
// Bad - 直接修改 state
const addTodo = (text) => {
  todos.push({ id: Date.now(), text });
  setTodos(todos);
};

// Good - 创建新数组
const addTodo = (text) => {
  setTodos([...todos, { id: Date.now(), text }]);
};

// Bad - 修改 props
function Profile({ user }) {
  user.name = 'John';
}

// Good - 创建新对象
function Profile({ user }) {
  const updatedUser = { ...user, name: 'John' };
}
```

---

## 四、React 负责调用组件和 Hooks

### 不要直接调用组件函数

```jsx
// Bad
function App() {
  const btn = Button();
}

// Good - JSX 中使用
function App() {
  return <Button />;
}
```

### 不要将 Hooks 作为普通值传递

```jsx
// Bad
function useMyHook() {
  const [, setState] = useState(0);
  return setState;
}

// Good - 返回包含 Hook 逻辑的函数
function useMyHook() {
  const [n, setN] = useState(0);
  const increment = () => setN(s => s + 1);
  return { state: n, increment };
}
```

---

## 五、Hooks 规则（Rules of Hooks）

### 5.1 只在顶层调用 Hooks

禁止在循环、条件、嵌套函数、事件处理器、类组件、try/catch 中调用。

```jsx
// Bad - 条件中调用
if (cond) { const [x] = useState(); }

// Bad - 循环中调用
for (let i = 0; i < 10; i++) { useContext(Theme); }

// Bad - 事件处理器中调用
function handleClick() { useState(0); }

// Bad - try/catch 中调用
try { useState(); } catch { useState(1); }

// Good - 顶层调用
function Good({ cond }) {
  const [x] = useState();
  if (cond) { /* 可以使用 x */ }
}
```

### 5.2 只在 React 函数中调用 Hooks

**good**: React 函数组件、自定义 Hooks
**bad**: 普通 JS 函数、类组件、事件处理器

---

## 六、推荐工具

```bash
npm install eslint-plugin-react-hooks --save-dev

<React.StrictMode>
  <App />
</React.StrictMode>
```
