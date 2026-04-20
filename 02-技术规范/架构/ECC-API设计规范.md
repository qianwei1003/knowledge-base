# ECC API 设计规范

> 来源：ECC `api-design` 技能 | 提炼时间：2026-04-20

---

## 目录

- [激活条件](#激活条件)
- [资源设计](#资源设计)
- [HTTP 方法与状态码](#http-方法与状态码)
- [响应格式](#响应格式)
- [分页](#分页)
- [过滤、排序与搜索](#过滤排序与搜索)
- [认证与授权](#认证与授权)
- [限流](#限流)
- [版本控制](#版本控制)
- [实现模式](#实现模式)
- [设计检查清单](#设计检查清单)

---

## 激活条件

- 设计新的 API 端点
- 审查现有 API 契约
- 添加分页、过滤或排序
- 实现 API 错误处理
- 规划 API 版本策略
- 构建公开或合作伙伴 API

---

## 资源设计

### URL 结构

```
# 资源用名词、复数、小写、kebab-case
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# 子资源表示关系
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# 不符合 CRUD 的操作（少用动词）
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
```

### 命名规则

```
# ✅ 好
/api/v1/team-members          # kebab-case
/api/v1/orders?status=active # 查询参数过滤
/api/v1/users/123/orders      # 嵌套资源表示所有权

# ❌ 差
/api/v1/getUsers              # URL 中使用动词
/api/v1/user                  # 单数（用复数）
/api/v1/team_members          # URL 中使用 snake_case
/api/v1/users/123/getOrders   # 嵌套资源中用动词
```

---

## HTTP 方法与状态码

### 方法语义

| 方法 | 幂等 | 安全 | 用途 |
|------|------|------|------|
| GET | 是 | 是 | 检索资源 |
| POST | 否 | 否 | 创建资源、触发操作 |
| PUT | 是 | 否 | 完整替换资源 |
| PATCH | 否* | 否 | 部分更新资源 |
| DELETE | 是 | 否 | 删除资源 |

### 状态码参考

```
# 成功
200 OK                    — GET, PUT, PATCH（有响应体）
201 Created               — POST（包含 Location 头）
204 No Content            — DELETE, PUT（无响应体）

# 客户端错误
400 Bad Request           — 验证失败、畸形 JSON
401 Unauthorized          — 缺少或无效认证
403 Forbidden             — 已认证但无权限
404 Not Found             — 资源不存在
409 Conflict              — 重复条目、状态冲突
422 Unprocessable Entity  — 语义无效（合法 JSON，数据错误）
429 Too Many Requests     — 超过限流

# 服务端错误
500 Internal Server Error — 意外失败（永远不暴露详情）
502 Bad Gateway           — 上游服务失败
503 Service Unavailable   — 临时过载，包含 Retry-After
```

### 常见错误

```
# ❌ 错误: 全部返回 200
{ "status": 200, "success": false, "error": "Not found" }

# ✅ 正确: 语义化使用 HTTP 状态码
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "User not found" } }

# ❌ 错误: 验证错误返回 500
# ✅ 正确: 400 或 422 并包含字段级详情

# ❌ 错误: 创建资源返回 200
# ✅ 正确: 201 并包含 Location 头
```

---

## 响应格式

### 成功响应

```json
{
  "data": {
    "id": "abc-123",
    "email": "alice@example.com",
    "name": "Alice",
    "created_at": "2025-01-15T10:30:00Z"
  }
}
```

### 集合响应（带分页）

```json
{
  "data": [
    { "id": "abc-123", "name": "Alice" },
    { "id": "def-456", "name": "Bob" }
  ],
  "meta": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  },
  "links": {
    "self": "/api/v1/users?page=1&per_page=20",
    "next": "/api/v1/users?page=2&per_page=20",
    "last": "/api/v1/users?page=8&per_page=20"
  }
}
```

### 错误响应

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "invalid_format"
      }
    ]
  }
}
```

---

## 分页

### Offset 分页（简单）

```
GET /api/v1/users?page=2&per_page=20
```

**适用：** 管理后台、小数据集（<10K）
**缺点：** 大偏移量时慢（OFFSET 100000）、并发插入时不一致

### Cursor 分页（可扩展）

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20
```

```json
{
  "data": [...],
  "meta": {
    "has_next": true,
    "next_cursor": "eyJpZCI6MTQzfQ"
  }
}
```

**适用：** 无限滚动、大数据集、公开 API
**优点：** 性能稳定、并发安全

### 选择建议

| 使用场景 | 分页类型 |
|----------|----------|
| 管理后台、小数据集 | Offset |
| 无限滚动、feeds | Cursor |
| 公开 API | Cursor（默认）+ Offset（可选） |
| 搜索结果 | Offset（用户期望页码） |

---

## 过滤、排序与搜索

### 过滤

```
# 简单相等
GET /api/v1/orders?status=active&customer_id=abc-123

# 比较运算符
GET /api/v1/products?price[gte]=10&price[lte]=100

# 多值（逗号分隔）
GET /api/v1/products?category=electronics,clothing
```

### 排序

```
# 单字段（负号表示降序）
GET /api/v1/products?sort=-created_at

# 多字段
GET /api/v1/products?sort=-featured,price,-created_at
```

### 全文搜索

```
GET /api/v1/products?q=wireless+headphones
```

---

## 认证与授权

### Token 认证

```
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# API Key（服务端到服务端）
GET /api/v1/data
X-API-Key: sk_live_abc123
```

### 授权模式

```typescript
// 资源级：检查所有权
app.get("/api/v1/orders/:id", async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order) return res.status(404).json({ error: { code: "not_found" } });
  if (order.userId !== req.user.id) return res.status(403).json({ error: { code: "forbidden" } });
  return res.json({ data: order });
});

// 角色级：检查权限
app.delete("/api/v1/users/:id", requireRole("admin"), async (req, res) => {
  await User.delete(req.params.id);
  return res.status(204).send();
});
```

---

## 限流

### 响应头

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

# 超限时
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

### 限流分层

| 层级 | 限制 | 窗口 | 适用场景 |
|------|------|------|----------|
| 匿名 | 30/分钟 | IP | 公开端点 |
| 认证用户 | 100/分钟 | 用户 | 标准 API |
| 高级 | 1000/分钟 | API Key | 付费 API |
| 内部 | 10000/分钟 | 服务 | 服务间调用 |

---

## 版本控制

### URL 路径版本（推荐）

```
/api/v1/users
/api/v2/users
```

### 版本策略

```
1. 从 /api/v1/ 开始——不到必要时不版本化
2. 最多维护 2 个活跃版本（当前 + 上一个）
3. 废弃时间线：
   - 公开 API 提前 6 个月通知
   - 添加 Sunset 头
   - 过期后返回 410 Gone
4. 非破坏性变更不需要新版本：
   - 响应中新增字段
   - 新增可选查询参数
   - 新增端点
5. 破坏性变更需要新版本：
   - 删除或重命名字段
   - 更改字段类型
   - 更改 URL 结构
```

---

## 实现模式

### TypeScript (Zod + Next.js)

```typescript
import { z } from "zod";
import { NextRequest, NextResponse } from "next/server";

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({
      error: {
        code: "validation_error",
        message: "Request validation failed",
        details: parsed.error.issues.map(i => ({
          field: i.path.join("."),
          message: i.message,
          code: i.code,
        })),
      },
    }, { status: 422 });
  }

  const user = await createUser(parsed.data);

  return NextResponse.json(
    { data: user },
    {
      status: 201,
      headers: { Location: `/api/v1/users/${user.id}` },
    },
  );
}
```

---

## 设计检查清单

发布新端点前：

- [ ] 资源 URL 符合命名规范（复数、kebab-case、无动词）
- [ ] 使用正确的 HTTP 方法
- [ ] 返回合适的状态码（不是全部 200）
- [ ] 输入通过 Schema 验证（Zod、Pydantic）
- [ ] 错误响应遵循标准格式（包含代码和消息）
- [ ] 列表端点实现了分页（cursor 或 offset）
- [ ] 需要认证（或明确标记为公开）
- [ ] 检查了授权（用户只能访问自己的资源）
- [ ] 配置了限流
- [ ] 响应不泄露内部细节（堆栈跟踪、SQL 错误）
- [ ] 与现有端点命名一致
- [ ] 已更新文档（OpenAPI/Swagger）

---

**记住**：API 是产品。好的 API 设计让开发者爱上你的产品，差的 API 让开发者逃离。
