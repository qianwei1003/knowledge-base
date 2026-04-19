---
来源: 官方文档
原始链接: https://getbem.com/ https://www.bemcss.com/
---

# BEM CSS 命名规范

## 什么是 BEM

BEM（Block Element Modifier）是一种 CSS 命名方法论，由 Yandex 团队开发，旨在帮助开发者创建可复用的组件和实现代码共享。

BEM 的核心思想是将用户界面分解为三个抽象层级：
- **Block（块）**：独立可复用的功能单元（如导航菜单、按钮）
- **Element（元素）**：属于块的组成部分（如菜单项、按钮图标）
- **Modifier（修饰符）**：表示块或元素的状态/变体（如禁用状态、不同尺寸）

## 命名约定

### 基本格式

```
.block__element--modifier
```

- `block`：模块名，使用 kebab-case（短横线分隔）
- `__`：元素分隔符（双下划线）
- `element`：元素名
- `--`：修饰符分隔符（双连字符）
- `modifier`：修饰符名

### 命名示例

```html
<!-- 块 -->
<div class="card">
  <!-- 元素 -->
  <h2 class="card__title">标题</h2>
  <p class="card__text">内容</p>
  <!-- 带修饰符的元素 -->
  <button class="card__button card__button--primary">提交</button>
</div>

<!-- 块修饰符 -->
<div class="card card--featured">
  ...
</div>
```

## BEM 解决的问题

### 1. 样式冲突

传统 CSS 样式是全局性的，没有作用域。组件样式容易相互覆盖。

**问题场景**：开发一个弹窗组件，在其他页面使用时样式冲突。

**BEM 解决方案**：通过组件名的唯一性保证选择器的唯一性。

```css
/* 错误：可能与其他组件冲突 */
.modal .title { }

/* 正确：BEM 命名 */
.modal__title { }
```

### 2. 选择器权重问题

使用嵌套选择器会导致权重过大，难以覆盖。

```css
/* 错误：嵌套选择器权重过大 */
.page-btn button:first-child { }
.page-btn ul li a { }

/* 正确：BEM 单类选择器 */
.page-btn__prev { }
.page-btn__link { }
```

### 3. 维护困难

子代选择器依赖 DOM 结构，结构变化时需要重写样式。

```css
/* 错误：依赖结构，结构变化需重写 */
.header .nav .item .link { }

/* 正确：BEM 不依赖结构 */
.header__link { }
```

## 核心原则

### 1. 不使用子代选择器

BEM 禁止使用子代选择器定位元素。

```css
/* 错误 */
.block .element { }
.block > .element { }

/* 正确 */
.block__element { }
```

### 2. 元素名不包含多个部分

```html
<!-- 错误：多个元素名 -->
<div class="block__element1__element2"></div>

<!-- 正确：元素名只有一层 -->
<div class="block__element1">
  <div class="block__element2"></div>
</div>
```

### 3. 修饰符独立使用

修饰符不能单独使用，必须与块或元素一起使用。

```html
<!-- 错误：修饰符单独使用 -->
<div class="--active"></div>

<!-- 正确：修饰符与块一起使用 -->
<div class="block block--active"></div>
```

## 最佳实践

### CSS 预处理器中的 BEM

```scss
// 使用 Sass 嵌套简化书写
.card {
  &__title {
    font-size: 18px;
  }
  
  &__text {
    color: #666;
  }
  
  &--featured {
    border: 2px solid gold;
    
    .card__title {
      color: gold;
    }
  }
}
```

编译后：
```css
.card__title { font-size: 18px; }
.card__text { color: #666; }
.card--featured { border: 2px solid gold; }
.card--featured .card__title { color: gold; }
```

### JavaScript 状态类名

对于 JS 控制的状态，推荐使用独立的状态类名：

```css
/* BEM 修饰符 - 用于静态样式变体 */
.button--primary { background: blue; }

/* JS 状态类 - 用于动态状态 */
.button.is-active { background: green; }
.button.is-disabled { opacity: 0.5; }
```

```html
<button class="button button--primary is-active">
  激活按钮
</button>
```

### 原子类与 BEM 配合

实用工具类（如 clearfix、ellipsis）可以与 BEM 配合使用：

```html
<div class="card clearfix">
  <h2 class="card__title text-ellipsis">标题</h2>
</div>
```

## 命名风格变体

团队可根据习惯调整分隔符：

| 风格 | 示例 |
|------|------|
| 经典 BEM | `block__element--modifier` |
| 驼峰式 | `blockName-elementName--modifierName` |
| 单下划线 | `block_element--modifier` |
| 单连字符修饰符 | `block__element-modifier` |

> 注意：保持团队统一比选择哪种风格更重要。

## 使用 BEM 的公司

- Yandex
- JetBrains
- Alfa Bank
- Megafon
- EPAM

## 参考资源

- [BEM 官方网站](https://getbem.com/)
- [CSS-BEM 命名规范（中文）](https://www.bemcss.com/)
- [Yandex BEM 文档](https://en.bem.info/)
