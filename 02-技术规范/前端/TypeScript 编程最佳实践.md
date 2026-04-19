---
来源: TypeScript官方文档
领域: 前端
关键词: TypeScript, 类型系统, 最佳实践, 类型收窄, 泛型, 函数重载, 联合类型, 代码规范
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://www.typescriptlang.org/docs/handbook
---

# TypeScript 编程最佳实践

> 基于 TypeScript 官方 Handbook 整理，涵盖类型选择、函数签名、类型收窄、泛型等核心编程规范。
> 来源：https://www.typescriptlang.org/docs/handbook

---

## 一、基本类型选择

### 1.1 使用原始类型，不要使用包装类型

```typescript
// ❌ 错误 — 永远不要使用 Number, String, Boolean, Symbol, Object
function reverse(s: String): String;

// ✅ 正确 — 使用小写的原始类型
function reverse(s: string): string;
```

**原因**：`String`、`Number` 等大写类型指的是 JavaScript 的包装对象，几乎从不应该被使用。需要非原始 object 时，使用小写的 `object` 类型。

### 1.2 避免使用 any

```typescript
// ❌ 错误 — any 关闭了类型检查
function process(data: any) {
  return data.foo(); // 编译通过，但可能运行时报错
}

// ✅ 正确 — 使用 unknown 表示未知类型
function process(data: unknown) {
  // 使用前必须进行类型检查
  if (typeof data === 'object' && data !== null) {
    return (data as Record<string, unknown>).foo;
  }
}
```

**原则**：只有在从 JavaScript 迁移到 TypeScript 的过渡期才使用 `any`。其他场景使用 `unknown`。

### 1.3 泛型必须使用类型参数

```typescript
// ❌ 错误 — 泛型类型参数未被使用
interface Empty<T> {}

// ✅ 正确 — 类型参数被实际使用
interface Box<T> {
  value: T;
}
```

---

## 二、函数类型规范

### 2.1 回调返回值使用 void 而非 any

```typescript
// ❌ 错误 — 回调返回值将被忽略时不应使用 any
function fn(x: () => any) {
  x();
}

// ✅ 正确 — 使用 void
function fn(x: () => void) {
  x();
}
```

**原因**：`void` 更安全，防止意外使用返回值：

```typescript
function fn(x: () => void) {
  var k = x();           // k 的类型为 void，而非 any
  k.doSomething();       // 编译错误！如果用 any 则不会报错
}
```

### 2.2 回调参数不要设为可选

```typescript
// ❌ 错误 — done 可能被调用 1 次或 2 次，语义不清
interface Fetcher {
  getObject(done: (data: unknown, elapsedTime?: number) => void): void;
}

// ✅ 正确 — 调用方可以忽略不需要的参数
interface Fetcher {
  getObject(done: (data: unknown, elapsedTime: number) => void): void;
}
```

**原因**：JavaScript 中回调总是可以接收比声明更少的参数。将参数设为可选暗示"可能不传"，但实际调用者只是"不关心"，语义完全不同。

### 2.3 回调不应按参数数量写重载

```typescript
// ❌ 错误 — 按回调参数数量分别重载
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number
): void;

// ✅ 正确 — 使用最大参数数量的单一重载
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number
): void;
```

---

## 三、函数重载规范

### 3.1 重载排序：从特殊到一般

```typescript
// ❌ 错误 — 更一般的重载放在前面，特殊重载被遮蔽
declare function fn(x: unknown): unknown;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;
var myElem: HTMLDivElement;
var x = fn(myElem); // x: unknown，不符合预期！

// ✅ 正确 — 按从特殊到一般的顺序排列
declare function fn(x: HTMLDivElement): string;
declare function fn(x: HTMLElement): number;
declare function fn(x: unknown): unknown;
var myElem: HTMLDivElement;
var x = fn(myElem); // x: string，符合预期
```

**原因**：TypeScript 使用第一个匹配的重载。如果一般性重载在前，特殊重载永远不会被匹配到。

### 3.2 尾部参数使用可选参数代替重载

```typescript
// ❌ 错误 — 仅尾部参数不同时不应写多个重载
interface Example {
  diff(one: string): number;
  diff(one: string, two: string): number;
  diff(one: string, two: string, three: boolean): number;
}

// ✅ 正确 — 使用可选参数
interface Example {
  diff(one: string, two?: string, three?: boolean): number;
}
```

**注意**：仅当所有重载的返回类型相同时才应合并。

### 3.3 单参数类型差异使用联合类型

```typescript
// ❌ 错误 — 仅一个参数类型不同
interface Moment {
  utcOffset(): number;
  utcOffset(b: number): Moment;
  utcOffset(b: string): Moment;
}

// ✅ 正确 — 使用联合类型
interface Moment {
  utcOffset(): number;
  utcOffset(b: number | string): Moment;
}
```

**原因**：联合类型使"传递"值到函数时类型检查更正确。

---

## 四、类型收窄（Narrowing）

### 4.1 typeof 类型守卫

```typescript
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;  // 此处 padding 收窄为 number
  }
  return padding + input;               // 此处 padding 收窄为 string
}
```

### 4.2 注意 typeof null === "object"

```typescript
// ⚠️ typeof null 返回 "object"，这是 JS 历史遗留问题
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    // strs 的类型是 string[] | null，不是 string[]！
    for (const s of strs) {  // 编译错误：strs 可能是 null
      console.log(s);
    }
  }
}
```

### 4.3 真值性检查的陷阱

```typescript
// ⚠️ 空字符串是 falsy，但可能是合法输入
function printAll(strs: string | string[] | null) {
  if (strs) {
    // strs 在此处排除了 null 和 undefined
    // 但也排除了空字符串 "" ！
  }
}

// ✅ 更精确的检查方式
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) { console.log(s); }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

### 4.4 == null 同时排除 null 和 undefined

```typescript
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  if (container.value != null) {
    // != 同时排除了 null 和 undefined
    console.log(container.value);  // 类型为 number
    container.value *= factor;     // 安全操作
  }
}
```

### 4.5 in 操作符收窄

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();  // animal 收窄为 Fish
  } else {
    animal.fly();   // animal 收窄为 Bird
  }
}
```

---

## 五、日常类型使用

### 5.1 数组类型

```typescript
// 两种等价写法
let nums: number[] = [1, 2, 3];
let strs: Array<string> = ["a", "b", "c"];
```

### 5.2 可选属性

```typescript
function printName(obj: { first: string; last?: string }) {
  console.log(obj.first);
  // 可选属性可能是 undefined，必须检查
  console.log(obj.last?.toUpperCase());
}
```

### 5.3 联合类型

```typescript
// 联合类型：值可以是多种类型之一
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
```

### 5.4 类型推断优于显式标注

```typescript
// 通常不需要显式标注 — TypeScript 会自动推断
let myName = "Alice";       // 推断为 string
const names = ["Alice"];    // 推断为 string[]

// 函数返回类型通常也能推断
function getFavoriteNumber() {
  return 26;  // 推断返回类型为 number
}
```

### 5.5 上下文类型

```typescript
// 回调参数类型可从上下文自动推断
const names = ["Alice", "Bob"];
names.forEach((s) => {
  console.log(s.toUpperCase());  // s 被推断为 string，无需标注
});
```

---

## 六、类型断言 vs 类型声明

### 6.1 尽量使用类型声明而非断言

```typescript
// ❌ 类型断言 — 告诉编译器"我相信你知道的比我多"
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

// ✅ 类型声明 — 让编译器为你验证
const myCanvas = document.querySelector<HTMLCanvasElement>("#main_canvas");
```

### 6.2 非空断言（!）

```typescript
// ⚠️ 使用非空断言要格外小心
const el = document.getElementById("foo")!;  // 你确定元素存在吗？
```

---

## 七、配置最佳实践

### 7.1 tsconfig.json 推荐

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "target": "ES2022",
    "moduleResolution": "bundler"
  }
}
```

### 7.2 关键编译选项说明

| 选项 | 说明 | 推荐值 |
|------|------|--------|
| `strict` | 启用所有严格检查 | `true` |
| `noImplicitAny` | 禁止隐式 any | `true` |
| `strictNullChecks` | 严格的 null/undefined 检查 | `true` |
| `noUncheckedIndexedAccess` | 数组/对象索引访问返回 `T \| undefined` | `true` |
| `exactOptionalPropertyTypes` | 区分"属性缺失"和"属性值为 undefined" | `true` |

---

## 八、总结检查清单

### 类型定义
- [ ] 使用小写原始类型（`string`/`number`/`boolean`），不用包装类型
- [ ] 避免使用 `any`，用 `unknown` 代替
- [ ] 泛型参数必须被实际使用

### 函数
- [ ] 回调返回值类型为 `void` 而非 `any`
- [ ] 回调参数不设为可选
- [ ] 优先使用可选参数和联合类型，减少重载数量

### 重载
- [ ] 按从特殊到一般的顺序排列
- [ ] 尾部参数差异用可选参数
- [ ] 单参数类型差异用联合类型

### 类型安全
- [ ] 启用 `strict` 模式
- [ ] 使用类型声明优于类型断言
- [ ] 善用类型收窄而非断言

> 📚 完整文档：https://www.typescriptlang.org/docs/handbook
> 🎯 类型体操练习：https://github.com/type-challenges/type-challenges
