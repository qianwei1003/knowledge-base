---
来源: GitHub 官方文档
原始链接: https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api
---

# GitHub REST API 最佳实践

使用 GitHub API 时应遵循以下最佳实践。

## 避免轮询

应订阅 webhook 事件而不是轮询 API 获取数据。这有助于保持集成在 API 速率限制之内。更多信息请参见 [Webhooks 文档](https://docs.github.com/en/webhooks)。

## 进行认证请求

认证请求比未认证请求有更高的主要速率限制。为避免超出速率限制，应进行认证请求。更多信息请参见 [REST API 速率限制](https://docs.github.com/en/rest/overview/rate-limits-for-the-rest-api)。

## 避免并发请求

为避免超出次要速率限制，应串行发送请求而不是并发。可以通过实现请求队列系统来实现。

## 在修改请求之间暂停

如果进行大量 `POST`、`PATCH`、`PUT` 或 `DELETE` 请求，在每个请求之间等待至少一秒。这有助于避免次要速率限制。

## 正确处理速率限制错误

如果收到速率限制错误，应根据以下指南暂时停止请求：

- 如果 `retry-after` 响应头存在，应在该秒数之后才重试请求
- 如果 `x-ratelimit-remaining` 头为 `0`，应在 `x-ratelimit-reset` 头指定的时间之后才能再次请求（UTC epoch 秒）
- 否则，等待至少一分钟再重试。如果由于次要速率限制继续失败，重试间隔应指数增加，并在特定次数后抛出错误

在速率限制期间继续请求可能导致集成被封禁。

## 随重定向

GitHub REST API 在适当情况下使用 HTTP 重定向。应假设任何请求可能产生重定向。接收 HTTP 重定向不是错误，应跟随重定向。

- `301` 状态码表示永久重定向。应向 `location` 头指定的 URL 重复请求，并更新代码使用此 URL
- `302` 或 `307` 状态码表示临时重定向。应向 `location` 头指定的 URL 重复请求，但不应更新代码

## 不要手动解析 URL

许多 API 端点在响应体中返回 URL 值字段。不应尝试解析这些 URL 或预测未来 URL 结构。GitHub 可能在未来更改 URL 结构，导致集成失效。

应查找包含所需信息的字段。例如，创建 issue 的端点返回 `html_url` 字段和 `number` 字段。如果需要 issue 编号，使用 `number` 字段而不是解析 `html_url`。

同样，不应尝试手动构建分页查询。应使用链接头确定可请求的结果页。更多信息请参见 [REST API 分页](https://docs.github.com/en/rest/guides/using-pagination-in-the-rest-api)。

## 适当使用条件请求

大多数端点返回 `etag` 头，许多端点返回 `last-modified` 头。可使用这些头值进行条件 `GET` 请求。如果响应未更改，将收到 `304 Not Modified` 响应。条件请求不计入主要速率限制（前提是正确授权）。

示例：使用 `if-none-match` 头

```shell
curl -H "Authorization: Bearer YOUR-TOKEN" https://api.github.com/meta --include --header 'if-none-match: "644b5b0155e6404a9cc4bd9d8b1ae730"'
```

示例：使用 `if-modified-since` 头

```shell
curl -H "Authorization: Bearer YOUR-TOKEN" https://api.github.com/repos/github/docs --include --header 'if-modified-since: Wed, 25 Oct 2023 19:17:59 GMT'
```

> **注意**：不安全方法（POST、PUT、PATCH、DELETE）的条件请求通常不被支持，除非端点文档另有说明。

## 不要忽略错误

不应忽略重复的 `4xx` 和 `5xx` 错误码。应确保正确与 API 交互。例如，端点请求字符串但传入数值会收到验证错误；访问未授权或不存在的端点会收到 `4xx` 错误。

故意忽略重复验证错误可能导致应用因滥用被暂停。

## 进一步阅读

- [使用 Webhooks 的最佳实践](https://docs.github.com/en/webhooks/using-webhooks/best-practices-for-using-webhooks)
- [创建 GitHub App 的最佳实践](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/best-practices-for-creating-a-github-app)