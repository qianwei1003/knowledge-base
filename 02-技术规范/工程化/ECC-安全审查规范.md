# ECC 安全审查规范

> 来源：ECC `security-review` 技能 | 提炼时间：2026-04-20

---

## 目录

- [激活条件](#激活条件)
- [Secrets 管理](#secrets-管理)
- [输入验证](#输入验证)
- [SQL 注入预防](#sql-注入预防)
- [认证与授权](#认证与授权)
- [XSS 预防](#xss-预防)
- [CSRF 保护](#csrf-保护)
- [限流](#限流)
- [敏感数据暴露](#敏感数据暴露)
- [区块链安全](#区块链安全-solana)
- [依赖安全](#依赖安全)
- [安全测试](#安全测试)
- [部署前检查清单](#部署前检查清单)

---

## 激活条件

- 实现认证或授权
- 处理用户输入或文件上传
- 创建新的 API 端点
- 使用 secrets 或凭据
- 实现支付功能
- 存储或传输敏感数据
- 集成第三方 API

---

## Secrets 管理

### ❌ 禁止

```typescript
const apiKey = "sk-proj-xxxxx"  // 硬编码 secret
const dbPassword = "password123" // 写在源码中
```

### ✅ 必须

```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 验证 secrets 存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

### 验证步骤

- [ ] 无硬编码 API keys、tokens 或 passwords
- [ ] 所有 secrets 在环境变量中
- [ ] `.env.local` 在 .gitignore 中
- [ ] git 历史中无 secrets
- [ ] 生产 secrets 在托管平台（Vercel、Railway）

---

## 输入验证

### 始终验证用户输入

```typescript
import { z } from 'zod'

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

### 文件上传验证

```typescript
function validateFileUpload(file: File) {
  // 大小检查（最大 5MB）
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // 类型检查
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 扩展名检查
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

---

## SQL 注入预防

### ❌ 禁止：拼接 SQL

```typescript
// 危险 - SQL 注入漏洞
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

### ✅ 必须：参数化查询

```typescript
// 安全 - 参数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 或使用原生 SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

---

## 认证与授权

### JWT Token 处理

```typescript
// ❌ 错误：localStorage（易受 XSS 攻击）
localStorage.setItem('token', token)

// ✅ 正确：httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

### 授权检查

```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 始终先验证授权
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  await db.users.delete({ where: { id: userId } })
}
```

### Supabase Row Level Security

```sql
-- 启用 RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 用户只能查看自己的数据
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);
```

---

## XSS 预防

### 净化 HTML

```typescript
import DOMPurify from 'isomorphic-dompurify'

function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

### Content Security Policy

```typescript
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

---

## CSRF 保护

### CSRF Tokens

```typescript
export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }
}
```

### SameSite Cookies

```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

---

## 限流

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每窗口 100 请求
  message: 'Too many requests'
})

app.use('/api/', limiter)

// 昂贵操作更严格限制
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 分钟
  max: 10, // 每分钟 10 次
  message: 'Too many search requests'
})
```

---

## 敏感数据暴露

### 日志

```typescript
// ❌ 错误：记录敏感数据
console.log('User login:', { email, password })

// ✅ 正确：脱敏
console.log('User login:', { email, userId })
```

### 错误消息

```typescript
// ❌ 错误：暴露内部细节
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 正确：通用错误消息
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

---

## 区块链安全（Solana）

### 钱包验证

```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

### 交易验证

```typescript
async function verifyTransaction(transaction: Transaction) {
  // 验证接收方
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // 验证金额
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // 验证余额
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

---

## 依赖安全

```bash
# 检查漏洞
npm audit

# 自动修复可修复的问题
npm audit fix

# 更新依赖
npm update

# 检查过期包
npm outdated
```

---

## 安全测试

```typescript
// 测试认证
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 测试授权
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 测试输入验证
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// 测试限流
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )
  const responses = await Promise.all(requests)
  expect(responses.some(r => r.status === 429)).toBe(true)
})
```

---

## 部署前检查清单

生产部署前必须检查：

- [ ] **Secrets**: 无硬编码 secrets，全部在环境变量中
- [ ] **输入验证**: 所有用户输入已验证
- [ ] **SQL 注入**: 所有查询参数化
- [ ] **XSS**: 用户内容已净化
- [ ] **CSRF**: 保护已启用
- [ ] **认证**: 正确的 token 处理
- [ ] **授权**: 角色检查到位
- [ ] **限流**: 所有端点启用
- [ ] **HTTPS**: 生产环境强制
- [ ] **安全头**: CSP、X-Frame-Options 已配置
- [ ] **错误处理**: 不向用户暴露敏感数据
- [ ] **日志**: 不记录敏感数据
- [ ] **依赖**: 最新、无漏洞
- [ ] **Row Level Security**: Supabase 已启用
- [ ] **CORS**: 正确配置
- [ ] **文件上传**: 已验证（大小、类型）
- [ ] **钱包签名**: 已验证（如有区块链）

---

**记住**：安全不是可选项。一个漏洞可以摧毁整个平台。有疑问时，宁可谨慎。
