---
来源: 稀土掘金
领域: 工程化
关键词: RESTful API, API设计, 最佳实践, HTTP, 版本控制, 错误处理, HATEOAS
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://juejin.cn/post/7470106567711490088
---

# RESTful API 设计最佳实践完整指南

> 来源：稀土掘金 - 【译】RESTful API 最佳实践
> 原文：[REST API Design Best Practices](https://tigerabrodi.blog/rest-api-design-best-practices)
> 整理日期：2026-04-19

---

## 一、核心原则

设计 API 应该遵循以下原则：

- **易读且易用** - API 应该直观、易于理解
- **难以误用** - 每一个接口都是明确的，无歧义
- **完整且简洁** - 不仅仅是项目的需求，还应该适当补全可能的需求，并且不失简洁

---

## 二、名称和 URL 结构

### 2.1 资源名称规范

| 规则 | 正确示例 | 错误示例 |
|------|----------|----------|
| 使用名词代替资源，而非动词 | `/items`, `/books` | `/createItems`, `/getBooks` |
| 使用复数名称表示集合 | `/books` | `/book` |
| 使用破折号提高可读性 | `/inventory-management` | `/inventory_management` |

### 2.2 URL 结构设计

#### 嵌套资源

为嵌套资源实现逻辑分组：

```
/customers/{id}/orders
```

#### 避免深层次嵌套

❌ 避免超过 2-3 层嵌套：
```
/users/123/posts/456/comments/789/likes
```

✅ 推荐做法：
```
/users/123/posts
/posts/456/comments
```

#### 避免暴露内部结构

避免在 URL 中透露数据库结构，以免泄露不必要的信息（例如按序生成的 ID）：

```
❌ /db/users/table/row/123
✅ /users/123
```

---

## 三、版本控制

### 3.1 版本策略

永远为接口提供版本开头的路由，以免在破坏性更新时增加处理逻辑的复杂性。

### 3.2 版本实现方式

| 方式 | 示例 | 说明 |
|------|------|------|
| **URL 路径版本** | `/v1/books`, `/v2/books` | 最常用，直观明了 |
| **查询参数版本** | `/books?version=2` | 可选方案 |
| **Header 版本** | `Accept: application/vnd.api.v2+json` | 较复杂，不推荐 |

---

## 四、分页设计

### 4.1 分页必要性

为大量的数据查询提供分页接口。

### 4.2 分页方式对比

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Offset/Limit** | 简单直观 | 数据变化时可能跳过或重复 | 数据量小、变化少 |
| **Cursor（光标）** | 高效、无重复 | 实现稍复杂 | 大数据量、实时数据 |

### 4.3 Cursor 分页示例

基于 Cursor 的分页机制，使用一个唯一的标识符（如 `lastItemId`）来标识上一次请求返回的最后一项数据。

```
GET /items?lastItemId=1000&limit=20
```

**优点**：
- 避免传统分页中可能遇到的跳过或重复数据的问题
- 如果数据在两次分页请求之间发生变化，使用光标分页可以确保每次都从上一项数据之后继续获取

---

## 五、筛选和排序

### 5.1 筛选

允许通过 URL 查询参数进行筛选：

```
/users?lastName=Smith&age=30
```

### 5.2 字段选择

支持动态选择数据的列，减少数据传输：

```
/products?fields=id,name,price
```

### 5.3 排序

使用清晰参数提供排序逻辑：

```
/posts?sort=+author,-datePublished
```

- `+` 表示升序
- `-` 表示降序

---

## 六、API 操作

### 6.1 幂等性（重点）

**定义**：多次相同的请求应产生相同的结果。

**重要性**：
- 对于 `DELETE` 和 `PUT` 操作尤为重要
- 对于敏感操作，可以考虑使用幂等性键

**示例**：Stripe 在收费操作中使用了幂等性键 `Idempotency-Key`

### 6.2 异步操作

对于耗时较长的操作：

1. 使用状态码 `202 Accepted` 表示操作已被接受但尚未完成
2. 提供状态端点用于跟踪进度：
   ```
   GET /orders/123/status
   ```
3. 在 `Location` 头部中包含状态端点的 URL
4. 考虑支持操作取消：
   ```
   DELETE /orders/123/cancel
   ```

### 6.3 部分响应

支持对大型资源的部分内容检索：

1. 客户端可以通过 `HEAD` 请求检查资源大小
2. 使用 `Range` 头部请求特定的分块

```
GET /videos/123.mp4
Range: bytes=0-1023
```

---

## 七、安全性

### 7.1 传输安全

- 使用 SSL/TLS 加密，通过 HTTPS 保护数据传输

### 7.2 认证与授权

- 采用适当的认证和授权机制，如 OAuth 2.0、JWT 等

### 7.3 速率限制

- 应用速率限制，以防止拒绝服务（DoS）攻击
- 返回相关头部信息：
  ```
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1640995200
  ```

### 7.4 错误信息处理

- 谨慎处理错误消息，避免泄露敏感信息
- 提供清晰的错误消息，说明问题所在以及下一步的操作

---

## 八、错误处理

### 8.1 HTTP 状态码使用

返回适当的 HTTP 状态码，以准确反映错误类型。

### 8.2 错误响应格式

提供清晰的错误消息，帮助开发者理解问题：

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The 'email' field must be a valid email address.",
    "details": {
      "field": "email",
      "value": "invalid-email"
    }
  }
}
```

### 8.3 成功空响应

对于成功的空响应，使用状态码 `204 No Content`。

---

## 九、应用级状态码 vs HTTP 状态码

### 9.1 分离关注点

| 层级 | 职责 |
|------|------|
| **HTTP 状态码** | 协议级别的错误处理（网络错误、未授权、服务器错误） |
| **应用级状态码** | 应用逻辑层的状态（操作成功、数据验证失败、权限不足） |

### 9.2 应用级状态码示例

```json
{
  "status": 200,
  "data": {
    "code": "USER_ACCOUNT_LOCKED",
    "message": "Your account has been temporarily locked due to multiple failed login attempts."
  }
}
```

### 9.3 好处

1. **增强灵活性** - 在 HTTP 协议层面保持标准化，同时在应用层提供更细粒度的控制
2. **清晰的错误处理** - 前端能更精确地响应不同的应用级别错误
3. **提升可扩展性** - 可以定义更加细化的业务逻辑处理流程
4. **用户体验优化** - 提供更细粒度的用户提示和引导
5. **便于国际化** - 根据应用级状态码返回本地化的错误信息
6. **可调试性** - 在日志中记录更具体的错误信息，有助于定位问题

---

## 十、文档

文档是重中之重，提供设计文档甚至优先于提供测试接口。

### 10.1 使用 OpenAPI（Swagger）

使用 OpenAPI 进行 API 文档编写。

### 10.2 文档内容应包括

- 端点结构
- 请求和响应格式
- 认证要求
- 错误代码和消息
- 示例代码

---

## 十一、HATEOAS（超媒体作为应用状态的引擎）

### 11.1 什么是 HATEOAS

HATEOAS (Hypermedia as the Engine of Application State) 是 RESTful API 的重要特性。客户端不仅能获取资源的数据，还能通过从资源响应中获得的超链接来进一步了解该资源及其操作。

### 11.2 响应示例

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "links": [
    {
      "rel": "self",
      "href": "/users/1"
    },
    {
      "rel": "update",
      "href": "/users/1",
      "method": "PUT"
    },
    {
      "rel": "delete",
      "href": "/users/1",
      "method": "DELETE"
    },
    {
      "rel": "friends",
      "href": "/users/1/friends"
    }
  ]
}
```

### 11.3 优点

- 客户端不需要依赖硬编码的 URL 结构
- API 变得更具自我描述性
- 客户端不需要经常查找文档

---

## 十二、总结

| 方面 | 最佳实践 |
|------|----------|
| **URL 设计** | 名词复数、破折号分隔、避免深层嵌套 |
| **版本控制** | URL 路径版本、多版本并存 |
| **分页** | 优先使用 Cursor 分页 |
| **筛选排序** | 查询参数、字段选择、清晰排序语法 |
| **安全性** | HTTPS、OAuth/JWT、速率限制 |
| **错误处理** | 标准 HTTP 状态码 + 应用级状态码 |
| **文档** | OpenAPI/Swagger、完整示例 |
| **HATEOAS** | 超媒体链接、自描述 API |

---

*本文档整理自稀土掘金翻译文章，用于团队 API 设计规范参考。*
