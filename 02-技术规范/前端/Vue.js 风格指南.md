---
来源: Vue.js 官方文档
原始链接: https://vuejs.org/style-guide/
---

# Vue.js 官方风格指南

## 核心要点

- 组件名必须多词 PascalCase，根 App 组件除外
- v-for 必须绑定 key；禁止与 v-if 在同一元素上同用
- Prop 定义必须指定类型，推荐包含 required 和 validator
- 所有组件（除顶层 App/布局）必须使用作用域样式

## 优先级总览

| 优先级 | 说明 | 违反后果 |
|--------|------|---------|
| A: Essential | 防止错误 | 可能导致 bug |
| B: Strongly Recommended | 提高可读性 | 代码仍可运行 |
| C: Recommended | 确保一致性 | 自由选择 |
| D: Use with Caution | 谨慎使用 | 增加维护难度 |

## Priority A: Essential（必要）

### 1. 多词组件名

防止与现有/未来 HTML 元素冲突。

```javascript
// ❌ Bad
<Item />
<item></item>

// ✅ Good
<TodoItem />
<todo-item></todo-item>
```

### 2. 详细 Prop 定义

至少指定类型，推荐完整声明。

```javascript
// ❌ Bad（仅原型阶段可用）
props: ['status']

// ✅ Good
props: {
  status: {
    type: String,
    required: true,
    validator: value => ['syncing', 'synced', 'version-conflict', 'error'].includes(value)
  }
}
```

### 3. v-for 设置 key

维护子树内部组件状态，保证可预测行为。

```html
<!-- ❌ Bad -->
<li v-for="todo in todos">{{ todo.text }}</li>

<!-- ✅ Good -->
<li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
```

### 4. 禁止 v-if 与 v-for 同用

```html
<!-- ❌ Bad -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">

<!-- ✅ Good - 用 computed 过滤 -->
<li v-for="user in activeUsers" :key="user.id">

<!-- ✅ Good - 用 template 包裹 -->
<template v-for="user in users" :key="user.id">
  <li v-if="user.isActive">{{ user.name }}</li>
</template>
```

### 5. 组件作用域样式

顶层 App/布局组件可用全局样式，其余必须作用域化。

```html
<!-- ❌ Bad -->
<style>
.btn-close { background-color: red; }
</style>

<!-- ✅ Good - scoped -->
<style scoped>
.button-close { background-color: red; }
</style>

<!-- ✅ Good - BEM -->
<style>
.c-Button--close { background-color: red; }
</style>
```

## Priority B: Strongly Recommended（强烈推荐）

- **文件名**：统一 PascalCase 或 kebab-case
- **组件名**：多词 PascalCase
- **自闭合组件**：单文件组件/字符串模板/JSX 中使用 `<Component />`
- **Prop 名**：声明用 camelCase，模板用 kebab-case
- **多 attribute**：分行书写
- **模板表达式**：只写简单表达式，复杂逻辑提取为 computed
- **computed 拆分**：复杂 computed 拆为多个简单的
- **attribute 引号**：非空值始终带引号
- **指令缩写**：`:` / `@` 统一使用或统一不使用
- **SFC 标签顺序**：`<script>` → `<template>` → `<style>`

## Priority C: Recommended（推荐）

- 统一组件/实例选项顺序
- 统一元素 attribute 顺序
- 多属性组件选项间加空行
- SFC 顶级 attribute 顺序：`lang` → `scoped`

## Priority D: Use with Caution（谨慎使用）

- 无 `v-bind` 的 scoped slots → 应使用明确 props
- 嵌套 v-for → 提取为独立组件
- 索引作为 key → 仅限静态列表
- 同一组件的 v-if/v-else → 优先使用 `:key`
