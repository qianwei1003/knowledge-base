---
来源: React 官方文档 (react.dev)
原始链接: https://react.dev/learn/react-compiler
---

# React Compiler 官方指南

## 核心要点

- **自动记忆化**：编译器在构建时自动优化组件渲染，无需手动 useMemo/useCallback/React.memo
- **级联重渲染跳过**：状态变化时仅重渲染相关组件，实现细粒度响应性
- **零侵入**：适用于普通 JavaScript，无需重写代码；已在 Meta 生产环境验证
- **构建工具广泛支持**：Babel / Vite / Metro / Rsbuild 官方支持，SWC 与 Next.js 合作中
- **渐进式采用**：可先在部分模块启用，逐步扩展

## 一、Compiler 解决什么问题

React 组件状态变化时会级联重渲染所有子组件。传统做法需手动记忆化来跳过不必要的渲染，但手动记忆化繁琐且容易出错。

❌ **手动记忆化的常见 Bug**：即使 handleClick 被 useCallback 包裹，箭头函数 () => handleClick(item) 每次渲染仍创建新函数，破坏 Item 的记忆化。

`jsx
// ❌ 手动记忆化（繁琐且易出错）
const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {
  const processedData = useMemo(() => expensiveProcessing(data), [data]);
  const handleClick = useCallback((item) => { onClick(item.id); }, [onClick]);
  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
});

// ✅ 使用 Compiler 后（自动优化，代码简洁）
function ExpensiveComponent({ data, onClick }) {
  const processedData = expensiveProcessing(data);
  const handleClick = (item) => { onClick(item.id); };
  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
}
`

## 二、Compiler 优化类型

### 1. 跳过级联重渲染

状态变化时仅重渲染相关组件：

`jsx
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />  {/* Compiler 确保此组件不因 friends 变化而重渲染 */}
    </div>
  );
}
`

### 2. 记忆化昂贵计算

Compiler 会自动记忆化组件内的昂贵计算调用，但不会记忆化独立函数（非组件/Hook 内）。

> ⚠️ 若昂贵函数在多个组件中使用，每个组件会独立运行。建议先性能分析确认瓶颈。

## 三、手动记忆化建议

| 场景 | 建议 |
|------|------|
| **新代码** | 依赖编译器，仅在需要精确控制 effect 依赖时使用 useMemo/useCallback |
| **现有代码** | 保留现有记忆化代码，移除可能改变编译输出，需仔细测试后再操作 |

## 四、构建工具支持

| 工具 | 状态 |
|------|------|
| Babel / Vite / Metro / Rsbuild | ✅ 官方支持 |
| SWC | 🔄 与 Next.js 合作开发中 |
| Next.js | ✅ v15.3.1+ 可通过 SWC 启用 |

## 五、快速上手

1. 根据构建工具安装 React Compiler
2. 在构建配置中启用编译器
3. 运行测试套件确保无问题
4. 不确定时先在部分模块启用，逐步扩展
5. 使用 React Compiler 调试工具监控优化效果