# ECC 编码规范

> 来源：ECC `coding-standards` 技能 | 提炼时间：2026-04-20

---

## 目录

- [激活条件](#激活条件)
- [代码质量原则](#代码质量原则)
- [TypeScript/JavaScript 标准](#typescriptjavascript-标准)
- [React 最佳实践](#react-最佳实践)
- [API 设计标准](#api-设计标准)
- [文件组织](#文件组织)
- [注释与文档](#注释与文档)
- [性能最佳实践](#性能最佳实践)
- [测试标准](#测试标准)
- [代码异味检测](#代码异味检测)

---

## 激活条件

- 启动新项目或模块
- 审查代码质量和可维护性
- 重构现有代码遵循规范
- 强制命名、格式或结构一致性
- 设置 lint、format 或类型检查规则

---

## 代码质量原则

### 1. 可读性优先
- 代码被读的次数多于写的次数
- 清晰的变量和函数名
- 优先使用自文档化代码而非注释
- 一致的格式

### 2. KISS（保持简单）
- 最简单的可行方案
- 避免过度工程
- 不做premature optimization
- 易于理解 > 聪明的代码

### 3. DRY（不要重复）
- 提取公共逻辑到函数
- 创建可复用组件
- 跨模块共享工具
- 避免复制粘贴编程

### 4. YAGNI（你不需要它）
- 不在需要之前构建功能
- 避免投机性泛化
- 仅在需要时添加复杂性
- 从简单开始，需要时重构

---

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// ✅ 好：描述性名称
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 差：含义不清
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// ✅ 好：动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 差：含义不清或只用名词
async function market(id: string) { }
function similarity(a, b) { }
```

### 不可变性模式（关键）

```typescript
// ✅ 始终使用展开运算符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 永远不要直接修改
user.name = 'New Name'  // ❌
items.push(newItem)     // ❌
```

### 错误处理

```typescript
// ✅ 好：全面的错误处理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ 差：无错误处理
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 最佳实践

```typescript
// ✅ 好：可能时并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 差：不必要时顺序执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 类型安全

```typescript
// ✅ 好：正确的类型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> { }

// ❌ 差：使用 any
function getMarket(id: any): Promise<any> { }
```

---

## React 最佳实践

### 组件结构

```typescript
// ✅ 好：带类型的函数组件
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}
```

### 自定义 Hooks

```typescript
// ✅ 好：可复用自定义 hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### 状态管理

```typescript
// ✅ 好：正确的状态更新
const [count, setCount] = useState(0)

// 基于前一状态更新
setCount(prev => prev + 1)

// ❌ 差：直接引用状态（异步场景可能过期）
setCount(count + 1)
```

### 条件渲染

```typescript
// ✅ 好：清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 差：三元地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

---

## API 设计标准

### REST API 约定

```
GET    /api/markets              # 列出所有市场
GET    /api/markets/:id          # 获取特定市场
POST   /api/markets              # 创建新市场
PUT    /api/markets/:id          # 更新市场（完整）
PATCH  /api/markets/:id          # 更新市场（部分）
DELETE /api/markets/:id          # 删除市场

# 查询参数过滤
GET /api/markets?status=active&limit=10&offset=0
```

### 响应格式

```typescript
// ✅ 好：一致的响应结构
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

---

## 文件组织

### 项目结构

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 路由
│   ├── markets/           # 市场页面
│   └── (auth)/           # 认证页面（路由组）
├── components/            # React 组件
│   ├── ui/               # 通用 UI 组件
│   ├── forms/            # 表单组件
│   └── layouts/          # 布局组件
├── hooks/                # 自定义 React hooks
├── lib/                  # 工具和配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型
└── styles/              # 全局样式
```

### 文件命名

```
components/Button.tsx          # 组件用 PascalCase
hooks/useAuth.ts              # hooks 用 camelCase 并加 use 前缀
lib/formatDate.ts             # 工具用 camelCase
types/market.types.ts         # 类型用 camelCase 加 .types 后缀
```

---

## 注释与文档

### 何时注释

```typescript
// ✅ 好：解释原因，不解释做什么
// Use exponential backoff to avoid overwhelming the API during outages
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// ❌ 差：陈述显而易见的事
// Increment counter by 1
count++
```

---

## 测试标准

### 测试结构（AAA 模式）

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// ✅ 好：描述性测试名
test('returns empty array when no markets match query', () => { })
test('throws error when OpenAI API key is missing', () => { })
test('falls back to substring search when Redis unavailable', () => { })

// ❌ 差：模糊测试名
test('works', () => { })
test('test search', () => { })
```

---

## 代码异味检测

### 1. 过长函数
```typescript
// ❌ 差：函数超过 50 行
function processMarketData() {
  // 100 lines of code
}

// ✅ 好：拆分为小函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深层嵌套
```typescript
// ❌ 差：5+ 层嵌套
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) { /* Do something */ }
      }
    }
  }
}

// ✅ 好：提前返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return
```

### 3. 魔法数字
```typescript
// ❌ 差：未解释的数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 好：命名常量
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500
if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

---

**记住**：代码质量是不可妥协的。清晰、可维护的代码才能实现快速开发和有信心的重构。
