# Google TypeScript 风格指南

> **来源**: [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
> **特点**: 严谨、全面、Google内部实践

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **类/接口/类型/枚举** | UpperCamelCase |
| **变量/函数/方法** | lowerCamelCase |
| **全局常量/枚举值** | CONSTANT_CASE |
| **类型** | 优先 interface，避免 any，用 unknown |
| **私有** | 用 `private`，不用 `#private` |
| **导出** | 优先命名导出，不用 default export |

---

## 一、标识符命名

### 命名风格对照表

| 类型 | 风格 | 示例 |
|------|------|------|
| 类/接口/类型/枚举/装饰器 | UpperCamelCase | `class MyClass` |
| 变量/参数/函数/方法/属性 | lowerCamelCase | `getName()` |
| 全局常量/枚举值 | CONSTANT_CASE | `MAX_COUNT` |

### 关键规则

```typescript
// ✅ 推荐：清晰、有意义的名称
const errorCount: number;
const customerId: string;

// ❌ 避免：模糊的缩写
const n;           // 无意义
const nErr;        // 含义不清

// ✅ 缩写按整词处理
loadHttpUrl  // 而非 loadHTTPURL

// ✅ 接口不要添加 I 前缀
interface User { ... }      // 而非 IUser
class UserService { ... }

// ❌ 不要使用前缀/后缀下划线表示私有
this._private  // ❌
this.private_  // ❌
```

---

## 二、类型系统

### 类型推断

```typescript
// ✅ 简单类型可省略注解
const x = 15;  // 自动推断为 number

// ❌ 避免：冗余的类型注解
const x: boolean = true;

// ✅ 复杂表达式建议添加类型注解
const value: string[] = await rpc.getSomeValue().transform();
```

### undefined 与 null

```typescript
// ✅ 优先使用可选字段（?）而非 |undefined
interface User {
  name: string;
  email?: string;  // 推荐
}

// ✅ 在使用时再添加 null 检查
type GoodResponse = Latte;
function getLatte(): GoodResponse | undefined { ... }
```

### any 类型

```typescript
// ❌ 尽量避免使用 any
const danger: any = value;      // ❌

// ✅ 使用 unknown 代替 any（更安全）
const val: unknown = value;      // ✅ 更安全
// 需要先收窄类型才能使用

// ✅ 必要时使用并添加注释
// tslint:disable-next-line:no-any
const mock = ({ get() {} } as any) as BookService;
```

---

## 三、接口 vs 类型别名

```typescript
// ✅ 优先使用接口定义对象类型
interface User {
  firstName: string;
  lastName: string;
}

// ❌ 避免用 type 别名定义对象
type User = {
  firstName: string,
  lastName: string,
};

// ✅ 类型别名用于其他场景（联合类型、元组等）
type StringOrNumber = string | number;
type Callback = () => void;
```

**为什么优先接口？**
- 更好的 IDE 支持
- 更好的错误提示位置
- 更好的类型显示性能

---

## 四、泛型使用

```typescript
// ✅ 优先使用简单的类型构造
// ❌ 避免过度使用映射/条件类型
type FoodPreferences = Pick<User, 'favoriteIcecream' | 'favoriteChocolate'>;

// ✅ 有时直接列出属性更清晰
interface FoodPreferences {
  favoriteIcecream: string;
  favoriteChocolate: string;
}
```

---

## 五、类规范

### 基础声明

```typescript
// ✅ 类声明不加分号
class Foo {
  method() { }
}

// ❌ 方法之间不用分号分隔
class Bad {
  method1();  // ❌
  method2();
}

// ✅ 构造函数必须使用括号
const x = new Foo();  // ✅
const y = new Foo;    // ❌
```

### 属性与可见性

```typescript
// ✅ 优先使用 readonly 和参数属性
class Foo {
  private readonly userList: string[] = [];
  
  constructor(private readonly service: BarService) {}
}

// ❌ 不要使用 #private 字段
class Bad {
  #ident = 1;  // ❌ 性能问题，不兼容旧环境
}

// ✅ 使用 TypeScript 可见性修饰符
class Good {
  private ident = 1;  // ✅
}
```

### this 使用规则

```typescript
// ❌ 禁止在静态方法中使用 this
class Bad {
  static foo() {
    return this.staticField;  // ❌
  }
}

// ✅ 事件处理器建议用法
class Component {
  onAttached() {
    window.addEventListener('click', () => {
      this.listener();
    });
  }
  
  private listener = () => {
    confirm('Exit?');
  }
}

// ✅ 不要用 bind() 安装事件处理器（无法卸载）
window.addEventListener('click', this.listener);  // ✅
window.removeEventListener('click', this.listener);
```

---

## 六、注释规范

### JSDoc vs 普通注释

```typescript
// ✅ JSDoc /** ... */ 用于用户应阅读的文档
// ✅ // 普通注释用于实现细节

/**
 * Posts the request to start coffee brewing.
 * @param amountLitres The amount to brew. Must fit the pot size!
 */
function brew(amountLitres: number) { }
```

### 文档规则

```typescript
// ✅ 导出符号必须添加 JSDoc
/** @deprecated Use newApi() instead. */
export function oldApi() { }

// ❌ 不要仅重复参数名和类型
/** @param fooBarService The Bar service. */  // ❌ 无意义
```

---

## 七、可见性规则

### 原则

```typescript
// 1. 最小化可见性
// ✅ 私有方法转为模块级函数
function helper() { }  // 模块内可见，比 private 方法更灵活

// 2. 模板访问的属性不能用 private
// Angular: 使用 protected
class AngularComponent {
  protected items: Item[];  // 模板需要访问
}

// 3. 永远不要用 obj['foo'] 绕过可见性
this['privateField']  // ❌ 可能被重构破坏
```

---

## 八、导入与导出

```typescript
// ✅ 使用命名导出，不用 default 导出
export class Foo { }
export const SOME_CONSTANT = ...;

// ❌ 避免 default 导出
export default class Foo { }  // ❌ 导致名称不统一

// ✅ 优先命名导入
import { Foo, Bar } from './module';
import * as namespace from './module';  // 大量符号时使用
```

---

## 九、异常处理

```typescript
// ✅ 只抛出 Error 对象
throw new Error('message');  // ✅ 有堆栈信息
throw 'error string';        // ❌ 无法追踪

// ✅ catch 使用 unknown 类型
try {
  doSomething();
} catch (e: unknown) {
  assertIsError(e);
  console.log(e.message);
}

// ✅ 空 catch 块必须注释说明
try { } catch (e) {
  // Response is not numeric. Continue.
}
```

---

## AI 编程调用示例

```markdown
请遵循 Google TypeScript 风格指南：
- 类/接口用 UpperCamelCase，变量用 lowerCamelCase
- 优先 interface 而非类型别名
- 避免 any，使用 unknown
- 私有用 private，不用 #private
- 优先命名导出，不用 default export
- 只 throw Error，catch 用 unknown
- 导出符号必须添加 JSDoc
```
