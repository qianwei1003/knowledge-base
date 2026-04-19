---
来源: 官方文档
原始链接: https://react.dev/learn/thinking-in-react
---

# React 官方最佳实践：Thinking in React

## 核心要点

- **五步思考法**：拆分组件 → 静态版本 → 确定 state → state 归属 → 反向数据流
- **DRY 原则**：state 只保留最小完整表示，可计算的数据不是 state
- **单向数据流**：数据从父组件通过 props 向下流动，子组件通过回调更新父组件 state
- **静态先行**：先用 props 构建无交互静态版，再添加 state 和交互
- **组件拆分遵循三种思维**：编程思维（关注点分离）、CSS 思维（class 选择器）、设计思维（设计分层）

---

## Step 1: 将 UI 拆分为组件层次结构

从设计稿出发绘制组件树，JSON 数据结构自然映射到组件层次：

```
FilterableProductTable
├── SearchBar
└── ProductTable
    ├── ProductCategoryRow
    └── ProductRow
```

## Step 2: 构建静态版本

- 用 **props** 传递数据，**不要用 state**
- 可自顶向下或自底向上构建

> ✅ 静态版本需要大量打字，不需要思考；添加交互需要大量思考，不需要打字。

## Step 3: 找到 state 的最小但完整表示

| 数据 | 是 state? | 原因 |
|------|:---------:|------|
| 原始产品列表 | ❌ | props 传入 |
| 搜索文本 | ✅ | 随时间变化，无法计算 |
| 复选框值 | ✅ | 随时间变化，无法计算 |
| 过滤后列表 | ❌ | 可从原始数据计算 |

> ❌ 反模式：把可计算的派生数据存为 state

## Step 4: 确定 state 归属

1. 找出依赖该 state 的**所有组件**
2. 找到它们的**最近公共父组件**
3. state 放在公共父组件，或更高层，或新建容器组件

## Step 5: 添加反向数据流

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly}
      />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```

---

## Props vs State 速查

| | Props | State |
|---|-------|-------|
| 类比 | 函数参数 | 组件记忆 |
| 方向 | 父→子（单向） | 组件内部 |
| 可变性 | 只读 ✅ | setter 修改 |
| 来源 | 父组件传递 | 组件自身 |
