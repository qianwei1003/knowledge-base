---
来源: OWASP 官方文档
原始链接: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
---

# REST API 安全最佳实践 (OWASP)

> 本文档整理自 OWASP REST Security Cheat Sheet，提供 RESTful API 安全开发的全面指南。

## 引言

REST（Representational State Transfer）是一种架构风格，由 Roy Fielding 在其博士论文中首次描述。REST API 的关键抽象是资源，由 URI 标识。

**REST API 的核心特性**：
- 无状态（Stateless）—— 有状态的 API 不符合 REST 架构风格
- 使用标准 HTTP 动词和错误码
- 支持 HATEOAS（Hypermedia As The Engine Of Application State）

---

## 1. HTTPS

### 强制要求

**安全 REST 服务必须仅提供 HTTPS 端点**。这可以：
- 保护传输中的认证凭据（密码、API 密钥、JWT）
- 允许客户端验证服务身份
- 保证传输数据的完整性

### 进阶建议

对于高权限 Web 服务，考虑使用**相互认证的客户端证书**提供额外保护。

---

## 2. 访问控制

### 核心原则

非公开 REST 服务必须在每个 API 端点执行访问控制。

### 现代架构最佳实践

1. **本地决策**：为了最小化延迟和减少服务间耦合，访问控制决策应由 REST 端点本地进行
2. **集中认证**：用户认证应集中在身份提供商（IdP），由其颁发访问令牌

---

## 3. JWT (JSON Web Tokens)

### 安全要求

**必须保护 JWT 完整性**：
- 使用签名或 MAC 保护 JWT 完整性
- 不允许不安全的 JWT：`{"alg":"none"}`
- **签名优于 MAC**：如果使用 MAC，每个能验证 JWT 的服务也能创建新 JWT

### 必须验证的标准声明

| 声明 | 说明 | 验证内容 |
|------|------|----------|
| `iss` | 发行者 | 是否为受信任的发行者？是否为签名密钥的预期所有者？ |
| `aud` | 受众 | 依赖方是否在此 JWT 的目标受众中？ |
| `exp` | 过期时间 | 当前时间是否在有效期结束之前？ |
| `nbf` | 生效时间 | 当前时间是否在有效期开始之后？ |

### JWT 黑名单

当显式会话终止事件发生时（如注销或空闲超时），应将关联 JWT 的摘要或哈希提交到 API 的黑名单，使该 JWT 在到期前失效。

---

## 4. API 密钥

### 用途

- 减少拒绝服务攻击的影响
- 组织将 API 货币化

### 最佳实践

- 为受保护端点的每个请求要求 API 密钥
- 如果请求过于频繁，返回 **429 Too Many Requests**
- 如果客户端违反使用协议，撤销 API 密钥
- **不要仅依赖 API 密钥保护敏感、关键或高价值资源**

---

## 5. 限制 HTTP 方法

### 安全要求

- 应用允许的 HTTP 方法白名单（如 GET、POST、PUT）
- 拒绝所有不匹配白名单的请求，返回 **405 Method not allowed**
- 确保调用者有权在资源集合、操作和记录上使用传入的 HTTP 方法

### Java EE 特别注意

参见：[Bypassing Web Authentication and Authorization with HTTP Verb Tampering](../assets/REST_Security_Cheat_Sheet_Bypassing_VBAAC_with_HTTP_Verb_Tampering.pdf)

---

## 6. 防止乱序 API 执行

### 问题

现代 REST API 通常通过端点序列实现业务工作流（例如：创建 → 验证 → 批准 → 完成）。如果后端不显式验证工作流状态转换，攻击者可能乱序调用端点以绕过预期控制。

### 攻击示例

```
预期序列：
POST /checkout/create
POST /checkout/pay
POST /checkout/confirm

攻击者直接调用：
POST /checkout/confirm  # 跳过支付步骤
```

### 防护指南

- 在服务器端为每个请求强制执行工作流状态验证
- 使用有限状态或状态机显式建模工作流
- 将令牌或标识符绑定到特定工作流阶段
- 避免依赖前端逻辑强制执行顺序
- 用清晰的错误响应拒绝无效或乱序转换

### 测试检查清单

- [ ] 端点是否可以乱序调用？
- [ ] 每个端点是否验证当前工作流状态？
- [ ] 令牌是否可跨工作流步骤重用？
- [ ] 无效状态转换是否一致被拒绝？

---

## 7. 输入验证

### 核心原则

**不要信任输入参数/对象**

### 验证要求

- 验证输入：长度、范围、格式和类型
- 通过使用强类型（数字、布尔值、日期、时间或固定数据范围）实现隐式输入验证
- 使用正则表达式约束字符串输入
- 拒绝意外/非法内容
- 使用验证/清理库或框架
- 定义适当的请求大小限制，超过限制返回 **413 Request Entity Too Large**
- 记录输入验证失败（假设每秒数百次失败验证的人在图谋不轨）
- 使用安全解析器解析传入消息（XML 需防止 XXE 攻击）

---

## 8. 验证内容类型

### 验证请求内容类型

- 拒绝包含意外或缺少内容类型头的请求，返回 **406 Unacceptable** 或 **415 Unsupported Media Type**
- 对于 Content-Length: 0 的请求，Content-type 头可选
- XML 内容类型确保适当的 XML 解析器加固
- 通过显式定义内容类型避免意外暴露

### 发送安全响应内容类型

- **不要**简单地将 Accept 头复制到响应的 Content-type 头
- 如果 Accept 头不包含允许类型之一，拒绝请求（理想情况返回 406 Not Acceptable）
- 确保响应中发送预期的内容类型头与正文内容匹配

---

## 9. 管理端点

### 安全要求

- 避免通过互联网暴露管理端点
- 如果必须通过互联网访问，确保用户必须使用强认证机制（如多因素）
- 通过不同 HTTP 端口或主机暴露管理端点，最好在不同 NIC 和受限子网上
- 通过防火墙规则或访问控制列表限制访问

---

## 10. 错误处理

### 最佳实践

- 使用通用错误消息响应 —— 避免不必要地透露失败细节
- 不要向客户端传递技术细节（如调用堆栈或其他内部提示）

---

## 11. 审计日志

### 要求

- 在安全相关事件之前和之后写入审计日志
- 考虑记录令牌验证错误以检测攻击
- 通过事先清理日志数据来防范日志注入攻击

---

## 12. 安全响应头

### API 响应必须包含的头

| 头 | 说明 |
|-----|------|
| `Cache-Control: no-store` | 指示不缓存包含敏感信息的响应 |
| `Content-Security-Policy: frame-ancestors 'none'` | 防止响应被框架嵌入，防范点击劫持 |
| `Content-Type` | 必须指定与 API 响应匹配的内容类型 |
| `Strict-Transport-Security` | 确保通过 HTTPS 访问 API |
| `X-Content-Type-Options: nosniff` | 防止浏览器进行 MIME 嗅探 |
| `X-Frame-Options: DENY` | 防止任何域框架响应（兼容旧浏览器） |

### 条件性头（响应可能渲染为 HTML 时）

| 头 | 示例 |
|-----|------|
| `Content-Security-Policy` | `Content-Security-Policy: default-src 'none'` |
| `Permissions-Policy` | 禁用浏览器功能权限 |
| `Referrer-Policy` | `Referrer-Policy: no-referrer` |

---

## 13. CORS (跨域资源共享)

### 配置建议

- 如果不支持/不预期跨域调用，禁用 CORS 头
- 设置跨域调用的来源时，尽可能具体，必要时足够通用

---

## 14. HTTP 请求中的敏感信息

### 核心原则

密码、安全令牌和 API 密钥不应出现在 URL 中，因为这可能被 Web 服务器日志捕获。

### 正确做法

- **POST/PUT 请求**：敏感数据应在请求正文或请求头中传输
- **GET 请求**：敏感数据应在 HTTP 头中传输

### 示例

✅ 正确：
```
https://example.com/resourceCollection/[ID]/action
https://twitter.com/vanderaj/lists
```

❌ 错误：
```
https://example.com/controller/123/action?apiKey=a53f435643de32
```

---

## 15. HTTP 返回码

### 安全相关状态码速查表

| 代码 | 消息 | 描述 |
|------|------|------|
| 200 | OK | 成功 REST API 操作响应 |
| 201 | Created | 请求已满足并创建资源，Location 头返回新资源 URI |
| 202 | Accepted | 请求已接受处理，但处理尚未完成 |
| 301 | Moved Permanently | 永久重定向 |
| 304 | Not Modified | 缓存相关响应，客户端与服务器有相同副本 |
| 307 | Temporary Redirect | 资源临时重定向 |
| 400 | Bad Request | 请求格式错误，如消息正文格式错误 |
| 401 | Unauthorized | 认证 ID/密码错误或未提供 |
| 403 | Forbidden | 认证成功但用户无权访问请求资源 |
| 404 | Not Found | 请求不存在的资源 |
| 405 | Method Not Acceptable | HTTP 方法错误 |
| 406 | Unacceptable | Accept 头中的内容类型不被支持 |
| 413 | Payload too large | 请求大小超过限制 |
| 415 | Unsupported Media Type | 请求内容类型不被支持 |
| 429 | Too Many Requests | 可能检测到 DOS 攻击或因速率限制拒绝请求 |
| 500 | Internal Server Error | 服务器遇到意外情况，不应透露内部信息 |
| 501 | Not Implemented | 服务尚未实现请求的操作 |
| 503 | Service Unavailable | 服务暂时无法处理请求，应通知客户端稍后重试 |

---

## 参考资料

- OWASP REST Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- Transport Layer Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
- JSON Web Token Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- OWASP Secure Headers Project: https://owasp.org/www-project-secure-headers/
