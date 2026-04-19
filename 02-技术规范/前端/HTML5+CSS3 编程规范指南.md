---
来源: 腾讯云开发者社区
原始链接: https://cloud.tencent.com/developer/article/2527574
---

# HTML5+CSS3 编程规范指南

## 核心要点

- 命名使用语义化英文，短横线连接，禁止数字标号和驼峰混用
- img 标签必须包含 src/width/height/alt 四要素，width/height 只写数字不带单位
- CSS 选择器嵌套不超过 3 级，属性书写按 定位→自身→文本→其他 顺序
- 链接伪类严格遵循 LVHA 顺序（link→visited→hover→active）

## HTML 命名规范

### 语义化命名

使用有语义的英文单词，禁止拼音、数字标号、特殊字符。

- ✅ `wrap`, `description`, `content-product`
- ❌ `aaaa`, `content1`, `$we`, `slideBar`

### 连接符

多单词用短横线 `-` 连接，风格统一，不混用驼峰。

- ✅ `header-nav`, `content-left`
- ❌ `headernav`, `ContentLeft`

### 层级控制

命名不超过 3-4 层结构，禁止数字标号。

- ✅ `head-tit-ico`
- ❌ `header-title-left-logo-icon`, `content1`

### 其他

- id 仅用于模块/一级结构，必须唯一，禁止与 class 重名
- 禁止修改线上项目的 id 名称

### 常用命名速查

| 中文 | 英文 | 中文 | 英文 |
|------|------|------|------|
| 头部 | header | 尾部 | footer |
| 导航 | nav | 主体 | main |
| 侧栏 | sidebar | 横幅 | banner |
| 菜单 | menu | 子菜单 | submenu |
| 标题 | title | 状态 | status |
| 按钮 | btn | 搜索 | search |
| 版权 | copyright | 标志 | logo |
| 面包屑 | breadcrumb | 外套 | wrap |

## img 标签四要素

**src、width、height、alt** 必须齐全。width/height 只填数字不带单位。

```html
<!-- ✅ 正确 -->
<img src="images/sample.png" width="125" height="125" alt="示例图">

<!-- ❌ 错误：缺少 alt，width 带了单位 -->
<img src="images/sample.png" width="125px" height="225px">
```

## HTML 书写规则

- 所有标签小写，双标签必须闭合
- 子级缩进 2 空格，属性值统一双引号
- 禁止连续两行空行
- ul/ol 内只能嵌套 li；a 不可嵌套 a；行内元素不嵌套块元素
- p/dt/h 内不可嵌套块元素

```html
<!-- 模块注释格式 -->
<!-- header module start -->
<div class="header">
  <h1>页面标题</h1>
</div>
<!-- header module end -->
```

## 图片文件命名

- 后缀一律小写，短横线连接
- 背景图 `bg-` 开头，按钮图 `btn-` 开头，图标 `icon-` 开头，精灵图 `spr-` 开头

## CSS 书写规范

### 注释

```css
/* 文件顶部 */
/* @description: 页面样式
   @author: zhifu.wang
   @update: zhifu.wang (2024-10-17) */

/* 模块注释 */
/* module: header styles */
.header { background-color: #f0f0f0; }

/* 特殊注释 */
/* TODO: 适配移动端 */
/* BUGFIX: 修复按钮边距 */
```

### 空格与格式

- 选择器与 `{` 之间必须有空格
- 属性名与冒号之间无空格，冒号与值之间必须有空格
- 属性定义另起一行，以分号结尾
- 选择器嵌套不超过 3 级
- 并集选择器逗号后换行

```css
/* ✅ */
.selector {
  margin: 0;
  padding: 0;
}

body,
ul,
p {
  margin: 0;
}

/* ❌ 选择器嵌套过深 */
.page .header .login #username input {}
```

### 属性值

- 全部小写，引号统一双引号
- 值为 0 时省略单位：`margin: 0`
- 背景 URL 不加引号：`background-image: url(images/bg.jpg)`

### 兼容前缀顺序

```css
.selector {
  -webkit-transition: all 0.3s ease;
  -moz-transition: all 0.3s ease;
  -o-transition: all 0.3s ease;
  transition: all 0.3s ease;
}
```

### CSS 书写顺序

1. **定位**：display, position, float, z-index, overflow, clear
2. **自身**：width, height, margin, padding, border, background
3. **文本**：color, font-*, text-*, line-height
4. **其他**：transform, box-shadow, animation

### 链接伪类顺序

严格 LVHA：`a:link → a:visited → a:hover → a:active`

## CSS 优化要点

- 使用缩写属性（margin、padding、border）
- 值为 0 省略单位
- 重复小图标使用 CSS Sprite 减少 HTTP 请求
- HTML5 可省略 script/style 的 type 属性
