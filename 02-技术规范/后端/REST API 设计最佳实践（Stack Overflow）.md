---
来源: Stack Overflow Blog
原始链接: https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/
---

# REST API 设计最佳实践

REST API 是当今最常见的 Web 接口类型之一。它们允许包括浏览器应用在内的各种客户端通过 REST API 与服务通信。因此，正确设计 REST API 非常重要，以免日后遇到问题。我们需要考虑安全性、性能以及 API 消费者的易用性。

## 接收和响应 JSON

REST API 应接收 JSON 请求负载并响应 JSON。JSON 是传输数据的标准。几乎所有网络技术都能使用它：JavaScript 有内置方法通过 Fetch API 或其他 HTTP 客户端编码和解码 JSON。服务端技术有无需太多工作就能解码 JSON 的库。

应确保 REST API 应用响应 JSON 时客户端将其解释为 JSON。在请求后应将响应头的 Content-Type 设置为 application/json。

## 端点路径使用名词而非动词

不应在端点路径中使用动词。应使用代表正在检索或操作的实体的名词作为路径名。

HTTP 请求方法已经有动词：
- **GET** - 检索资源
- **POST** - 向服务器提交新数据
- **PUT** - 更新现有数据
- **DELETE** - 删除数据

这些动词对应 CRUD 操作。应创建类似以下的路由：
- `GET /articles/` - 获取文章列表
- `POST /articles/` - 添加新文章
- `PUT /articles/:id` - 更新指定 ID 的文章
- `DELETE /articles/:id` - 删除指定 ID 的文章

## 用复数名词命名集合

端点路径中的名词应使用复数形式，如 `/articles/` 而非 `/article/`。这让所有 API 用户清楚地理解这是一个资源集合。

## 层级对象的嵌套资源

如果一个对象可以包含另一个对象，应设计端点反映这种关系。例如获取文章评论，应将 `/comments` 路径附加到 `/articles` 路径末尾：

```
GET /articles/:articleId/comments
```

但嵌套不应超过两三层。更深层级应返回资源 URL：
```json
"author": "/users/:userId"
```

## 优雅处理错误并返回标准错误码

消除 API 用户遇到错误时的困惑，应优雅处理错误并返回指示错误类型的 HTTP 响应码。

常见错误 HTTP 状态码：
- **400 Bad Request** - 客户端输入验证失败
- **401 Unauthorized** - 用户未认证，无权访问资源
- **403 Forbidden** - 用户已认证但不允许访问
- **404 Not Found** - 资源未找到
- **500 Internal Server Error** - 通用服务器错误
- **502 Bad Gateway** - 上游服务器无效响应
- **503 Service Unavailable** - 服务器意外问题

错误码应附带消息，让维护者有足够信息排查问题，但攻击者无法利用错误内容进行攻击。

## 允许过滤、排序和分页

数据库可能非常大，不应一次返回所有数据。需要过滤、排序和分页方法减少返回数据量。

通过查询字符串参数实现：

```javascript
app.get('/employees', (req, res) => {
  const { firstName, lastName, age } = req.query;
  let results = [...employees];
  if (firstName) results = results.filter(r => r.firstName === firstName);
  if (lastName) results = results.filter(r => r.lastName === lastName);
  if (age) results = results.filter(r => +r.age === +age);
  res.json(results);
});
```

## 保持良好的安全实践

大多数 REST API 是公开的，因此安全性至关重要。基本安全原则：

1. **认证** - 应在 REST API 请求上正确认证
2. **授权** - 确认用户有权访问资源
3. **输入验证** - 验证所有输入数据
4. **速率限制** - 设置请求次数限制防止滥用
5. **HTTPS** - 所有通信都应加密

## 缓存数据以提高性能

可以缓存请求响应以提高性能。常用缓存方案：
- 服务端缓存（Redis、Memcached）
- HTTP 缓存头（Cache-Control、Expires）
- 条件请求（ETag、Last-Modified）

## API 版本控制

API 可能会更改，应版本化以便用户继续使用旧版本。常用方法：

1. **URL 路径版本** - `/v1/articles/` vs `/v2/articles/`
2. **查询参数版本** - `/articles/?version=1`
3. **请求头版本** - `Accept: application/vnd.myapi.v1+json`

版本控制让用户有时间迁移到新版本，同时保持向后兼容。

## 总结

遵循这些最佳实践可以让 REST API 易于理解、未来可扩展、安全且快速。关键是坚持行业公认约定，避免给 API 消费者带来困惑。