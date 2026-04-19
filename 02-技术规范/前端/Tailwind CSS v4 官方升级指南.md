---
来源: 官方文档
领域: 前端
关键词: Tailwind CSS, v4, 升级指南, 迁移, CSS优先配置, @theme, @utility, @plugin, @import
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://tailwindcss.com/docs/upgrade-guide
---

# Tailwind CSS v4 官方升级指南

> 来源：Tailwind CSS 官方文档 https://tailwindcss.com/docs/upgrade-guide

Tailwind CSS v4.0 是新的大版本，虽然已尽可能减少破坏性变更，但仍有必要了解升级步骤。

**浏览器要求**：Tailwind CSS v4.0 需要 Safari 16.4+、Chrome 111+、Firefox 128+。如需支持旧浏览器，请继续使用 v3.4。

---

## 1. 使用升级工具（推荐）

```bash
$ npx @tailwindcss/upgrade
```

升级工具会自动完成：更新依赖、将配置文件迁移为 CSS、处理模板文件变更。

**要求**：Node.js 20 或更高版本。

建议在新分支中运行升级工具，然后仔细检查 diff 并在浏览器中测试项目。

---

## 2. 手动升级

### 2.1 使用 PostCSS

v3 中 `tailwindcss` 包是 PostCSS 插件，v4 中 PostCSS 插件移至专用 `@tailwindcss/postcss` 包：

```js
// postcss.config.mjs — v3
export default {
  plugins: {
    "postcss-import": {},
    tailwindcss: {},
    autoprefixer: {},
  },
};

// postcss.config.mjs — v4
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

> v4 自动处理 imports 和 vendor prefixing，可移除 `postcss-import` 和 `autoprefixer`。

### 2.2 使用 Vite（推荐方式）

推荐从 PostCSS 插件迁移到专用的 Vite 插件，以获得更好的性能和开发体验：

```ts
// vite.config.ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
});
```

### 2.3 使用 Tailwind CLI

v4 中 CLI 移至专用 `@tailwindcss/cli` 包：

```bash
# v3
npx tailwindcss -i input.css -o output.css
# v4
npx @tailwindcss/cli -i input.css -o output.css
```

---

## 3. 从 v3 的破坏性变更

### 3.1 移除 @tailwind 指令

```css
/* v3 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* v4 */
@import "tailwindcss";
```

### 3.2 移除已废弃的工具类

| 已废弃 | 替代方案 |
|--------|---------|
| `bg-opacity-*` | 使用不透明度修饰符如 `bg-black/50` |
| `text-opacity-*` | 使用不透明度修饰符如 `text-black/50` |
| `border-opacity-*` | 使用不透明度修饰符如 `border-black/50` |
| `flex-shrink-*` | `shrink-*` |
| `flex-grow-*` | `grow-*` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` |

### 3.3 重命名的工具类

| v3 | v4 |
|----|-----|
| `shadow-sm` | `shadow-xs` |
| `shadow` | `shadow-sm` |
| `drop-shadow-sm` | `drop-shadow-xs` |
| `drop-shadow` | `drop-shadow-sm` |
| `blur-sm` | `blur-xs` |
| `blur` | `blur-sm` |
| `backdrop-blur-sm` | `backdrop-blur-xs` |
| `backdrop-blur` | `backdrop-blur-sm` |
| `rounded-sm` | `rounded-xs` |
| `rounded` | `rounded-sm` |
| `outline-none` | `outline-hidden` |
| `ring` | `ring-3` |

### 3.4 outline 工具类变更

`outline` 工具类现在默认设置 `outline-width: 1px`，与 `border` 和 `ring` 保持一致。所有 `outline-<number>` 工具类默认 `outline-style: solid`：

```html
<!-- v3 -->
<input class="outline outline-2" />
<!-- v4 -->
<input class="outline-2" />
```

### 3.5 ring 默认宽度变更

v3 中 `ring` 工具类添加 3px 的环，v4 改为 1px 以与 border 和 outline 一致：

```html
<!-- v3 -->
<input class="ring ring-blue-500" />
<!-- v4 -->
<input class="ring-3 ring-blue-500" />
```

### 3.6 默认边框颜色变更

v3 中 `border-*` 和 `divide-*` 默认使用 gray-200，v4 改为 `currentColor`：

```html
<!-- 需要显式指定颜色 -->
<div class="border border-gray-200 px-2 py-3">
```

如需保留 v3 行为，添加：

```css
@layer base {
  *, ::after, ::before, ::backdrop, ::file-selector-button {
    border-color: var(--color-gray-200, currentColor);
  }
}
```

### 3.7 space 和 divide 选择器变更

为解决大页面上的严重性能问题，选择器已变更：

```css
/* v3 */
.space-y-4 > :not([hidden]) ~ :not([hidden]) {
  margin-top: 1rem;
}
/* v4 */
.space-y-4 > :not(:last-child) {
  margin-bottom: 1rem;
}
```

如有问题，建议迁移到 flex/grid 布局并使用 `gap`：

```html
<!-- 推荐 -->
<div class="flex flex-col gap-4 p-4">
```

### 3.8 渐变工具类与变体

v4 中变体覆盖渐变时不再重置整个渐变：

```html
<!-- v4：dark:from-blue-500 不会重置 to-* 颜色 -->
<div class="bg-gradient-to-r from-red-500 to-yellow-400 dark:from-blue-500">
```

如需取消三段渐变的中间点：

```html
<div class="bg-linear-to-r from-red-500 via-orange-400 to-yellow-400 dark:via-none dark:from-blue-500 dark:to-teal-400">
```

---

## 4. 配置迁移

### 4.1 主题配置（@theme）

v3 使用 `tailwind.config.js` 的 `theme.extend`，v4 改为 CSS 文件中的 `@theme` 块：

```css
@import "tailwindcss";

@theme {
  --color-brand: #6366f1;
  --color-brand-light: #818cf8;
  --color-brand-dark: #4f46e5;
  --font-sans: 'Inter', system-ui, sans-serif;
  --breakpoint-3xl: 1920px;
  --spacing-18: 4.5rem;
}
```

### 4.2 插件配置（@plugin）

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```

### 4.3 自定义工具类（@utility）

```css
@import "tailwindcss";

@utility text-gradient {
  background: linear-gradient(135deg, #6366f1, #a855f7);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
```

### 4.4 自定义变体（@custom-variant）

```css
@custom-variant children (& > *);

/* 使用 */
<div class="children:mb-2">
```

---

## 5. Preflight 变更

### 5.1 占位符颜色

v3 使用 gray-400，v4 改为当前文字颜色 50% 透明度。

### 5.2 按钮光标

按钮现在使用 `cursor: default` 而非 `cursor: pointer`，匹配浏览器默认行为。

如需恢复：

```css
@layer base {
  button:not(:disabled), [role="button"]:not(:disabled) {
    cursor: pointer;
  }
}
```

### 5.3 hidden 属性优先级

`display` 类（如 `block`、`flex`）不再覆盖 `hidden` 属性。

---

## 6. 前缀与 important 变更

### 6.1 前缀

前缀现在看起来像变体，始终位于类名开头：

```html
<div class="tw:flex tw:bg-red-500 tw:hover:bg-red-600">
```

### 6.2 important 修饰符

v4 中 `!` 应放在类名末尾：

```html
<div class="flex! bg-red-500! hover:bg-red-600/50!">
```

---

## 7. 容器工具类

v3 的 `container` 工具类的 `center` 和 `padding` 选项在 v4 中不存在，需使用 `@utility` 自定义：

```css
@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
}
```

---

## 8. 完整迁移示例

### v3 → v4 对照

| 对比项 | v3 | v4 |
|--------|----|----|
| 配置位置 | `tailwind.config.js` | CSS 文件 |
| 主题配置 | `theme.extend` | `@theme` 块 |
| 插件配置 | `plugins` 数组 | `@plugin` 指令 |
| 自定义类 | JavaScript 插件 | `@utility` 指令 |
| 自定义变体 | JavaScript 插件 | `@custom-variant` |
| CSS 导入 | `@tailwind base/components/utilities` | `@import "tailwindcss"` |

**一句话总结**：Tailwind v4 从 JavaScript 配置转向 CSS 优先，配置更直观，性能更好，但需要按升级指南迁移现有项目。
