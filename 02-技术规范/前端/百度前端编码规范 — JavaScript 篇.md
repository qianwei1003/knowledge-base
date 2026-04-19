---
来源: 百度EFE（百度前端工程部）
原始链接: https://github.com/ecomfe/spec/blob/master/javascript-style-guide.md
---

# 百度前端编码规范 — JavaScript 篇

## 核心要点

- **分级标注**：[强制]必须遵守，[建议]推荐遵守，团队可灵活执行
- **4空格缩进 + 单引号 + 分号**：三大基础风格铁律，不可违背
- **严格等于 `===`**：仅判断 null/undefined 时允许 `== null`
- **变量即用即声明**：禁止函数起始统一声明，每个var只声明一个变量
- **也适用于TypeScript**：规范明确覆盖预编译语言场景

## 代码风格

### 缩进与空格

- **[强制]** 4个空格缩进，禁止2空格或tab
- **[强制]** switch的case/default必须增加缩进层级
- **[强制]** 二元运算符两侧有空格，一元运算符无空格
- **[强制]** `{` 前必须有空格，函数名与`(`之间无空格
- **[强制]** 对象`:`后必须有空格，前不允许；`,`前无空格后必须跟空格

```javascript
// ✅
switch (variable) {
    case '1':
        break;
    default:
        break;
}
```

### 换行与语句

- **[强制]** 每行不超过120字符，运算符换行时放新行行首
- **[强制]** 不得省略分号；if/else等即使一行也不省略`{...}`
- **[强制]** IIFE必须在函数表达式外添加`(`
- **[强制]** 函数定义结束不加 分号

### 命名

| 类型 | 命名法 | 示例 |
|------|--------|------|
| 变量/函数 | Camel | `myName`、`getUserName` |
| 常量 | 全大写+下划线 | `MAX_COUNT` |
| 类 | Pascal | `PersonModel` |
| 枚举变量 | Pascal | `Direction` |
| 枚举属性 | 全大写+下划线 | `UP, DOWN` |

- **[强制]** 缩写词按命名法保持一致：`parseXML`(Camel)、`HTTPRequest`(Pascal)
- **[建议]** boolean用`is`/`has`开头；函数名用动宾短语

### 注释

- **[强制]** 单行注释独占一行，`//`后跟空格
- **[强制]** 文档注释用`/**...*/`，前空一行
- **[强制]** 文件顶部必须有`@file`；函数注释必须含说明、参数和返回值类型
- **[建议]** 注释说明what而非how

## 语言特性

### 变量与条件

- **[强制]** 变量使用前必须先定义，即用即声明
- **[强制]** 每个`var`只声明一个变量

```javascript
// ✅
const name = 'test';
const age = 18;

// ❌
const name = 'test', age = 18;
```

- **[强制]** 使用`===`，仅`== null`判断null/undefined
- **[建议]** 按执行频率排列分支；同变量多值用switch

### 类型转换

| 目标 | 方式 |
|------|------|
| string | `+ ''` |
| number | `+` / `Number()` |
| parseInt | **必须指定进制** `parseInt(str, 10)` |
| boolean | `!!` |

### 字符串与对象

- **[强制]** 字符串用**单引号** `'`
- **[强制]** 用字面量`{}`/`[]`创建对象和数组
- **[强制]** 禁止修改原生对象和宿主对象原型
- **[强制]** 遍历数组不用`for in`
- **[建议]** `for in`时用`hasOwnProperty`过滤

### 函数

- **[建议]** 函数≤50行，参数≤6个
- **[建议]** 非数据输入型参数用options对象传递

```javascript
// ✅
function createUser({ name, age, role }) { }

// ❌
function createUser(name, age, role, department, level, active) { }
```

### 面向对象与动态特性

- **[强制]** 类继承需修正`constructor`
- **[强制]** 自定义事件名全小写，只能有一个`event`参数
- **[强制]** 避免直接`eval`
- **[建议]** 减少`delete`和`with`使用，避免修改外部传入对象

## 浏览器环境

### DOM操作

- **[建议]** 单元素用`getElementById`；多元素用`getElementsByTagName`
- **[建议]** 获取子元素用`children`而非`childNodes`
- **[建议]** 样式变更用`className`，避免直接操作`style`
- **[强制]** 带单位的非0值不允许省略单位
- **[建议]** 优先`addEventListener`，适时移除监听器防内存泄漏
