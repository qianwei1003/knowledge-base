---
来源: Airbnb JavaScript 规范
原始链接: https://github.com/airbnb/javascript/blob/master/react/README.md
---

# Airbnb React/JSX 代码规范

## 核心要点

- 每个文件一个 React 组件，始终使用 JSX 语法
- Class 组件用于有 state/refs 的场景，纯展示组件用函数组件
- 文件名 PascalCase，组件引用 PascalCase，实例 camelCase
- Props 用 camelCase 命名，布尔值省略 `={true}`，避免复用 DOM 属性名

## 基本规则

- 每个文件只包含一个 React 组件，始终使用 JSX 语法
- 禁止使用 Mixins → 用 HOC、Hooks 或工具模块替代

## Class vs Function Components

✅ 有 state/refs → `class extends React.Component`

```jsx
class Listing extends React.Component {
  render() {
    return <div>{this.state.hello}</div>;
  }
}
```

✅ 无 state/refs → 普通函数（非箭头函数）

```jsx
function Listing({ hello }) {
  return <div>{hello}</div>;
}
```

❌ 不依赖箭头函数的 displayName 推断

## 命名规范

| 场景 | 规范 | 示例 |
|------|------|------|
| 文件扩展名 | `.jsx` | `ReservationCard.jsx` |
| 文件名 | PascalCase | `ReservationCard.jsx` |
| 组件引用 | PascalCase | `import ReservationCard` |
| 组件实例 | camelCase | `const reservationItem = <ReservationCard />` |
| HOC displayName | 复合命名 | `withFoo(Bar)` |

### Props 命名

- ✅ camelCase，prop 值为 React 组件时用 PascalCase
- ✅ 布尔值省略 `={true}`：`<Foo visible />`
- ❌ 避免复用 DOM 属性名（如 `style`、`className`）

## 对齐与格式

✅ 多行属性每个属性独占一行：

```jsx
<Foo
  superLongParam="bar"
  anotherSuperLongParam="baz"
/>
```

- JSX 属性使用**双引号**，其他 JS 使用**单引号**
- 自闭合标签 `/>` 前加一个空格：`<Foo />`
- JSX 花括号内不加空格：`{foo}`

## Props 最佳实践

- ✅ 为所有非 required props 定义 `defaultProps`
- ✅ 使用稳定 ID 作为 key，避免数组索引
- ✅ `<img>` 必须包含 `alt`，alt 中避免 "image"、"photo"、"picture"
- ❌ 谨慎使用 spread props `{...props}`，过滤不必要的属性

## Refs

✅ 始终使用 ref 回调：

```jsx
<Foo ref={(ref) => { this.myRef = ref; }} />
```

❌ 不使用字符串 refs

## 标签与顺序

- 无子元素标签自闭合：`<Foo />`
- 多行 JSX 用括号包裹，多行属性在新行关闭
- Class 成员顺序：`extends` → `statics` → `propTypes` → `defaultProps` → `contextType` → `state`
- ❌ 避免使用 `isMounted`

## 方法

- ✅ 使用箭头函数捕获局部变量
- ⚠️ 传给 PureComponent 时箭头函数会每次创建新引用，注意性能
