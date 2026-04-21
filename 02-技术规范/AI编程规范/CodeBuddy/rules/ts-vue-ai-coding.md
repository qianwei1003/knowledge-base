---
name: ts-vue-ai-coding
description: TypeScript+Vue AI编程规范。触发词：开始开发、实现功能、写代码、重构、修复bug、优化性能、添加测试
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.vue"
alwaysApply: true
---

# TypeScript + Vue AI 编程规范

> 基于《通病与陷阱》和 ECC AI Engineering 最佳实践
> 来源：[[02-技术规范/AI编程规范/通病与陷阱]]、[[02-技术规范/AI编程规范/ECC-AI编程与自主循环模式]]
> 更新：2026-04-22 新增 AI 行为约束、封装要求、组件拆分

---

## 核心原则

### AI-First Engineering
- **规划质量 > 打字速度**：先理解需求再动手
- **评估覆盖率 > 主观信心**：用验证代替猜测
- **15分钟单元规则**：每个任务可独立验证、有清晰完成条件

### 编码哲学
- **KISS**：简单 > 巧妙 > 复杂
- **DRY**：不要重复自己
- **YAGNI**：不要实现不需要的功能

### AI 行为底线
- **禁止敷衍**：不快速扫描、不生成模板化报告
- **逐行审查**：每行代码都要检查
- **零容忍**：发现问题必须修复才能继续

---

## 零、AI 行为约束（关键）

### 0.1 禁止敷衍式代码生成

**❌ 错误行为：**
- 快速扫描需求后立即开始编码
- 只关注"功能是否实现"，不关注"实现质量"
- 生成代码后不做自我审查
- 留下明显的低质量问题（硬编码、魔法字符串、未封装等）

**✅ 正确做法：**
- 深入理解需求后再动手
- 生成代码后逐行自我审查
- 主动发现并修复问题
- 以"这代码能过 code review"为标准

**强制执行**：生成代码后必须执行"执行检查清单"第 18 项 ECC 验证

---

### 0.2 禁止敷衍式代码审查

**❌ 错误审查：**
- 快速扫描代码结构
- 确认"有错误处理、有类型声明"就通过
- 给出模板化的"合规"结论
- 不发现具体问题

**✅ 正确审查（逐行证明）：**
```markdown
逐行审查报告：

第 6 行：const API_URL = 'https://api.example.com'
→ ❌ 硬编码，应使用 import.meta.env.VITE_API_URL

第 24 行：throw new Error('登录失败')
→ ❌ 魔法字符串，应使用 ERROR_MESSAGES.LOGIN_FAILED

第 35 行：localStorage.getItem('user')
→ ❌ 未封装，应使用 useStorage composable
```

**强制执行**：
- 审查报告必须包含行号级别的具体问题
- 不能只给出"合规"的整体评价
- 必须对照《通病与陷阱》逐项核对

---

## 一、代码生成类约束

### 1.0 原生 API 必须封装

**❌ 错误模式：**
```typescript
// 直接在业务代码中使用原生 API
const response = await fetch('/api/login', {
  method: 'POST',
  body: JSON.stringify(data)
})

const user = localStorage.getItem('user')
localStorage.setItem('token', token)
```

**✅ 正确做法：**
```typescript
// composables/useApi.ts
export function useApi() {
  const baseURL = import.meta.env.VITE_API_BASE_URL
  
  async function post<T>(url: string, data: unknown): Promise<T> {
    const response = await fetch(`${baseURL}${url}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    })
    if (!response.ok) {
      throw new Error(ERROR_MESSAGES.NETWORK_ERROR)
    }
    return response.json()
  }
  
  return { post }
}

// composables/useStorage.ts
const STORAGE_KEYS = {
  USER: 'app:user',
  TOKEN: 'app:token'
} as const

export function useStorage() {
  function getUser(): User | null {
    const data = localStorage.getItem(STORAGE_KEYS.USER)
    return data ? JSON.parse(data) : null
  }
  
  function setUser(user: User): void {
    localStorage.setItem(STORAGE_KEYS.USER, JSON.stringify(user))
  }
  
  return { getUser, setUser }
}

// 业务代码中使用
const { post } = useApi()
const { setUser } = useStorage()
```

**规则**：
- `fetch` 必须封装在 `useApi` 或类似 composable
- `localStorage`/`sessionStorage` 必须封装
- 第三方 SDK/API 必须抽象适配层
- 禁止在业务组件中直接使用原生 API

---

### 1.1 禁止硬编码配置值

**❌ 错误模式：**
```typescript
const API_URL = "https://api.example.com/v1";
const MAX_RETRIES = 3;
const TIMEOUT_MS = 5000;
```

**✅ 正确做法：**
```typescript
// 使用环境变量 + 配置验证
const API_URL = process.env.API_URL;
const MAX_RETRIES = parseInt(process.env.MAX_RETRIES || '3', 10);

if (!API_URL) {
  throw new Error('API_URL environment variable is required');
}
```

**强制执行：** 发现硬编码配置值时必须重构到配置文件

---

### 1.2 禁止硬编码敏感信息

**❌ 错误模式：**
```typescript
const API_KEY = "sk-1234567890abcdef";
const PASSWORD = "admin123";
```

**✅ 正确做法：**
```typescript
const API_KEY = process.env.API_KEY;
const PASSWORD = process.env.ADMIN_PASSWORD;

if (!API_KEY) {
  throw new Error('API_KEY environment variable is required');
}
```

**安全检查：** 每次提交前扫描 `sk-`、`password`、`secret` 等关键词

---

### 1.3 禁止过度设计

**判断标准：**
- 一个类/函数只被使用一次 → 简化
- 简单功能使用工厂模式/多层抽象 → 重构
- 过度使用泛型/模板 → 简化

**KISS原则：** 如果不用设计模式也能清晰实现，就不用

---

### 1.4 禁止遗留 TODO/FIXME

**❌ 错误模式：**
```typescript
function processData(data: unknown) {
  // TODO: implement validation
  // FIXME: add error handling
  // TODO: handle edge cases
  return data;
}
```

**✅ 正确做法：**
- 拆分复杂任务为多个可独立完成的子任务
- 每个子任务必须完整实现，不允许遗留 TODO
- 确实需要后续处理的，使用 issue 跟踪而非代码注释

**强制执行：** 代码审查时必须清零 TODO/FIXME

---

## 二、错误处理类约束

### 2.1 异步操作必须错误处理

**❌ 错误模式：**
```typescript
const data = JSON.parse(userInput);
const user = await getUser(userId);
```

**✅ 正确做法：**
```typescript
// JSON解析
try {
  const data = JSON.parse(userInput);
} catch (error) {
  logger.error('Invalid JSON input:', error);
  return { error: 'Invalid input format' };
}

// 异步操作
try {
  const user = await getUser(userId);
} catch (error) {
  if (error instanceof NotFoundError) {
    return { error: 'User not found' };
  }
  logger.error('Failed to get user:', error);
  throw error;
}
```

**规则：** 所有 `await` 和 `JSON.parse` 必须包裹 try-catch

---

### 2.2 必须空值检查

**❌ 错误模式：**
```typescript
const name = user.profile.name;
const value = data.items[0].value;
```

**✅ 正确做法：**
```typescript
// 使用可选链
const name = user?.profile?.name ?? 'Anonymous';

// 数组检查
const value = data.items?.[0]?.value;

// 提前返回模式
if (!user?.profile) {
  return { error: 'Profile not found' };
}
const name = user.profile.name;
```

**规则：** 访问对象属性或数组元素前必须验证存在性

---

### 2.3 禁止过于宽泛的异常捕获

**❌ 错误模式：**
```typescript
try {
  doSomething();
} catch (error) {
  // 什么都不做或只打印日志
  console.log(error);
}
```

**✅ 正确做法：**
```typescript
try {
  doSomething();
} catch (error) {
  if (error instanceof ValidationError) {
    return { error: 'Invalid input', details: error.message };
  }
  if (error instanceof ConnectionError) {
    logger.error('Connection failed:', error);
    return { error: 'Service unavailable' };
  }
  throw error; // 未知错误继续抛出
}
```

**规则：** 捕获具体异常类型，禁止裸 `catch (e) { }`

---

## 三、安全类约束

### 3.1 防止 SQL 注入

**❌ 错误模式：**
```typescript
const query = `SELECT * FROM users WHERE name = '${username}'`;
```

**✅ 正确做法：**
```typescript
// 使用参数化查询
const query = "SELECT * FROM users WHERE name = $1";
await db.query(query, [username]);

// 或使用 ORM
await User.findOne({ where: { name: username } });
```

**规则：** 永远使用参数化查询，禁止字符串拼接 SQL

---

### 3.2 防止 XSS

**❌ 错误模式：**
```vue
<template>
  <div v-html="userInput"></div>
</template>
```

**✅ 正确做法：**
```vue
<template>
  <!-- Vue 默认转义 -->
  <div>{{ userInput }}</div>
  
  <!-- 必须使用 v-html 时先净化 -->
  <div v-html="purifiedHtml"></div>
</template>

<script setup>
import DOMPurify from 'dompurify';
const purifiedHtml = DOMPurify.sanitize(userInput);
</script>
```

**规则：** 用户输入必须转义后输出

---

### 3.3 必须输入验证

**❌ 错误模式：**
```typescript
function calculateAge(birthYear: number) {
  return new Date().getFullYear() - birthYear;
}
```

**✅ 正确做法：**
```typescript
import { z } from 'zod';

// 使用 Zod 定义 schema
const CalculateAgeSchema = z.object({
  birthYear: z.number()
    .int()
    .min(1900)
    .max(new Date().getFullYear())
});

function calculateAge(input: unknown) {
  const { birthYear } = CalculateAgeSchema.parse(input);
  return new Date().getFullYear() - birthYear;
}
```

**规则：** 所有函数参数必须验证类型、格式、范围

---

## 四、TypeScript 类型安全约束

### 4.1 禁止随意使用 any

**❌ 错误模式：**
```typescript
function processData(data: any) {
  return data.value.name;
}
```

**✅ 正确做法：**
```typescript
// 定义具体类型
interface UserData {
  value: {
    name: string;
  };
}

function processData(data: UserData) {
  return data.value.name;
}

// 确实无法确定类型时使用 unknown
function processUnknown(data: unknown) {
  if (isUserData(data)) {
    return data.value.name;
  }
  throw new Error('Invalid data format');
}
```

**规则：**
- 禁止使用 `any`，使用 `unknown` + 类型守卫替代
- 特殊情况需使用 `any` 时必须注释理由

---

### 4.2 禁止滥用类型断言

**❌ 错误模式：**
```typescript
const user = getData() as User;
const element = document.getElementById('app') as HTMLElement;
```

**✅ 正确做法：**
```typescript
// 使用类型守卫
const data = getData();
if (isUser(data)) {
  processUser(data);
}

// 使用条件判断
const element = document.getElementById('app');
if (!element) {
  throw new Error('Element #app not found');
}
```

**规则：** 类型断言是最后手段，优先使用类型守卫

---

### 4.3 优先使用 interface

**✅ 推荐：**
```typescript
// 使用 interface
interface User {
  id: string;
  name: string;
}

// 可扩展
interface AdminUser extends User {
  permissions: string[];
}
```

**规则：** 定义对象类型时优先 `interface` 而非 `type`

---

## 五、Vue 3 组合式 API 约束

### 5.1 强制使用 script setup

**❌ 错误模式：**
```vue
<script lang="ts">
export default {
  data() {
    return { count: 0 };
  },
  methods: {
    increment() {
      this.count++;
    }
  }
}
</script>
```

**✅ 正确做法：**
```vue
<script setup lang="ts">
import { ref } from 'vue';

const count = ref(0);

function increment() {
  count.value++;
}
</script>
```

**规则：** 必须使用 `<script setup>` 语法

---

### 5.2 Props 必须类型声明

**❌ 错误模式：**
```vue
<script setup>
const props = defineProps(['title', 'count']);
</script>
```

**✅ 正确做法：**
```vue
<script setup lang="ts">
interface Props {
  title: string;
  count: number;
  optional?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  optional: false
});
</script>
```

**规则：** Props 必须使用泛型类型声明

---

### 5.3 Emits 必须声明

**❌ 错误模式：**
```vue
<script setup>
// 直接emit，不声明
function handleClick() {
  emit('update', value);
}
</script>
```

**✅ 正确做法：**
```vue
<script setup lang="ts">
interface Emits {
  (e: 'update', value: string): void;
  (e: 'delete', id: number): void;
}

const emit = defineEmits<Emits>();

function handleClick() {
  emit('update', 'new value');
}
</script>
```

**规则：** Emits 必须使用 `defineEmits` 声明类型

---

## 六、代码质量类约束

### 6.1 命名规范

**❌ 错误模式：**
```typescript
let dataArray = [];
let tempVar = getSomething();
let result2 = process(data2);
```

**✅ 正确做法：**
```typescript
let userList = [];
let temporaryValue = getUserInput();
let processedResult = process(rawData);
```

**规则：**
- 变量名：camelCase，有意义
- 常量：UPPER_SNAKE_CASE
- 类/接口：PascalCase
- 布尔值：isXxx、hasXxx、shouldXxx

---

### 6.2 禁止魔法数字/字符串（强化版）

**包括范围**（不仅限于业务常量）：
- 业务常量（年龄限制、分页大小等）
- 错误消息
- 事件名称
- 存储 key
- API 路径/端点
- CSS 类名

**❌ 错误模式：**
```typescript
// 业务常量
if (user.age > 18) { }

// 错误消息
throw new Error('登录失败')
throw new Error('网络错误')

// 事件名称
emit('success', user)
emit('update', value)

// 存储 key
localStorage.setItem('user', data)
localStorage.getItem('token')

// API 路径
fetch('/api/v1/users')
```

**✅ 正确做法：**
```typescript
// constants/business.ts
export const BUSINESS_RULES = {
  ADULT_AGE: 18,
  MAX_RETRY_COUNT: 3,
  DEFAULT_PAGE_SIZE: 20
} as const

// constants/messages.ts
export const ERROR_MESSAGES = {
  LOGIN_FAILED: '登录失败，请检查账号密码',
  NETWORK_ERROR: '网络连接失败，请稍后重试',
  VALIDATION_ERROR: '输入信息有误，请检查'
} as const

// constants/events.ts
export const EVENTS = {
  LOGIN_SUCCESS: 'auth:login-success',
  LOGIN_FAILURE: 'auth:login-failure',
  USER_UPDATED: 'user:updated'
} as const

// constants/storage.ts
export const STORAGE_KEYS = {
  USER: 'app:user:v1',
  TOKEN: 'app:token:v1',
  PREFERENCES: 'app:prefs:v1'
} as const

// constants/api.ts
export const API_ENDPOINTS = {
  AUTH_LOGIN: '/auth/login',
  AUTH_LOGOUT: '/auth/logout',
  USER_PROFILE: '/users/profile'
} as const

// constants/css.ts
export const CSS_CLASSES = {
  BUTTON_PRIMARY: 'btn-primary',
  BUTTON_DISABLED: 'btn-disabled',
  FORM_ERROR: 'form-error'
} as const
```

**规则**：
- 所有字面量字符串/数字必须提取为常量
- 按用途分类到不同常量文件
- 使用 `as const` 确保类型安全
- 存储 key 建议加版本前缀（如 `app:user:v1`）便于升级

---

### 6.3 组件拆分与单一职责

**❌ 错误模式：**
```vue
<!-- UserLogin.vue - 250 行 -->
<template>
  <div class="login-page">
    <div class="form-group">
      <label>用户名</label>
      <input v-model="username" @blur="validateUsername" />
      <span v-if="usernameError">{{ usernameError }}</span>
    </div>
    <div class="form-group">
      <label>密码</label>
      <input v-model="password" type="password" @blur="validatePassword" />
      <span v-if="passwordError">{{ passwordError }}</span>
    </div>
    <!-- 还有 200 行... -->
  </div>
</template>
```

**✅ 正确做法：**
```vue
<!-- UserLogin.vue - 80 行 -->
<template>
  <form @submit.prevent="handleLogin">
    <BaseInput
      v-model="form.username"
      label="用户名"
      :error="errors.username"
      @blur="validateField('username')"
    />
    <BaseInput
      v-model="form.password"
      type="password"
      label="密码"
      :error="errors.password"
      @blur="validateField('password')"
    />
    <BaseButton type="submit" :loading="isLoading">
      登录
    </BaseButton>
  </form>
</template>

<!-- BaseInput.vue - 可复用组件 -->
<template>
  <div class="form-field">
    <label v-if="label">{{ label }}</label>
    <input
      :type="type"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @blur="$emit('blur')"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>
```

**规则**：
- 组件超过 150 行必须考虑拆分
- 可复用的 UI 元素必须组件化（BaseInput、BaseButton 等）
- 复杂表单拆分子组件
- 遵循单一职责原则：一个组件只做一件事

---

### 6.4 避免重复代码

**规则：**
- 相同逻辑出现 3 次以上 → 提取公共函数
- 相似组件 → 提取可复用组件
- 定期重构，遵循 DRY 原则

---

## 七、性能类约束

### 7.1 避免循环内重复计算

**❌ 错误模式：**
```typescript
for (const item of items) {
  const result = expensiveCalculation(); // 每次循环都计算
  process(item, result);
}
```

**✅ 正确做法：**
```typescript
const cachedResult = expensiveCalculation(); // 循环外计算一次
for (const item of items) {
  process(item, cachedResult);
}
```

---

### 7.2 避免不必要内存分配

**❌ 错误模式：**
```typescript
const result = data.map(x => {
  const temp = compute(x);
  return temp * 2;
});
```

**✅ 正确做法：**
```typescript
const result = data.map(x => compute(x) * 2);
```

---

## 八、测试类约束

### 8.1 必须测试覆盖

**测试场景：**
1. 正常输入（happy path）
2. 边界值（0、最大值、空值、null）
3. 异常输入（非法格式、超大值）
4. 并发情况（如有）

**覆盖率要求：**
- 单元测试：≥80%
- 集成测试：关键路径全覆盖
- E2E测试：核心用户流程

---

### 8.2 测试行为而非实现

**❌ 错误模式：**
```typescript
// 测试内部状态
expect(component.state.count).toBe(5);
expect(obj._cache.get('key')).toBe(value);
```

**✅ 正确做法：**
```typescript
// 测试用户可见行为
expect(screen.getByText('Count: 5')).toBeInTheDocument();
expect(await obj.getValue('key')).toBe(expectedValue);
```

---

## 九、文档类约束

### 9.1 公共 API 必须文档注释

**规范格式：**
```typescript
/**
 * 获取用户信息
 * 
 * @param userId - 用户 ID，支持 UUID 格式
 * @param options - 可选配置
 * @param options.includeProfile - 是否包含详细资料，默认 false
 * @returns 用户信息对象，包含 id、name、email 字段
 * @throws {NotFoundError} 当用户不存在时
 * @throws {ValidationError} 当 userId 格式无效时
 * 
 * @example
 * const user = await getUser('123', { includeProfile: true });
 */
async function getUser(
  userId: string,
  options?: { includeProfile?: boolean }
): Promise<User> {
  // ...
}
```

**规则：**
- 所有导出函数必须有 JSDoc 注释
- 参数、返回值、异常必须说明
- 提供使用示例

---

## 十、ECC 验证循环（强制）

每次代码产出后必须执行：

```
Phase 1: 构建验证    → npm run build
Phase 2: 类型检查    → npx tsc --noEmit
Phase 3: Lint 检查   → npm run lint
Phase 4: 测试套件    → npm run test -- --coverage (目标80%)
Phase 5: 安全扫描    → 检查 sk-、console.log、敏感信息
Phase 6: Diff 审查   → git diff --stat 审查每个变更
```

**验证报告格式：**
```
VERIFICATION REPORT
==================
构建:     [PASS/FAIL]
类型:     [PASS/FAIL] (X errors)
Lint:     [PASS/FAIL] (X warnings)
测试:     [PASS/FAIL] (X/Y passed, Z% coverage)
安全:     [PASS/FAIL] (X issues)
Diff:     [X files changed]

Overall:  [READY/NOT READY] for PR

Issues to Fix:
1. ...
```

**规则：** 验证不通过不得提交代码

---

## 执行检查清单

### 生成代码前检查

- [ ] 深入理解需求（不是扫描关键词）
- [ ] 规划实现方案（不是立即编码）

### 生成代码时检查

- [ ] 无硬编码配置值/敏感信息
- [ ] 原生 API（fetch/localStorage）已封装
- [ ] 无 TODO/FIXME/NotImplementedError
- [ ] 异步操作有 try-catch
- [ ] 有空值检查（?. ??）
- [ ] 异常捕获具体类型
- [ ] 用户输入已验证（Zod）
- [ ] 无 SQL 注入/XSS 风险
- [ ] 无随意使用 any
- [ ] 优先使用 interface
- [ ] 使用 script setup
- [ ] Props/Emits 有类型声明
- [ ] 命名规范（camelCase/PascalCase）
- [ ] 无魔法数字/字符串（含错误消息、事件名、存储key）
- [ ] 组件不超过 150 行（否则拆分）
- [ ] 可复用 UI 已组件化
- [ ] 循环内无重复计算

### 生成代码后检查（逐行审查）

- [ ] 逐行阅读代码，不是扫描
- [ ] 检查变量/函数命名是否清晰
- [ ] 检查是否有重复代码可提取
- [ ] 检查组件是否过度臃肿
- [ ] 对照《通病与陷阱》逐一核对
- [ ] 问自己：这代码能过 review 吗？

### 测试与文档

- [ ] 测试覆盖率 ≥80%
- [ ] 公共 API 有文档注释
- [ ] ECC 六阶段验证通过
- [ ] 审查报告包含行号级具体问题

---

## 经验教训

> 参见：[[ai-coding-test-project/docs/ai-review-lessons]]

本次规范迭代的核心改进来自实践教训：

1. **AI 敷衍问题**：快速扫描→逐行审查
2. **原生 API 未封装**：直接调用→强制封装
3. **魔法值范围窄**：业务常量→含错误消息/事件名/存储key
4. **组件臃肿**：无限制→150行上限/强制拆分

---

*规范来源：[[02-技术规范/AI编程规范/通病与陷阱]]、[[02-技术规范/AI编程规范/ECC-AI编程与自主循环模式]]*  
*更新记录：2026-04-22 新增 AI 行为约束、API 封装要求、组件拆分规范*
