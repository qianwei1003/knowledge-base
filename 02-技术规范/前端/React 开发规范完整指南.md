---
来源: 博客园
原始链接: https://www.cnblogs.com/yangstar90/p/6374374.html
---

# React 开发规范完整指南

## 核心要点
- 文件/组件名统一使用 PascalCase，单文件单组件
- JSX 属性多行排列时每行一个属性，闭合标签单独成行
- Props 和 State 不可直接修改，始终创建新引用
- 组件内部按生命周期顺序组织方法，render 放最后
- CSS 使用连字符命名，嵌套不超过 3 层

---

## 命名规范

**文件与扩展名**
| 项目 | 规范 | 示例 |
|------|------|------|
| 扩展名 | `.js` 或 `.jsx` | `MyComponent.jsx` |
| 文件名 | PascalCase | `MyComponent.js` |
| 目录根组件 | `index.js` | `components/Button/index.js` |

**引用命名**
- React 组件：大驼峰（PascalCase）
- HTML 标签：小驼峰（camelCase）

```jsx
// good
const Footer = require('./Footer')

// bad
const Footer = require('./Footer/Footer.js')
```

**属性命名**
- Props：小驼峰；HTML 属性用 `className`（非 `class`）、`htmlFor`（非 `for`）
- 自定义属性加 `data-` 前缀；`aria-*` 正常可用

---

## 属性设置

```jsx
// good - 行内设置
<Component foo={x} bar={y} />

// good - 展开运算符
var component = <Component {...props} />

// bad - 外部修改
component.props.foo = x

// 属性多行排列
<input
  type="text"
  value={this.state.newDinosaurName}
  onChange={this.handleChange}
/>
```

---

## 组件声明与组织

```jsx
// good
const ReservationCard = React.createClass({ ... });
export default ReservationCard;

// bad
export default React.createClass({ displayName: 'ReservationCard', ... });
```

**内部顺序**
```
displayName -> mixins -> statics -> propTypes
-> getDefaultProps() -> getInitialState()
-> componentWillMount -> componentDidMount
-> 事件处理器（handleClick）-> getter
-> render() 始终放最后
```

---

## JSX 规范

**多行用括号，单行省略**
```jsx
// good
return (
  <div>
    <ComponentOne />
    <ComponentTwo />
  </div>
);

// bad
return (<div><ComponentOne /><ComponentTwo /></div>);
```

**自闭合标签：`/` 前留一个空格**
```jsx
<Logo />   // good
<Logo></Logo>  // bad
```

**条件渲染**
```jsx
// 简短用三元
{this.state.show && 'Shown'}
{this.state.on ? 'On' : 'Off'}

// 复杂结构在 render 内定义变量
var dinosaurHtml = '';
if (this.state.showDinosaurs) {
  dinosaurHtml = <section><DinosaurTable /><Pager /></section>;
}
return <div>{dinosaurHtml}</div>;
```

**注释用 `{}` 包裹**
```jsx
<Nav>
  {/* child comment */}
  <Person name={window.isLoggedIn ? window.name : ''} />
</Nav>
```

---

## CSS 规范

```css
/* good */
#nav {}
.box-video {}
#username input {}

/* bad */
.red {}
.box_green {}
ul#example {}
.page .header .login #username input {}  /* 嵌套过深 */
```

- 选择器左 `{` 前一个空格，右 `}` 单独成行
- `:` 后一个空格，分号结尾
- 颜色值小写并简写（`#fff` 代替 `#ffffff`）
- 零值不带单位（`margin: 0` 代替 `margin: 0px`）

---

## 条件操作符

```javascript
// bad
if (val != 0) return foo(); else return bar();

// good
return val ? foo() : bar();
```
