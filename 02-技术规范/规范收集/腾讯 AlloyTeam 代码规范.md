# 腾讯 AlloyTeam 代码规范

> **来源**: [AlloyTeam/CodeGuide](https://github.com/AlloyTeam/CodeGuide)
> **Star**: 1.3k | **Fork**: 305
> **特点**: 最佳实践积累，团队协作导向

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **命名** | camelCase变量，PascalCase类，_下划线私有 |
| **缩进** | 4空格 |
| **引号** | 优先单引号 |
| **分号** | 必须使用 |
| **变量** | const > let > var |
| **私有** | 下划线前缀 `_privateMethod()` |

---

## 一、命名规范

| 类型 | 规则 | 示例 |
|------|------|------|
| 项目命名 | 小写，中线分隔 | `my-project-name` |
| 目录命名 | 小写，中线分隔 | `scripts/` |
| JS/CSS文件 | 小写，下划线分隔 | `my_module.js` |
| 变量/函数 | camelCase | `getUserName()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_COUNT` |
| 类/构造函数 | PascalCase | `class UserInfo {}` |
| 私有属性 | 下划线前缀 | `_initData()` |

### 命名严谨性

```javascript
// ✅ 使用完整单词
const userName = 'John';

// ❌ 避免缩写（常见约定俗成的除外）
const usr = 'John';  // ❌
const user = 'John';  // ✅

// ❌ 避免无意义命名
const data = {};
const temp = {};

// ✅ 命名要具有描述性
const userList = [];
const tempData = {};  // 临时数据应尽快使用或重命名
```

---

## 二、HTML 规范

### 基本规范

```html
<!-- ✅ 使用语义化标签 -->
<header></header>
<nav></nav>
<main></main>
<article></article>
<section></section>
<footer></footer>

<!-- ❌ 避免无语义标签 -->
<div class="header"></div>
<div class="nav"></div>

<!-- ✅ 标签闭合 -->
<div>...</div>
<img src="..." alt="描述">

<!-- ✅ 属性顺序 -->
<!-- id > class > name > data-xxx > src/for/type/href > 其他自定义属性 -->
<input id="username" class="input" name="username" type="text">

<!-- ✅ 布尔属性不写值 -->
<input type="text" disabled>
<input type="checkbox" checked>
```

### 注释规范

```html
<!-- 模块描述 start -->
<div class="module">
  <!-- 子模块A -->
  <div class="sub-module-a">
    ...
  </div>
  <!-- /子模块A -->
</div>
<!-- /模块描述 end -->
```

---

## 三、CSS 规范

### 命名规范 (BEM)

```css
/* Block */
.search { }

/* Element */
.search__input { }
.search__button { }

/* Modifier */
.search--large { }
.search__button--primary { }
.search__button--disabled { }
```

### 属性书写顺序

```css
/* ✅ 推荐顺序 */
.selector {
  /* 定位 */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* 盒模型 */
  display: block;
  width: 100px;
  height: 100px;
  margin: 10px;
  padding: 10px;
  border: 1px solid #ccc;

  /* 文字 */
  font-size: 14px;
  font-weight: bold;
  color: #333;
  text-align: center;

  /* 背景装饰 */
  background-color: #f5f5f5;
  border-radius: 4px;

  /* 其他 */
  opacity: 1;
  cursor: pointer;
}
```

### 其他规范

```css
/* ✅ 使用类选择器，避免标签选择器 */
.list .item { }      /* ✅ */
.list li { }         /* ❌ 性能差 */

/* ✅ 使用组合选择器时注意权重 */
.class-name { }       /* ✅ 单一类名 */
.input-text { }      /* ✅ */
ul.list { }          /* ✅ 具体标签 */

/* ❌ 避免 !important */
.element { color: red !important; }

/* ✅ 媒体查询放文件底部或单独文件 */
@media screen and (max-width: 768px) { }

/* ✅ 0 值不加单位 */
margin: 0;
padding: 0;
```

---

## 四、JavaScript 规范

### 基本规范

```javascript
// ✅ 缩进：4空格
function greet(name) {
    if (name === 'world') {
        console.log('Hello, World!');
    }
}

// ✅ 引号：优先单引号
const str = 'Hello, World!';
const html = '<div class="container">content</div>';

// ✅ 分号：必须使用
const foo = 'foo';

// ✅ 变量声明：const > let > var
const PI = 3.14;
let count = 0;
```

### 函数规范

```javascript
// ✅ 使用箭头函数
const add = (a, b) => a + b;

// ✅ 函数参数不超过3个
// ❌ 超过3个参数应使用对象
function createUser(name, age, email, phone) { }
// ✅ 改为
function createUser({ name, age, email, phone }) { }

// ✅ 使用默认参数
function greet(name = 'World') {
    return `Hello, ${name}!`;
}
```

### 对象与数组

```javascript
// ✅ 使用字面量
const obj = {};
const arr = [];

// ✅ 对象方法简写
const utils = {
    add(a, b) {
        return a + b;
    }
};

// ✅ 使用扩展运算符
const copy = { ...original };
const merged = { ...obj1, ...obj2 };

// ✅ 使用解构
const { name, age } = user;
```

### 注释规范

```javascript
/**
 * 描述函数功能的注释
 * @param {string} name - 参数说明
 * @returns {boolean} 返回值说明
 */
function isValid(name) {
    // 单行注释：说明为什么
    if (name.length > 0) {
        return true;
    }

    return false;
}
```

---

## 五、工具链

| 工具 | 用途 |
|------|------|
| jscs | 代码风格检查 |
| jshint | JavaScript 语法检查 |
| sass-lint | SCSS 语法检查 |
| csslint | CSS 语法检查 |
| JsFormat | 代码格式化 |
| CSScomb | CSS 格式化 |
| Sublime3 + Grunt | 推荐编辑器配置 |

---

## AI 编程调用示例

```markdown
请遵循腾讯 AlloyTeam 代码规范：
- 变量/函数用 camelCase，类用 PascalCase
- 私有属性以下划线 _ 开头
- 4空格缩进
- 优先使用单引号
- 必须使用分号
- const > let > var
- CSS 使用 BEM 命名法
- HTML 使用语义化标签
- 函数参数不超过3个
```
