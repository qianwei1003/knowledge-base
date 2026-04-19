---
来源: 官方文档 / GitHub
领域: 前端
关键词: JavaScript, Airbnb, 代码规范, ESLint, 最佳实践
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://github.com/airbnb/javascript
---

# Airbnb JavaScript 代码规范（目录概览）

> 来源：Airbnb 官方 GitHub https://github.com/airbnb/javascript
>
> 世界上最流行的 JavaScript 代码规范之一

## 安装

```bash
npx install-peerdeps --dev eslint-config-airbnb
```

- ESLint 配置包：`eslint-config-airbnb`
- 基础版（不含 React）：`eslint-config-airbnb-base`

## 规范目录

| 章节 | 内容 |
|------|------|
| **Types** | 原始类型、引用类型、类型转换 |
| **References** | 引用赋值，避免使用 `new` |
| **Objects** | 对象创建、属性访问、展开运算符 |
| **Arrays** | 数组字面量、解构、展开、回调 |
| **Destructuring** | 对象/数组解构赋值 |
| **Strings** | 字符串字面量、模板字符串 |
| **Functions** | 函数声明、箭头函数、默认参数 |
| **Arrow Functions** | 箭头函数使用场景 |
| **Classes & Constructors** | 类定义、继承、方法 |
| **Modules** | import/export 规范 |
| **Iterators and Generators** | 迭代器和生成器 |
| **Properties** | 属性访问（点 vs 方括号） |
| **Variables** | 变量声明、const 优先、解构 |
| **Hoisting** | 变量提升相关 |
| **Comparison Operators & Equality** | 等值比较，严格等于 |
| **Blocks** | 代码块使用 |
| **Control Statements** | 控制语句规范 |
| **Comments** | 注释风格（JSDoc、行注释） |
| **Whitespace** | 缩进、空格、空行 |
| **Commas** | 逗号使用（前导 vs 尾随） |
| **Semicolons** | 分号使用 |
| **Type Casting & Coercion** | 类型转换 |
| **Naming Conventions** | 命名约定（camelCase、PascalCase） |
| **Accessors** | getter/setter |
| **Events** | 事件处理 |
| **ECMAScript 6+** | ES6+ 特性使用 |
| **Standard Library** | 标准库使用 |
| **Testing** | 测试规范 |
| **Performance** | 性能相关 |

## 关键原则

1. **const 优先**：所有引用使用 `const`，避免 `var`
2. **对象字面量**：使用字面量语法创建对象和数组
3. **对象展开**：浅拷贝优先使用对象展开 `...`
4. **模板字符串**：字符串拼接使用模板字符串
5. **箭头函数**：回调函数使用箭头函数
6. **解构**：函数参数超过 2 个时使用对象解构
7. **严格等于**：始终使用 `===` 代替 `==`
8. **分号**：始终使用分号

## 扩展规范

Airbnb 还提供了以下扩展规范：
- [React](https://github.com/airbnb/javascript/tree/master/react) - React 编码规范
- [CSS-in-JavaScript](https://github.com/airbnb/javascript/tree/master/css-in-javascript) - CSS-in-JS 规范
- [CSS & Sass](https://github.com/airbnb/css) - CSS/Sass 规范
- [Ruby](https://github.com/airbnb/ruby) - Ruby 规范

## 多语言支持

该规范已被社区翻译为多种语言：中文、日文、韩文、泰文、葡萄牙文、西班牙文等。

---

> ⚠️ **注意**：此规范假设使用 Babel，并要求安装 `babel-preset-airbnb` 或等效配置。
