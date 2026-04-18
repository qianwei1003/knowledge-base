# 阿里 JavaScript 编码规约

> **来源**: [alibaba/f2e-spec](https://github.com/alibaba/f2e-spec)
> **配套工具**: eslint-config-ali
> **特点**: 强制性规则 + 推荐性规则，阿里内部实践

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **命名** | 小驼峰变量，大驼峰类，UPPER_CASE 常量 |
| **变量** | 优先 const，禁止 var |
| **分号** | 必须使用 |
| **缩进** | 2空格 |
| **相等** | 严格相等 === |
| **禁止** | eval、debugger、new Function |

---

## 一、命名规范

| 类型 | 规则 | 示例 |
|------|------|------|
| 文件命名 | 小写字母，- 连接 | `hello-world.js` |
| 变量/函数 | 小驼峰 camelCase | `thisIsMyFunction()` |
| 类/构造函数 | 大驼峰 PascalCase | `class User {}` |
| 常量 (export) | UPPERCASE_VARIABLES | `export const API_KEY` |
| 私有属性 | 下划线前缀 | `_privateMethod()` |
| ❌ 禁止 | 下划线开头/结尾 | `this._firstName` |

---

## 二、语句规范

### 变量声明

```javascript
// ✅ 使用 const/let，不用 var
const foo = 'foo';
let bar;

// ✅ 优先使用 const
const arr = [];
arr.push('item'); // ✅ 修改引用属性不是重新赋值

// ✅ 一条语句声明一个变量
const foo = 1;
const bar = 2;

// ✅ 声明的变量必须被使用
```

### 分号与逗号

```javascript
// ✅ 必须使用分号
const foo = 'foo';

// ✅ 行首不用逗号，末尾加逗号
const hero = {
  firstName: 'Ada',
  lastName: 'Lovelace',  // 末尾逗号
};
```

### 缩进与空格

```javascript
// ✅ 2个空格缩进
// ✅ 左大括号前有空格
function test() { }

// ✅ 控制语句小括号前后有空格
if (isJedi) { }

// ✅ 运算符两侧有空格
const x = y + 5;
```

### 块与括号

```javascript
// ✅ 始终使用大括号包裹代码块
if (foo) {
  bar();
  baz();
}

// ✅ Egyptian Brackets 风格
if (foo) {
  thing1();
} else {
  thing2();
}
```

### 运算符

```javascript
// ✅ 使用严格相等 ===
if (Number(id) === 83949) { }

// ✅ 禁止一元自增/自减 (for循环除外)
num += 1;  // ✅
num++;     // ❌

// ✅ 避免嵌套三元表达式
const qu = qux === quxx ? bing : bam;
```

### 函数

```javascript
// ✅ 箭头函数替代匿名函数
[1, 2, 3].map((x) => x * x);

// ✅ 单参数可省略小括号，使用 return 简写
[1, 2, 3].map(x => x * x);

// ✅ 使用默认参数
function multiply(a = 0, b = 0) { }

// ✅ 默认参数放最后
function multiply(a, b = 1) { }
```

### 数组与对象

```javascript
// ✅ 使用字面量
const arr = [1, 2, 3];
const obj = {};

// ✅ 数组方法回调必须包含 return
arr.map((item, index) => {
  memo[item] = index;
  return memo;
});

// ✅ 使用扩展运算符
const copy = [...original];
const merged = { ...obj1, ...obj2 };

// ✅ 使用解构
const { firstName, lastName } = user;
```

### 控制语句

```javascript
// ✅ switch case 必须以 break 结尾
switch(foo) {
  case 1: doSomething(); break;
  default: doDefault(); break;
}

// ✅ switch 应包含 default 分支
// ✅ 嵌套层级不超过 4 层
```

---

## 三、注释规范

| 规则 | 说明 |
|------|------|
| 单行注释 | 使用 `//`，写在被注释对象**上方** |
| 多行注释 | 使用 `/** ... */` |
| 空格 | 注释内容与 `//` 之间有空格 |
| 特殊标记 | `// FIXME:` 说明问题，`// TODO:` 说明待办 |
| 文档注释 | 推荐使用 JSDoc 规范 |
| 无用注释 | 无用代码注释应即时删除 |

```javascript
// ✅ 正确示例
// is current tab
const active = true;

// ❌ 行内注释放上方
const active = true;  // is current tab
```

---

## 四、模块规范

```javascript
// ✅ 使用 ES6 modules (import/export)
import React, { Component } from 'react';
export default Component;

// ✅ 多个 import 合成一条
import React, { Component } from 'react';

// ✅ import 放在模块最上方
import foo from 'foo';
import bar from 'bar';
foo.init();

// ❌ 禁止循环引用

// ✅ 单个 export 时使用 default export
```

---

## 五、禁止项清单

| 规则 | 说明 |
|------|------|
| ❌ 禁止 `new Number/String/Boolean` | 使用原始类型 |
| ❌ 禁止 `eval` | 安全风险 |
| ❌ 禁止 `debugger` | 不应发布到线上 |
| ❌ 禁止 `new Function` | 同 eval |
| ❌ 禁止 `void` | 使用 `undefined` |
| ❌ 禁止连续赋值 | `let a = b = c = 1` 会产生全局变量 |
| ❌ 禁止空代码块 | 需写明注释或删除 |
| ❌ 禁止重复声明 | 变量/函数不可重名 |
| ❌ 禁止块中函数声明 | 使用函数表达式 |

---

## 六、代码行数限制

| 限制项 | 最大值 |
|--------|--------|
| 单行最大字符数 | 100 |
| 文件最大行数 | 1000 |
| 函数最大行数 | 80 |
| 函数参数数量 | 建议不超过 3 个 |
| 函数圈复杂度 | ≤ 10 |
| 认知复杂度 | ≤ 15 |

---

## AI 编程调用示例

```markdown
请遵循阿里 JavaScript 编码规约：
- 优先使用 const，禁止 var
- 使用严格相等 ===
- 2空格缩进，左大括号前有空格
- 禁止 eval、debugger、new Function
- 注释写在被注释对象上方
- 函数不超过80行，单行不超过100字符
- switch 必须有 default 和 break
```
