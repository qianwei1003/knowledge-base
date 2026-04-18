# 阿里 TypeScript 编码规约

> **来源**: [alibaba/f2e-spec](https://github.com/alibaba/f2e-spec)
> **配套工具**: eslint-config-ali
> **特点**: 与 JavaScript 规约一脉相承，强制规则 + 推荐规则

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **类型断言** | 必须用 `as Type`，禁止 `<Type>` |
| **接口 vs 类型** | 优先使用 `interface` |
| **成员分隔符** | 必须使用分号 `;` |
| **void** | 禁止与除 `never` 外做联合/交叉 |
| **私有** | 使用 `private` 修饰符 |
| **字符串** | 必须使用单引号 |

---

## 一、类型规范

### 1. 类型断言

```typescript
// ✅ 必须使用 as Type 风格
const foo = 'bar' as string;
const x: T = { ... };

// ❌ 禁止使用 <Type> 语法（易与 JSX 混淆）
const foo = <string>'bar';

// ❌ 对象字面量禁止类型断言（断言成 any 除外）
const x = { ... } as T;  // ❌
```

### 2. void 类型

```typescript
// ❌ 禁止在返回类型或泛型类型参数之外使用 void
type PossibleValues = string | number | void;  // ❌
function logSomething(thing: void) {}         // ❌

// ✅ 正确用法
type NoOp = () => void;
function noop(): void {}
```

### 3. 类型推断

```typescript
// ✅ 简单类型可省略注解
const foo = 1;
const bar = '';
const isValid = true;

// ❌ 避免冗余的类型注解
const foo: number = 1;    // ❌
const bar: string = '';   // ❌
```

### 4. 可选链与非空断言

```typescript
// ❌ 禁止在可选链之后使用非空断言
foo?.bar!;  // ❌

// ✅ 正确用法
foo?.bar;    // ✅
```

---

## 二、接口与类型别名

### 1. 优先使用 interface

```typescript
// ❌ 避免用 type 别名定义对象
type T = { x: number };

// ✅ 优先使用 interface
interface T { x: number; }
```

### 2. 成员分隔符

```typescript
// ❌ 必须使用分号作为成员分隔符
interface Foo {
  name: string,    // ❌
  greet(): void,   // ❌
}

// ✅ 正确用法
interface Foo {
  name: string;
  greet(): void;
}
```

### 3. 方法定义风格

```typescript
// ❌ 避免
interface T1 { func(arg: string): number; }

// ✅ 推荐：使用属性方式定义
interface T1 { func: (arg: string) => number; }
```

### 4. 禁止空接口

```typescript
// ❌ 禁止定义空接口
interface Foo {}

// ✅ 正确用法
interface Foo { name: string; }
```

### 5. 禁止 namespace/module

```typescript
// ❌ 禁止
module foo {}
namespace foo {}

// ✅ 使用 ES2015 模块语法
declare module 'foo' {}
declare namespace foo {}  // 仅用于声明文件
```

---

## 三、泛型规范

### 1. 函数重载 vs 联合类型

```typescript
// ❌ 避免过度重载
function f(x: number): void;
function f(x: string): void;

// ✅ 优先使用参数的联合类型
function f(x: number | string): void;
```

### 2. 泛型中的 void

```typescript
// ❌ 禁止在泛型类型参数中使用 void
```

---

## 四、命名规范

| 类别 | 规则 | 示例 |
|------|------|------|
| 字符串 | **必须**单引号 | `'hello'` |
| 类成员 | 设置可见性修饰符 | `public`/`protected`/`private` |
| 静态/实例 | 静态优先于实例 | 先 static 后 instance |

---

## 五、类与成员顺序

**推荐顺序**：

```typescript
class Foo {
  // 1. 静态成员
  public static foo1 = 'foo1';
  protected static foo2 = 'foo2';
  private static foo3 = 'foo3';

  // 2. 实例属性
  public bar1 = 'bar1';
  protected bar2 = 'bar2';
  private bar3 = 'bar3';

  // 3. 构造函数
  public constructor() {}

  // 4. 方法
  public getBar1() {}
  protected getBar2() {}
  private getBar3() {}
}
```

**原则**：
- 静态成员 > 实例成员
- 属性 > 构造函数 > 方法
- 公开 > 保护 > 私有

---

## 六、其他重要规则

| 规则 | 级别 | 说明 |
|------|------|------|
| 数组类型 | 推荐 | 简单类型用 `T[]`，复杂类型用 `Array<T>` |
| 导入语法 | 推荐 | 使用 ES2015 `import` 代替 `require` |
| this 赋值 | 推荐 | 避免将 this 赋值给变量，使用箭头函数 |
| as const | 推荐 | 值与类型相等时优先使用 |
| 类型注解空格 | **强制** | 冒号前无空格，冒号后一空格 |

```typescript
// ✅ 类型注解格式
let foo: string = 'bar';
function foo(): string {}
type Foo = () => void;
```

---

## 七、ESLint 规则对应

| 规则 | 说明 |
|------|------|
| `@typescript-eslint/consistent-type-assertions` | 类型断言风格 |
| `@typescript-eslint/consistent-type-definitions` | interface 优先 |
| `@typescript-eslint/no-explicit-any` | 禁止 any |
| `@typescript-eslint/no-non-null-assertion` | 禁止 `!` |

---

## AI 编程调用示例

```markdown
请遵循阿里 TypeScript 编码规约：
- 必须使用 as Type 类型断言，禁止 <Type>
- 优先使用 interface，成员分隔符用分号
- 禁止空接口，禁止 namespace/module
- 字符串必须使用单引号
- 类成员顺序：静态 > 实例，公开 > 保护 > 私有
- 类型注解冒号后有空格
- 避免将 this 赋值给变量
```
