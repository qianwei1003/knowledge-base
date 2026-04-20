# ECC 后端开发模式

> 来源：ECC `backend-patterns` 技能 | 提炼时间：2026-04-20

---

## 目录

- [激活条件](#激活条件)
- [API 设计模式](#api-设计模式)
- [数据库模式](#数据库模式)
- [缓存策略](#缓存策略)
- [错误处理模式](#错误处理模式)
- [认证与授权](#认证与授权)
- [限流](#限流)
- [后台任务与队列](#后台任务与队列)
- [日志与监控](#日志与监控)

---

## 激活条件

- 设计 REST 或 GraphQL API 端点
- 实现 Repository、Service 或 Controller 分层
- 优化数据库查询（N+1、索引、连接池）
- 添加缓存（Redis、内存、HTTP 缓存头）
- 设置后台任务或异步处理
- 结构性错误处理与验证
- 构建中间件（认证、日志、限流）

---

## API 设计模式

### RESTful API 结构

```typescript
// ✅ 资源型 URL
GET    /api/markets                   // 列出资源
GET    /api/markets/:id              // 获取单个资源
POST   /api/markets                   // 创建资源
PUT    /api/markets/:id              // 替换资源
PATCH  /api/markets/:id              // 部分更新
DELETE /api/markets/:id              // 删除资源

// ✅ 查询参数用于过滤、排序、分页
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### Repository 模式

```typescript
// 抽象数据访问逻辑
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>
  findById(id: string): Promise<Market | null>
  create(data: CreateMarketDto): Promise<Market>
  update(id: string, data: UpdateMarketDto): Promise<Market>
  delete(id: string): Promise<void>
}

class SupabaseMarketRepository implements MarketRepository {
  async findAll(filters?: MarketFilters): Promise<Market[]> {
    let query = supabase.from('markets').select('*')

    if (filters?.status) {
      query = query.eq('status', filters.status)
    }

    if (filters?.limit) {
      query = query.limit(filters.limit)
    }

    const { data, error } = await query

    if (error) throw new Error(error.message)
    return data
  }

  // 其他方法...
}
```

### Service 层模式

```typescript
// 业务逻辑与数据访问分离
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // 业务逻辑
    const embedding = await generateEmbedding(query)
    const results = await this.vectorSearch(embedding, limit)

    // 获取完整数据
    const markets = await this.marketRepo.findByIds(results.map(r => r.id))

    // 按相似度排序
    return markets.sort((a, b) => {
      const scoreA = results.find(r => r.id === a.id)?.score || 0
      const scoreB = results.find(r => r.id === b.id)?.score || 0
      return scoreA - scoreB
    })
  }
}
```

### Middleware 模式

```typescript
// 请求/响应处理管道
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '')

    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' })
    }

    try {
      const user = await verifyToken(token)
      req.user = user
      return handler(req, res)
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' })
    }
  }
}
```

---

## 数据库模式

### 查询优化

```typescript
// ✅ 只选择需要的列
const { data } = await supabase
  .from('markets')
  .select('id, name, status, volume')
  .eq('status', 'active')
  .order('volume', { ascending: false })
  .limit(10)

// ❌ 不要选择全部
const { data } = await supabase.from('markets').select('*')
```

### N+1 查询预防

```typescript
// ❌ N+1 查询问题
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id)  // N 次查询
}

// ✅ 批量获取
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 1 次查询
const creatorMap = new Map(creators.map(c => [c.id, c]))

markets.forEach(market => {
  market.creator = creatorMap.get(market.creator_id)
})
```

### 事务模式

```typescript
async function createMarketWithPosition(
  marketData: CreateMarketDto,
  positionData: CreatePositionDto
) {
  const { data, error } = await supabase.rpc('create_market_with_position', {
    market_data: marketData,
    position_data: positionData
  })

  if (error) throw new Error('Transaction failed')
  return data
}
```

---

## 缓存策略

### Redis 缓存层

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient
  ) {}

  async findById(id: string): Promise<Market | null> {
    // 先检查缓存
    const cached = await this.redis.get(`market:${id}`)

    if (cached) {
      return JSON.parse(cached)
    }

    // 缓存未命中 - 从数据库获取
    const market = await this.baseRepo.findById(id)

    if (market) {
      // 缓存 5 分钟
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
    }

    return market
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`)
  }
}
```

### Cache-Aside 模式

```typescript
async function getMarketWithCache(id: string): Promise<Market> {
  const cacheKey = `market:${id}`

  // 尝试缓存
  const cached = await redis.get(cacheKey)
  if (cached) return JSON.parse(cached)

  // 缓存未命中 - 从 DB 获取
  const market = await db.markets.findUnique({ where: { id } })

  if (!market) throw new Error('Market not found')

  // 更新缓存
  await redis.setex(cacheKey, 300, JSON.stringify(market))

  return market
}
```

---

## 错误处理模式

### 集中式错误处理器

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
    Object.setPrototypeOf(this, ApiError.prototype)
  }
}

export function errorHandler(error: unknown, req: Request): Response {
  if (error instanceof ApiError) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: error.statusCode })
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json({
      success: false,
      error: 'Validation failed',
      details: error.errors
    }, { status: 400 })
  }

  // 记录意外错误
  console.error('Unexpected error:', error)

  return NextResponse.json({
    success: false,
    error: 'Internal server error'
  }, { status: 500 })
}
```

### 指数退避重试

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError: Error

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error as Error

      if (i < maxRetries - 1) {
        // 指数退避: 1s, 2s, 4s
        const delay = Math.pow(2, i) * 1000
        await new Promise(resolve => setTimeout(resolve, delay))
      }
    }
  }

  throw lastError!
}
```

---

## 认证与授权

### JWT Token 验证

```typescript
export async function requireAuth(request: Request) {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')

  if (!token) {
    throw new ApiError(401, 'Missing authorization token')
  }

  return verifyToken(token)
}

// 使用
export async function GET(request: Request) {
  const user = await requireAuth(request)
  const data = await getDataForUser(user.userId)
  return NextResponse.json({ success: true, data })
}
```

### 基于角色的访问控制

```typescript
type Permission = 'read' | 'write' | 'delete' | 'admin'

const rolePermissions: Record<User['role'], Permission[]> = {
  admin: ['read', 'write', 'delete', 'admin'],
  moderator: ['read', 'write', 'delete'],
  user: ['read', 'write']
}

export function hasPermission(user: User, permission: Permission): boolean {
  return rolePermissions[user.role].includes(permission)
}

export function requirePermission(permission: Permission) {
  return (handler: (request: Request, user: User) => Promise<Response>) => {
    return async (request: Request) => {
      const user = await requireAuth(request)

      if (!hasPermission(user, permission)) {
        throw new ApiError(403, 'Insufficient permissions')
      }

      return handler(request, user)
    }
  }
}
```

---

## 限流

### 内存限流器

```typescript
class RateLimiter {
  private requests = new Map<string, number[]>()

  async checkLimit(
    identifier: string,
    maxRequests: number,
    windowMs: number
  ): Promise<boolean> {
    const now = Date.now()
    const requests = this.requests.get(identifier) || []

    // 移除窗口外的请求
    const recentRequests = requests.filter(time => now - time < windowMs)

    if (recentRequests.length >= maxRequests) {
      return false  // 超过限流
    }

    recentRequests.push(now)
    this.requests.set(identifier, recentRequests)

    return true
  }
}
```

---

## 后台任务与队列

### 简单队列模式

```typescript
class JobQueue<T> {
  private queue: T[] = []
  private processing = false

  async add(job: T): Promise<void> {
    this.queue.push(job)
    if (!this.processing) {
      this.process()
    }
  }

  private async process(): Promise<void> {
    this.processing = true
    while (this.queue.length > 0) {
      const job = this.queue.shift()!
      try {
        await this.execute(job)
      } catch (error) {
        console.error('Job failed:', error)
      }
    }
    this.processing = false
  }

  private async execute(job: T): Promise<void> {
    // 任务执行逻辑
  }
}
```

---

## 日志与监控

### 结构化日志

```typescript
class Logger {
  log(level: 'info' | 'warn' | 'error', message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context
    }
    console.log(JSON.stringify(entry))
  }

  info(message: string, context?: LogContext) {
    this.log('info', message, context)
  }

  error(message: string, error: Error, context?: LogContext) {
    this.log('error', message, {
      ...context,
      error: error.message,
      stack: error.stack
    })
  }
}
```

---

**记住**：后端模式使可扩展、可维护的服务端应用成为可能。选择适合你复杂度级别的模式。
