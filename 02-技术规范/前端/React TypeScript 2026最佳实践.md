---
来源: CSDN博客
领域: 前端
关键词: React, TypeScript, React.FC, 泛型组件, 可辨识联合, strict模式, JSX Transform, 自定义Hook
质量: ⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://blog.csdn.net/Ed7zgeE9X/article/details/158159826
---

# React + TypeScript 2026 最佳实践：告别旧模式

## 一、TypeScript 为什么配和 React 搭档？

写 JavaScript 的 React 项目，就像在没有地图的城市里开车——每次拐弯都要猜：这个 props 到底传了什么？这个函数的返回值是啥类型？

TypeScript 给你装上了导航。2025年，JS 生态里的 TypeScript 采用率已达 78%（State of JS 数据），大厂项目几乎清一色 TypeScript。

## 二、新 JSX Transform：从此不用每个文件写 import React

### 以前为什么要 import React？

React 17 之前，写 JSX 其实是在写 `React.createElement('div', null, 'hello')`，所以必须引入 React。

### 现在不需要了

React 17+ 引入了新的 JSX Transform，编译器会自动从 `react/jsx-runtime` 引入需要的函数：

```typescript
// ✅ 2026 写法：不需要 import React
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
}

export const Button = ({
  label,
  onClick,
  variant = 'primary'
}: ButtonProps) => {
  return (
    <button
      onClick={onClick}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
};
```

tsconfig.json 配置：

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

## 三、告别 React.FC：显式声明 props 才是正道

### 为什么要放弃 React.FC？

**问题一：隐式包含 children（React 18 之前）**

```typescript
// ❌ 旧写法：React.FC 隐式包含 children
const Card: React.FC<{ title: string }> = ({ title, children }) => {
  // children 哪来的？React.FC 帮你偷偷加上了
  return <div>{title}{children}</div>
}
```

**问题二：表达能力不够强**

```typescript
// ✅ 2026 推荐写法：完全显式
interface CardProps {
  title: string;
  subtitle?: string;
  children?: React.ReactNode;  // 你自己决定要不要接收 children
  onClose?: () => void;
}

const Card = ({ title, subtitle, children, onClose }: CardProps) => {
  return (
    <div className="card">
      <div className="card-header">
        <h3>{title}</h3>
        {subtitle && <p>{subtitle}</p>}
        {onClose && <button onClick={onClose}>×</button>}
      </div>
      {children && <div className="card-body">{children}</div>}
    </div>
  );
};
```

## 四、泛型组件：一次编写，无数复用

场景：需要通用列表组件，既展示商品、又展示订单、又展示用户。

```typescript
// 定义泛型列表组件
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyText?: string;
  loading?: boolean;
}

const List = <T,>({
  items,
  renderItem,
  keyExtractor,
  emptyText = '暂无数据',
  loading = false
}: ListProps<T>) => {
  if (loading) return <div className="loading">加载中...</div>;
  if (items.length === 0) return <div className="empty">{emptyText}</div>;

  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
};
```

使用时 TypeScript 完全自动推断类型：

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

// TypeScript 知道 product 是 Product 类型，直接有完整提示
<List
  items={products}
  renderItem={(product) => (
    <span>{product.name} - ¥{product.price}</span>
  )}
  keyExtractor={(product) => product.id}
  emptyText="暂无商品"
/>
```

## 五、可辨识联合类型：告别 undefined 地狱

### 旧写法的问题

```typescript
// ❌ 容易出问题的写法
interface State {
  loading: boolean;
  data?: User[];
  error?: string;
}
// loading: false + data: undefined + error: undefined ——初始状态还是请求失败？
```

### 可辨识联合类型：让非法状态不可能存在

```typescript
// ✅ 可辨识联合类型
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string; retryable: boolean };

const UserList = () => {
  const [state, setState] = useState<RequestState<User[]>>({
    status: 'idle'
  });

  switch (state.status) {
    case 'idle':
      return <button onClick={fetchUsers}>加载用户</button>;

    case 'loading':
      return <Skeleton count={5} />;

    case 'success':
      // TypeScript 100% 确定 state.data 存在且是 User[]
      return <ul>{state.data.map(user => <li key={user.id}>{user.name}</li>)}</ul>;

    case 'error':
      return (
        <div>
          <p>出错了：{state.error}</p>
          {state.retryable && <button onClick={fetchUsers}>重试</button>}
        </div>
      );
  }
};
```

## 六、自定义 Hook 的类型：返回元组别踩坑

```typescript
// 返回元组，用 as const 锁定类型
const useLocalStorage = <T,>(key: string, initialValue: T) => {
  const [value, setValue] = useState<T>(() => {
    try {
      const stored = localStorage.getItem(key);
      return stored ? (JSON.parse(stored) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setStoredValue = (newValue: T | ((prev: T) => T)) => {
    try {
      const valueToStore = newValue instanceof Function
        ? newValue(value)
        : newValue;
      setValue(valueToStore);
      localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('写入 localStorage 失败:', error);
    }
  };

  const removeValue = () => {
    localStorage.removeItem(key);
    setValue(initialValue);
  };

  // as const 关键！让 TypeScript 把返回值推断为元组而不是数组
  return [value, setStoredValue, removeValue] as const;
};

// 使用
const [theme, setTheme, removeTheme] = useLocalStorage('theme', 'light');
// theme: string ✅
// setTheme: (newValue: string | ((prev: string) => string)) => void ✅
// removeTheme: () => void ✅
```

## 七、开启严格模式：让 TypeScript 真正帮你抓 Bug

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true
  }
}
```

- **strict: true** = 开启安全驾驶模式（强制系安全带）
- **noUncheckedIndexedAccess: true** = 给数组访问装上安全气囊，`arr[10]` 类型变为 `number | undefined`
- **exactOptionalPropertyTypes: true** = 区分"属性不存在"和"属性值为 undefined"

## 八、快速落地检查清单

| 检查项 | 旧做法 | 2026 做法 |
|--------|--------|----------|
| 组件声明 | `React.FC<Props>` | `(props: Props) => JSX` |
| children | `React.FC` 隐式包含 | 显式写 `children?: ReactNode` |
| JSX Transform | `import React from 'react'` | 无需 import，配置 `react-jsx` |
| 异步状态 | `loading/data/error` 分散字段 | 可辨识联合类型 |
| 通用组件 | `any` 或写多份 | 泛型组件 `<T,>` |
| Hook 返回 | 直接 return 数组 | `return [...] as const` |
| 严格模式 | 默认配置 | 开启 `strict` + 额外检查 |
| 运行时校验 | 无 | Zod/valibot 配合 TypeScript |

## 总结

TypeScript 本质上是给你的代码系了一条安全带。2026年，React + TypeScript 的最佳实践已经相当成熟。这些模式不是让代码"看起来更高级"，而是让你真正少写 bug、少加班、重构的时候少崩溃。
