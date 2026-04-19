---
来源: Google 官方文档
领域: 前端
关键词: HTML, CSS, Google, 风格指南, 编码规范, Web标准
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://google.github.io/styleguide/htmlcssguide.html
---

# Google HTML/CSS 风格指南

> 来源：Google 官方风格指南  
> 原文：https://google.github.io/styleguide/htmlcssguide.html  
> 版本：Revision 2.1

本文档定义了 HTML/CSS 的编写格式和风格规则，旨在提高协作效率、代码质量并支持基础设施。

---

## 一、通用风格规则

### 1.1 协议（Protocol）

尽可能对嵌入式资源使用 HTTPS。

```html
<!-- 不推荐：省略协议 -->
<script src="//ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>

<!-- 不推荐：使用 HTTP -->
<script src="http://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>

<!-- 推荐 -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>
```

```css
/* 不推荐：省略协议 */
.example {
  background: url(//www.google.com/images/example);
}

/* 不推荐：使用 HTTP */
.example {
  background: url(http://www.google.com/images/example);
}

/* 推荐 */
.example {
  background: url(https://www.google.com/images/example);
}
```

---

## 二、通用排版规则

### 2.1 缩进（Indentation）

每次缩进使用 **2 个空格**，不要使用 Tab 或混合使用 Tab 和空格。

```html
<!-- 推荐 -->
<ul>
  <li>Fantastic
  <li>Great
</ul>
```

```css
/* 推荐 */
.example {
  color: blue;
}
```

### 2.2 大小写（Capitalization）

所有代码只使用小写，包括 HTML 元素名称、属性、属性值（text/CDATA 除外）、CSS 选择器、属性和属性值。

```html
<!-- 不推荐 -->
<A HREF="/">Home</A>

<!-- 推荐 -->
<img src="google.png" alt="Google">
```

```css
/* 不推荐 */
color: #E5E5E5;

/* 推荐 */
color: #e5e5e5;
```

### 2.3 行尾空格（Trailing Whitespace）

删除行尾空格，它们是多余的。

---

## 三、通用元数据规则

### 3.1 编码（Encoding）

使用 **UTF-8 编码**（不带 BOM 头）。

在 HTML 中指定编码：
```html
<meta charset="utf-8">
```

在 CSS 中指定编码（首行）：
```css
@charset "utf-8";
```

### 3.2 注释（Comments）

尽可能解释代码，注释应该说明「是什么」和「为什么」，而不是「怎么做」。

### 3.3 待办事项标记（Action Items）

使用 `TODO` 标记待办事项，格式为 `TODO(contact)`。

```html
<!-- TODO(john.doe): 重新设计这个组件 -->
```

```css
/* TODO(jane.doe): 优化选择器性能 */
```

---

## 四、HTML 风格规则

### 4.1 文档类型（Document Type）

使用 HTML5 文档类型：
```html
<!DOCTYPE html>
```

### 4.2 HTML 有效性（HTML Validity）

尽量使用有效的 HTML 代码，使用 [W3C HTML Validator](https://validator.w3.org/nu/) 等工具测试。

### 4.3 语义化（Semantics）

根据 HTML 元素的用途使用它们，使用语义化标签（如 `<article>`, `<section>`, `<nav>` 等）。

```html
<!-- 不推荐 -->
<div onclick="goToRecommendations();">All recommendations</div>

<!-- 推荐 -->
<a href="recommendations/">All recommendations</a>
```

### 4.4 多媒体后备方案（Multimedia Fallback）

为多媒体提供替代内容。

```html
<!-- 不推荐 -->
<img src="spreadsheet.png">

<!-- 推荐 -->
<img src="spreadsheet.png" alt="Spreadsheet screenshot.">
```

### 4.5 关注点分离（Separation of Concerns）

将结构（HTML）、表现（CSS）和行为（JavaScript）分离。

```html
<!-- 不推荐 -->
<p style="font-weight: bold;">Important</p>
<p class="warning" onclick="alert('Warning!')">Warning</p>

<!-- 推荐 -->
<p class="important">Important</p>
<p class="warning" id="warning-msg">Warning</p>
```

```css
.important {
  font-weight: bold;
}
```

```javascript
document.getElementById('warning-msg').addEventListener('click', showWarning);
```

### 4.6 实体引用（Entity References）

不要使用实体引用（如 `&mdash;`, `&rdquo;`, `&#x263a;`），直接使用 UTF-8 字符。

```html
<!-- 不推荐 -->
<p>The currency symbol for the Euro is &ldquo;&eur;&rdquo;.</p>

<!-- 推荐 -->
<p>The currency symbol for the Euro is "€".</p>
```

### 4.7 可选标签（Optional Tags）

省略可选标签（如 `<html>`, `<head>`, `<body>`, `<p>`, `<li>` 的结束标签等）。

```html
<!-- 不推荐 -->
<!DOCTYPE html>
<html>
  <head>
    <title>Spending money, spending bytes</title>
  </head>
  <body>
    <p>Sic.</p>
  </body>
</html>

<!-- 推荐 -->
<!DOCTYPE html>
<title>Saving money, saving bytes</title>
<p>Qed.
```

### 4.8 type 属性（type Attributes）

在引用样式表和脚本时，不要指定 `type` 属性（CSS 和 JavaScript 是默认值）。

```html
<!-- 不推荐 -->
<link rel="stylesheet" href="https://www.google.com/css/maia.css" type="text/css">
<script src="https://www.google.com/js/gweb/analytics/autotrack.js" type="text/javascript"></script>

<!-- 推荐 -->
<link rel="stylesheet" href="https://www.google.com/css/maia.css">
<script src="https://www.google.com/js/gweb/analytics/autotrack.js"></script>
```

---

## 五、HTML 格式规则

### 5.1 通用格式（General Formatting）

每个块元素、列表元素或表格元素都独占一行，每个子元素都相对于父元素进行缩进。

```html
<!-- 推荐 -->
<blockquote>
  <p>
    <em>Space</em>, the final frontier.
  </p>
</blockquote>

<ul>
  <li>Moe
  <li>Larry
  <li>Curly
</ul>

<table>
  <thead>
    <tr>
      <th scope="col">Income
      <th scope="col">Taxes
  <tbody>
    <tr>
      <td>$ 5.00
      <td>$ 4.50
</table>
```

---

## 六、CSS 风格规则

### 6.1 CSS 有效性（CSS Validity）

尽量使用有效的 CSS 代码，使用 [W3C CSS Validator](https://jigsaw.w3.org/css-validator/) 等工具测试。

### 6.2 ID 和 Class 命名（ID and Class Naming）

使用有意义或通用的 ID 和 class 名称。

```css
/* 不推荐：无意义 */
#yee-1901 {}

/* 不推荐：呈现化 */
.button-green {}
.clear {}

/* 推荐：具体 */
#gallery {}
#login {}
.video {}

/* 推荐：通用 */
.aux {}
.alt {}
```

### 6.3 ID 和 Class 命名风格（ID and Class Name Style）

ID 和 class 名称应该尽可能短，但又要足够长以表达含义。

```css
/* 不推荐 */
.navigation {}
.atr {}

/* 推荐 */
.nav {}
.author {}
```

### 6.4 类型选择器（Type Selectors）

避免在 ID 和 class 名称前使用元素类型选择器。

```css
/* 不推荐 */
ul#example {}
div.error {}

/* 推荐 */
#example {}
.error {}
```

### 6.5 属性缩写（Shorthand Properties）

尽量使用简写属性。

```css
/* 不推荐 */
border-top-style: none;
font-family: palatino, georgia, serif;
font-size: 100%;
line-height: 1.6;
padding-bottom: 2em;
padding-left: 1em;
padding-right: 1em;
padding-top: 0;

/* 推荐 */
border-top: 0;
font: 100%/1.6 palatino, georgia, serif;
padding: 0 1em 2em;
```

### 6.6 0 和单位（0 and Units）

省略值为 0 时的单位。

```css
/* 不推荐 */
margin: 0px;
padding: 0em;

/* 推荐 */
margin: 0;
padding: 0;
```

### 6.7 0 开头的小数（Leading 0s）

省略小数点前的 0。

```css
/* 不推荐 */
font-size: 0.8em;

/* 推荐 */
font-size: .8em;
```

### 6.8 URI 外的引号（Quotation Marks in URI Values）

URI 值中不要使用引号。

```css
/* 不推荐 */
@import url("https://www.google.com/css/maia.css");

/* 推荐 */
@import url(https://www.google.com/css/maia.css);
```

### 6.9 十六进制表示法（Hexadecimal Notation）

在可能的情况下使用 3 个字符的十六进制表示法。

```css
/* 不推荐 */
color: #eebbcc;

/* 推荐 */
color: #ebc;
```

### 6.10 前缀（Prefixes）

选择器前面加上特殊应用标识的前缀（可选）。

```css
.adw-help {}
#maia-note {}
```

### 6.11 ID 和 Class 命名的定界符（ID and Class Name Delimiters）

ID 和 class 名字有多单词组合的用短破折号 `-` 分开。

```css
/* 不推荐 */
.demoimage {}
.error_status {}

/* 推荐 */
.video-container {}
.tweet-post {}
```

### 6.12 Hacks

避免使用 CSS hacks，先尝试使用其他解决方法。

---

## 七、CSS 格式规则

### 7.1 声明顺序（Declaration Order）

按字母顺序排列声明（便于记忆和查找）。

```css
/* 不推荐 */
background: fuchsia;
border: 1px solid;
-moz-border-radius: 4px;
-webkit-border-radius: 4px;
border-radius: 4px;
color: black;
text-align: center;

/* 推荐 */
background: fuchsia;
border: 1px solid;
-moz-border-radius: 4px;
-webkit-border-radius: 4px;
border-radius: 4px;
color: black;
text-align: center;
```

### 7.2 代码块内容缩进（Block Content Indentation）

缩进所有块内容（规则内的内容）。

```css
/* 不推荐 */
@media screen, projection {
html {
background: #fff;
color: #444;
}
}

/* 推荐 */
@media screen, projection {
  html {
    background: #fff;
    color: #444;
  }
}
```

### 7.3 声明完结（Declaration Stops）

每个声明以分号结尾。

```css
/* 不推荐 */
.test {
  display: block;
  height: 100px
}

/* 推荐 */
.test {
  display: block;
  height: 100px;
}
```

### 7.4 属性名完结（Property Name Stops）

属性名后使用冒号和一个空格。

```css
/* 不推荐 */
h3 {
  font-weight:bold;
}

/* 推荐 */
h3 {
  font-weight: bold;
}
```

### 7.5 选择器和声明分行（Selector and Declaration Separation）

每个选择器和声明都单独成行。

```css
/* 不推荐 */
h1, h2, h3 {
  font-weight: normal; line-height: 1.2;
}

/* 推荐 */
h1,
h2,
h3 {
  font-weight: normal;
  line-height: 1.2;
}
```

### 7.6 规则分行（Rule Separation）

规则之间用空行分隔。

```css
html {
  background: #fff;
}

body {
  margin: auto;
  width: 50%;
}
```

---

## 八、CSS 元数据规则

### 8.1 注释部分（Section Comments）

按组件对规则进行分组，并在每组前添加注释。

```css
/* Header */
header {
  /* ... */
}

/* Footer */
footer {
  /* ... */
}
```

---

## 九、核心要点总结

| 类别 | 核心原则 |
|------|----------|
| **编码** | 使用 UTF-8（无 BOM） |
| **大小写** | 全部使用小写 |
| **缩进** | 2 个空格 |
| **协议** | 始终使用 HTTPS |
| **命名** | 使用短横线连接多单词（kebab-case） |
| **语义** | 使用语义化 HTML 标签 |
| **分离** | HTML/CSS/JS 关注点分离 |
| **有效性** | 保持代码有效，通过验证器检查 |

---

## 参考链接

- [Google HTML/CSS Style Guide (官方)](https://google.github.io/styleguide/htmlcssguide.html)
- [W3C HTML Validator](https://validator.w3.org/nu/)
- [W3C CSS Validator](https://jigsaw.w3.org/css-validator/)
