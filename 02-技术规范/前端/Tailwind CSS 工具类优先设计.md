---
来源: Tailwind CSS 官方文档
原始链接: https://tailwindcss.com/docs/styling-with-utility-classes
---

# Tailwind CSS 工具类优先设计

## 核心要点

- 通过组合单用途工具类直接在 HTML 中设计样式，无需自定义 CSS
- 相比内联样式支持状态变体（hover/focus）、响应式、暗黑模式
- 扫描项目文件按需生成 CSS，不会产生冗余样式表
- 任意值语法 `[...]` 覆盖设计系统之外的定制需求

## 为什么工具类优先

| 优势 | 说明 |
|------|------|
| 开发更快 | 不取名、不切换文件，设计快速成型 |
| 修改安全 | 添加/删除类只影响当前元素 |
| 维护简单 | 改样式 = 改 HTML，不需记忆自定义 CSS |
| 可移植 | 结构和样式在同一处，跨项目复制容易 |
| 不膨胀 | 工具类高度复用，新功能不线性增加 CSS |

## 对比内联样式

| 能力 | 内联样式 | Tailwind 工具类 |
|------|---------|----------------|
| 设计约束 | 魔法数字 | 预定义设计系统 |
| 状态样式 | ❌ 不支持 | ✅ `hover:bg-sky-700` |
| 响应式 | ❌ 不支持 | ✅ `sm:grid-cols-3` |
| 暗黑模式 | ❌ 不支持 | ✅ `dark:bg-gray-800` |

## 状态变体（Variants）

✅ Hover / Focus：

```html
<button class="bg-sky-500 hover:bg-sky-700 active:bg-sky-900">Save</button>
```

✅ 响应式：

```html
<div class="grid grid-cols-2 sm:grid-cols-3"><!-- sm(40rem) 以上 3 列 --></div>
```

✅ 暗黑模式：

```html
<div class="bg-white dark:bg-gray-800 rounded-lg px-6 py-8"></div>
```

✅ 组合使用：

```html
<button class="bg-sky-500 disabled:hover:bg-sky-500"><!-- disabled 时 hover 无效 --></button>
```

## 类组合与任意值

✅ 多类组合表达单个属性：

```html
<div class="blur-sm grayscale"><!-- filter: blur() + grayscale() --></div>
```

✅ 方括号语法覆盖设计系统：

```html
<button class="bg-[#316ff6]">Custom Color</button>
<div class="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">Complex Grid</div>
<div class="[--gutter-width:1rem] lg:[--gutter-width:2rem]">CSS Variables</div>
```

## 工作原理

Tailwind **不是**静态样式表，扫描项目中所有文件找到类名标记，按需生成 CSS。自动识别模板字符串中的类名：

```jsx
export default function Button({ size, children }) {
  let sizeClasses = {
    md: "px-4 py-2 rounded-md text-sm",
    lg: "px-8 py-3 rounded-lg text-base",
  }
  return <button class={`bg-blue-500 ${sizeClasses[size]}`}>{children}</button>
}
```
