---
来源: GitHub / Airbnb
原始链接: https://github.com/airbnb/css
---

# Airbnb CSS/Sass 代码规范

## 核心要点

- 使用 **2 空格缩进**，类名用 **kebab-case**（BEM 可用下划线）
- **禁止 ID 选择器** — 特异性过高，不可复用
- CSS 和 JS 不要绑定同一个 class，用 `.js-` 前缀分离
- Sass 最多 **3 层嵌套**，避免 `@extend`
- 属性声明顺序：标准属性 → @include → 嵌套选择器

## CSS 格式规则

```css
/* ✅ Good */
.avatar {
  border-radius: 50%;
  border: 2px solid white;
}

.one,
.selector,
.per-line {
  /* 多选择器每个独占一行 */
}
```

- `{` 前加空格，`:` 后加空格，`}` 独占一行
- 规则声明之间空一行
- 注释独占一行，优先用 `//`（Sass）

## BEM 命名约定

```jsx
// ListingCard.jsx
<article class="ListingCard ListingCard--featured">
  <img class="ListingCard__image" src="..." />
  <h1 class="ListingCard__title">Adorable 2BR</h1>
</article>
```

```css
.ListingCard { }           /* Block - 高层组件 */
.ListingCard--featured { } /* Modifier - 状态/变体 */
.ListingCard__title { }    /* Element - 后代组成部分 */
```

**推荐 PascalCase 的 Block**，配合 React 组件使用。

## 避免的模式

### ❌ ID 选择器
```css
/* ❌ Bad */
#sidebar { ... }

/* ✅ Good */
.sidebar { ... }
```

### ❌ CSS/JS 绑定同一个 class
```html
<!-- ✅ Good: JS 专用前缀 -->
<button class="btn btn-primary js-request-to-book">Request</button>
```

### ❌ border: none
```css
/* ❌ Bad */
.foo { border: none; }

/* ✅ Good */
.foo { border: 0; }
```

## Sass 规范

### 属性声明顺序
```scss
.btn {
  background: green;           // 1. 标准属性
  font-weight: bold;
  @include transition(...);    // 2. @include

  .icon {                      // 3. 嵌套选择器
    margin-right: 10px;
  }
}
```

### 变量与嵌套
- 变量用 `kebab-case`：`$my-variable`
- 私有变量加下划线：`$_my-variable`
- **最多 3 层嵌套**，超过说明 CSS 与 HTML 强耦合
- **永远不要嵌套 ID 选择器**
- **避免 `@extend`** — 行为不直观，gzip 能替代其优化
