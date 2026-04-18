# Airbnb JavaScript 规范

> **来源**: [airbnb/javascript](https://github.com/airbnb/javascript)
> **版权**: MIT License
> **Star**: 148k | **Fork**: 26.7k
> **特点**: 现代、最流行、社区广泛采用

---

## 核心要点速查

| 类别 | 核心规则 |
|------|----------|
| **引用** | 优先 `const`，用 `let`，禁用 `var` |
| **对象** | 字面量、简写、展开运算符、Object.hasOwn() |
| **数组** | 字面量、push、展开拷贝、Array.from() |
| **函数** | 函数表达式、rest 参数、默认参数放最后 |
| **箭头函数** | 匿名回调、参数括号、隐式返回 |
| **类** | class 语法、extends、返回 this |
| **模块** | ES6 import/export、default export |
| **命名** | camelCase 变量、PascalCase 类、UPPER_CASE 常量 |
| **比较** | 全等 ===、显式类型比较、?? 空值合并 |

---

## 1. 类型 (Types)

```javascript
// ✅ 原始类型 - 值拷贝
const foo = 1;
let bar = foo;
bar = 9;
console.log(foo, bar); // 1, 9

// ❌ 复杂类型 - 引用拷贝
const foo = [1, 2];
const bar = foo;
bar[0] = 9;
console.log(foo[0], bar[0]); // 9, 9
```

---

## 2. 引用 (References)

```javascript
// ✅ 正确
const a = 1;
const b = 2;
let count = 1;
if (true) {
  count += 1;
}

// ❌ 错误 - var 是函数作用域，不是块作用域
{
  let a = 1;
  const b = 1;
  var c = 1;
}
console.log(c); // 1 (泄露到外部)
```

---

## 3. 对象 (Objects)

```javascript
// ✅ 使用字面量
const item = {};

// ✅ 计算属性名
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};

// ✅ 方法简写
const atom = {
  value: 1,
  addValue(value) {
    return atom.value + value;
  },
};

// ✅ 使用 Object.hasOwn() (ES2022+)
console.log(Object.hasOwn(object, key));

// ✅ 使用展开运算符浅拷贝
const copy = { ...original, c: 3 };
```

---

## 4. 数组 (Arrays)

```javascript
// ✅ 使用字面量
const items = [];

// ✅ 使用 push 添加元素
someStack.push('abracadabra');

// ✅ 使用展开拷贝
const itemsCopy = [...items];

// ✅ Array.from() 用于类数组对象
const arrLike = { 0: 'foo', length: 3 };
const arr = Array.from(arrLike);

// ✅ 数组方法回调中必须有 return
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

---

## 5. 函数 (Functions)

```javascript
// ✅ 使用命名函数表达式
const short = function longUniqueMoreDescriptiveLexicalFoo() {};

// ✅ 立即执行函数用括号包裹
(function () {
  console.log('Welcome');
}());

// ✅ 使用 rest 参数
function concatenateAll(...args) {
  return args.join('');
}

// ✅ 默认参数放最后
function handleThings(name, opts = {}) {}

// ❌ 避免副作用的默认参数
// 不要这样做: function count(a = b++)
```

---

## 6. 箭头函数 (Arrow Functions)

```javascript
// ✅ 匿名回调使用箭头函数
[1, 2, 3].map((x) => x + 1);

// ✅ 单行表达式可隐式返回
[1, 2, 3].map((number) => `A string containing the ${number}`);

// ✅ 始终给参数加括号
[1, 2, 3].map((x) => x * x);

// ❌ 避免混淆箭头和比较运算符
const itemHeight = (item) => (item.height <= 256 ? item.largeSize : item.smallSize);
```

---

## 7. 类与构造函数

```javascript
// ✅ 使用 class
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}

// ✅ 使用 extends 继承
class PeekableQueue extends Queue {
  peek() {
    return this.queue[0];
  }
}

// ✅ 返回 this 支持链式调用
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }
}
```

---

## 8. 模块 (Modules)

```javascript
// ✅ 使用 import/export
import { es6 } from './AirbnbStyleGuide';
export default es6;

// ❌ 避免通配符导入
import * as AirbnbStyleGuide from './AirbnbStyleGuide';

// ✅ 单导出用 default export
export default function foo() {}

// ✅ 导入放在顶部
import foo from 'foo';
import bar from 'bar';
foo.init();

// ❌ 不要加文件扩展名
import foo from './foo'; // ✅
import foo from './foo.js'; // ❌
```

---

## 9. 迭代器

```javascript
// ✅ 优先使用高阶函数而非循环
const sum = numbers.reduce((total, num) => total + num, 0);
const increasedByOne = numbers.map((num) => num + 1);

// ❌ 避免使用 for-of 循环
for (let num of numbers) {
  sum += num;
}
```

---

## 10. 命名规范

| 类型 | 规则 | 示例 |
|------|------|------|
| 变量/函数 | camelCase | `thisIsMyObject` |
| 类/构造函数 | PascalCase | `class User` |
| 常量 | UPPER_CASE (仅限导出的不可变常量) | `export const API_KEY` |
| 首字母缩写 | 全大写或全小写 | `httpRequests` |
| ❌ 避免 | 下划线开头/结尾 | `_private` |

---

## 11. 其他重要规则

### 比较操作符
```javascript
// ✅ 始终使用 === 和 !==
if (isValid) {}

// ✅ 使用空值合并运算符 ??
const age = user.age ?? 18;
```

### 分号
```javascript
// ✅ 必须使用分号
const luke = {};
```

### 注释
```javascript
// ✅ 多行注释
/**
 * 多行注释内容
 */

// ❌ 行内注释放上方，而非右侧
const active = true; // 避免这种写法
```

---

## AI 编程调用示例

```markdown
请遵循 Airbnb JavaScript 规范：
- 优先使用 const
- 使用 === 而非 ==
- 对象使用字面量和展开运算符
- 数组使用 push 和展开拷贝
- 使用 ES6 import/export 模块
- 变量命名用 camelCase，类用 PascalCase
```
