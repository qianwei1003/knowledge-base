---
来源: CSDN 博客
原始链接: https://blog.csdn.net/weiyong_94/article/details/111034532
---

# 前端开发规范 - HTML/Vue/React 综合指南

## 核心要点

- 命名全部**小写 + 中划线分隔**（项目/目录/文件）
- 禁止拼音与英文混合，禁止中文命名
- HTML 优先语义化标签，CSS 避免嵌套过深
- Vue/React 组件名用 PascalCase，prop 必须完整定义
- 统一 2 空格缩进，ES6+ 语法优先

---

## 一、命名规范

| 对象 | 规则 | 正例 | 反例 |
|------|------|------|------|
| 项目名 | 小写，中划线分隔 | `mall-management-system` | `mallManagementSystem` |
| 目录名 | 小写，中划线，复数 | `scripts` / `components` | `demoScripts` |
| 文件名 | 小写，中划线 | `render-dom.js` | `renderDom.js` |

```js
// ❌ 禁止
const DaZhePromotion = '打折'; // 严禁拼音+中文
const absClass = '...';         // 随意缩写

// ✅ 用国际通用英文
const discountPromotion = '...';
const discountRatio = 0.8;
```

---

## 二、HTML 规范

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
    <meta charset="UTF-8" />
    <title>Page title</title>
  </head>
  <body>
    <!-- 分块注释 -->
    <!-- header start -->
    <header></header>
    <!-- header end -->

    <!-- ✅ 语义化标签 -->
    <header> <footer> <main> <nav> <article>
    <!-- ❌ 全 div -->
    <div class="header"> <div class="main">
  </body>
</html>
```

- 使用 **双引号**，嵌套统一 2 空格缩进

---

## 三、CSS 规范

```css
/* ✅ 类名小写，中划线，语义化 */
.heavy { font-weight: 800; }
.important { color: red; }

/* ❌ 语义模糊 */
.fw-800 { }

/* ✅ 优先直接子选择器 */
.content > .title { font-size: 2rem; }

/* ❌ 嵌套过深 */
.content .box .title .text { }

/* ✅ 缩写属性 */
border-top: 0;
padding: 0 1em 2em;

/* ❌ 冗余写法 */
border-top-style: none;
padding-top: 0; padding-bottom: 2em; padding-left: 1em; padding-right: 1em;
```

---

## 四、JavaScript 规范

```js
// ✅ 小写驼峰，常量大写 + 下划线
const MAX_COUNT = 10;
function getUserName() {}
let userInfo = {};

// ❌ 下划线/美元符开头/结尾
function _getUser() {}

// ✅ 字面值创建对象
const obj = { name: 'test' };
const arr = [];

// ❌ 构造器方式
const obj = new Object();

// ✅ 多行字符串用模板字符串
const msg = `第一行
第二行`;

// ❌ this 敏感场景避免直接使用
const _this = this;
setTimeout(() => { _this.fetch(); }, 1000);
```

---

## 五、Vue 规范

### 组件命名

```js
// ✅ 目录结构
components/
|- BaseButton.vue      // 基础组件：Base 前缀
|- BaseTable.vue
|- TodoList.vue        // 紧密耦合：父名前缀
|- TodoListItem.vue
```

```js
// ✅ props 必须详细定义
props: {
  status: {
    type: String,
    required: true,
    validator: value => ['success', 'warning', 'danger'].includes(value)
  }
}
```

### 模板规范

```html
<!-- ✅ v-for 必须 :key，指令缩写 -->
<li v-for="item in list" :key="item.id" @click="handleClick">
  {{ item.name }}
</li>

<!-- ✅ v-show 频繁切换，v-if 条件渲染 -->
<div v-show="isActive">频繁切换</div>
<div v-if="isLoggedIn">需要登录</div>
```

### 路由

```js
// ✅ path 使用 kebab-case，懒加载
{
  path: '/user-info',
  name: 'UserInfo',
  component: () => import('@/views/user-info.vue')
}
```

---

## 六、React 规范

```js
// ✅ 组件文件统一 .jsx，导出 default，声明 propTypes
import PropTypes from 'prop-types';
function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}
Button.propTypes = {
  label: PropTypes.string.isRequired,
  onClick: PropTypes.func,
};
export default Button;

// ✅ props 回调用 onXxx，高阶组件用 withXxx
function Card({ onDelete }) { ... }
const withLogging = (Component) => { ... };

// ✅ 同一目录不得有同名的 .js 和 .jsx
```
