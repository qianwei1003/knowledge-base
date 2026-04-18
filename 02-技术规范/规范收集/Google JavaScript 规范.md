# Google JavaScript 编码规范

> **来源**: [Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-javascript-styleguide/)
> **特点**: 严谨、全面、适合大型项目

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **命名** | 函数/变量 camelCase，类 PascalCase，常量 UPPER_CASE |
| **引号** | 单引号 `'`，而非双引号 `"` |
| **缩进** | 2空格 |
| **私有** | 下划线 `_` 开头 |
| **注释** | JSDoc 规范 |
| **文件** | 小写、`.js` 结尾、推荐 `-` 分隔 |

---

## 一、命名规范

### 1. 基本命名风格

| 类型 | 风格 | 示例 |
|------|------|------|
| 函数/变量 | functionNamesLikeThis | `myFunction()` |
| 类名 | ClassNamesLikeThis | `MyClass` |
| 枚举 | EnumNamesLikeThis | `ColorEnum` |
| 常量 | CONSTANT_VALUES_LIKE_THIS | `MAX_COUNT` |
| 文件名 | filenameslikethis.js | `myfile.js` |

### 2. 私有/受保护成员

- **私有属性和方法**：以下划线 `_` 开头命名
- **受保护属性和方法**：无下划线开头（与公共成员相同）

```javascript
/** @private */
this._privateMethod = function() {};

/** @protected */
this.protectedMethod = function() {};
```

### 3. 函数参数

- **可选参数**：`opt_` 开头，如 `opt_name`
- **可变参数**：以 `var_args` 命名最后一个参数

```javascript
/**
 * @param {string} opt_name 可选参数
 * @param {...*} var_args 可变参数
 */
function foo(opt_name, var_args) {}
```

### 4. 存取函数（Getter/Setter）

```javascript
// 读取方法
this.getFoo();

// 写入方法
this.setFoo(value);

// 布尔型读取（可选）
this.isFoo();
```

---

## 二、代码格式

### 1. 大括号

```javascript
// ✅ 正确：同一行开始大括号
if (something) {
  // ...
} else {
  // ...
}
```

### 2. 字符串

- ✅ 使用**单引号** `'`，而非双引号 `"`

```javascript
var msg = 'This is some HTML';
```

### 3. 数组和对象初始化

```javascript
// ✅ 单行允许
var arr = [1, 2, 3];
var obj = {a: 1, b: 2, c: 3};

// ✅ 多行缩进两个空格
var inset = {
  top: 10,
  right: 20,
  bottom: 15,
  left: 12
};
```

### 4. 函数参数换行

```javascript
// ✅ 推荐格式，每行不超过80字符
goog.foo.bar.doThingThatIsVeryDifficultToExplain = function(
    veryDescriptiveArgumentNumberOne,
    veryDescriptiveArgumentNumberTwo,
    tableModelEventHandlerProxy,
    artichokeDescriptorAdapterIterator) {
  // ...
};
```

### 5. 二元/三元操作符

```javascript
var x = a ? b : c;  // 单行OK

var y = a ?
    longButSimpleOperandB :
    longButSimpleOperandC;
```

---

## 三、注释规范

### JSDoc 基本语法

```javascript
/**
 * A JSDoc comment should begin with a slash and 2 asterisks.
 * @desc Block tags should always start on their own line.
 */
```

### 主要 JSDoc 标签

| 标签 | 用途 |
|------|------|
| `@constructor` | 标识构造函数 |
| `@param {Type} name` | 参数说明 |
| `@return {Type}` | 返回值说明 |
| `@type {Type}` | 变量类型 |
| `@private` | 私有成员 |
| `@protected` | 受保护成员 |
| `@deprecated` | 已废弃 |
| `@override` | 重写父类方法 |
| `@interface` | 定义接口 |
| `@enum {Type}` | 枚举类型 |

### 类型语法

```javascript
/** @type {number} */
var count = 0;

// 常用类型
// {number} {string} {boolean}
// {?number} 可为空
// {!Object} 非空
// {number|boolean} 联合类型
// {Array.<string>} 数组
// {function(string, boolean): number} 函数
```

---

## 四、可见性（私有/受保护）

```javascript
/**
 * @private
 * @constructor
 */
AA_PrivateClass_ = function() {};

/** @private */
function AA_init_() {
  return new AA_PrivateClass_();
}
```

---

## 五、类型系统

| 类型 | 语法 |
|------|------|
| 原始类型 | `{number}`, `{string}`, `{boolean}` |
| 可为空 | `{?number}` |
| 非空 | `{!Object}` |
| 联合类型 | `{number\|boolean}` |
| 数组 | `{Array.<string>}` |
| 对象 | `{Object.<string, number>}` |
| 函数 | `{function(string, boolean): number}` |

```javascript
/**
 * @param {!Object} nonNull      // 必不为null
 * @param {Object} mayBeNull     // 可为null
 * @param {!Object=} opt_nonNull // 可选但必不为null
 * @param {Object=} opt_mayBeNull // 可选可为null
 */
```

---

## 六、实用技巧

### 布尔表达式

- 返回 `false`：`null`, `undefined`, `''`, `0`
- 返回 `true`：`'0'`, `[]`, `{}`

### 使用 || 简化

```javascript
// ✅ 简化前
if (opt_win) {
  win = opt_win;
} else {
  win = window;
}

// ✅ 简化后
var win = opt_win || window;
```

### 遍历节点列表

```javascript
// ✅ 高效遍历
for (var i = 0, paragraph; paragraph = paragraphs[i]; i++) {
  doSomething(paragraph);
}
```

---

## AI 编程调用示例

```markdown
请遵循 Google JavaScript 编码规范：
- 变量/函数用 camelCase，类用 PascalCase
- 字符串使用单引号
- 私有成员以下划线 _ 开头
- 使用 JSDoc 注释规范
- 文件名小写，推荐用 - 分隔
- 每行不超过 80 字符
```
