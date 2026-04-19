---
来源: GitHub 官方仓库
原始链接: https://github.com/goldbergyoni/nodebestpractices
---

# Node.js 最佳实践指南

> 本文档整理自 Node.js Best Practices 项目，包含 102+ 最佳实践，涵盖项目架构、错误处理、代码风格、测试、生产部署、安全、Docker 等多个方面。更新至 2024 年。

## 1. 项目架构实践

### 1.1 按业务组件结构化解决方案

**核心原则**：系统的根目录应包含代表合理大小业务模块的文件夹或仓库。每个组件代表一个产品领域（即有界上下文），如 'user-component'、'order-component' 等。每个组件都有自己的 API、逻辑和逻辑数据库。

**推荐结构**：
```
my-system
├─ apps (components)
│  ├─ orders
│  ├─ users
│  ├─ payments
├─ libraries (通用跨组件功能)
│  ├─ logger
│  ├─ authenticator
```

**好处**：使用自治组件，每个变更都在粒度更小、范围更小的范围内进行，心智负担、开发摩擦和部署恐惧都大大减少，开发者可以更快地移动。

### 1.2 分层组件，保持 Web 层在其边界内

**核心原则**：每个组件应包含"层"——存放共同关注点的专用文件夹：'entry-point'（控制器所在）、'domain'（逻辑所在）、'data-access'（数据访问）。

```
my-system
├─ apps (components)
│  └─ component-a
│     ├─ entry-points
│     │  ├─ api         # 控制器
│     │  └─ message-queue  # 消息消费者
│     ├─ domain         # 特性和流程：DTO、服务、逻辑
│     └─ data-access    # 数据库调用
```

### 1.3 将通用工具包装为包，考虑发布

将所有可重用模块放在专用文件夹（如 "libraries"），每个模块在自己的文件夹中，使其成为独立的包，拥有自己的 package.json 文件以增加模块封装性。

### 1.4 使用环境感知、安全和分层配置

**关键要求**：
- 键可从文件和环境变量读取
- 密钥保持在提交代码之外
- 配置分层以便查找
- 类型支持
- 验证以快速失败
- 为每个键指定默认值

**推荐包**：convict、env-var、zod

### 1.5 选择主框架时考虑所有后果

**2024 年推荐框架**：
- **Nest.js**：适合希望使用 OOP 和/或构建无法分割为更小自治组件的大规模应用的团队
- **Fastify**：适合围绕简单 Node.js 机制构建的合理大小组件（如微服务）的应用
- **Express**：成熟稳定，生态丰富
- **Koa**：轻量级，中间件模型优雅

### 1.6 节制且深思熟虑地使用 TypeScript

使用 TypeScript 定义变量和函数返回类型。但要谨慎使用，主要使用简单类型，只有在真正需要时才利用高级特性。

---

## 2. 错误处理实践

### 2.1 使用 Async-Await 或 Promises 处理异步错误

避免回调风格的"金字塔地狱"。使用 Promises 和 async-await 可以获得更紧凑和熟悉的代码语法，如 try-catch。

### 2.2 扩展内置 Error 对象

创建扩展内置 Error 对象的应用错误对象/类，在拒绝、抛出或发出错误时使用它。应用错误应添加有用的属性，如错误名称/代码和 isCatastrophic。

### 2.3 区分操作错误与程序员错误

**操作错误**：已知情况，错误影响完全理解并可周到处理（如 API 收到无效输入）。

**灾难性错误（程序员错误）**：指异常代码失败，需要优雅地重启应用程序。

### 2.4 集中处理错误，不要在中间件内处理

错误处理逻辑（如日志记录、决定是否崩溃、监控指标）应封装在专用的集中对象中，所有入口点（API、cron 作业、计划作业）在错误进入时调用它。

### 2.5 使用 OpenAPI 或 GraphQL 文档化 API 错误

让 API 调用者知道可能返回哪些错误，以便他们周到地处理而不崩溃。

### 2.6 当陌生人进城时优雅退出进程

当未知错误发生（灾难性错误）时，应用程序健康状况存在不确定性。在这种情况下，必须让错误可观察，关闭连接并退出进程。

### 2.7 使用成熟的日志记录器

使用 **Pino** 或 **Winston** 等健壮的日志工具，它们提供日志级别、美化打印着色等功能。

### 2.8 使用测试框架测试错误流程

确保代码不仅满足正面场景，还能正确处理和返回错误。

### 2.9 使用 APM 产品发现错误和停机

监控和性能产品可以自动突出显示您遗漏的错误、崩溃和慢速部分。

### 2.10 捕获未处理的 Promise 拒绝

注册 `process.unhandledRejection` 事件以捕获未处理的 Promise 拒绝。

### 2.11 使用专用库快速失败，验证参数

使用 **ajv**、**zod** 或 **typebox** 等现代验证库断言 API 输入。

### 2.12 返回前始终 await Promise 以避免部分堆栈跟踪

在返回 Promise 时始终使用 `return await`，以获得完整的错误堆栈跟踪。

### 2.13 订阅事件发射器和流的 'error' 事件

与典型函数不同，try-catch 子句不会捕获源自 Event Emitters 的错误。必须订阅事件发射器的 'error' 事件。

---

## 3. 代码风格实践

### 3.1 使用 ESLint

ESLint 是检查可能代码错误和修复代码风格的事实标准。

### 3.2 使用 Node.js ESLint 扩展插件

添加 Node.js 特定插件：`eslint-plugin-node`、`eslint-plugin-security`、`eslint-plugin-require`

### 3.3 代码块的大括号从同一行开始

```javascript
// 正确
function someFunction() {
  // 代码块
}

// 避免
function someFunction()
{
  // 代码块
}
```

### 3.4 正确分隔语句

使用 ESLint 获得分隔关注点的意识。Prettier 或 Standardjs 可自动解决这些问题。

### 3.5 命名函数

命名所有函数，包括闭包和回调。避免匿名函数。

### 3.6 使用命名约定

- **lowerCamelCase**：常量、变量和函数
- **UpperCamelCase**：类
- **UPPER_SNAKE_CASE**：全局或静态变量

### 3.7 优先使用 const，抛弃 var

使用 const 意味着一旦变量被赋值，就不能重新赋值。

### 3.8 首先在函数外 require 模块

在每个文件的开头，在任何函数之外 require 模块。

### 3.9 为模块/文件夹设置显式入口点

设置显式的根文件，导出公共和有趣的代码。

### 3.10 使用 === 运算符

优先使用严格相等运算符 `===`，而不是较弱抽象相等运算符 `==`。

### 3.11 使用 Async Await，避免回调

Async-await 是表达异步流程的最简单方式，它使异步代码看起来像同步代码。

### 3.12 使用箭头函数表达式

箭头函数使代码结构更紧凑，并保持根函数的词法上下文（即 this）。

### 3.13 避免函数外的副作用

避免在网络或数据库调用等副作用代码放在函数之外。

---

## 4. 测试和整体质量实践

### 4.1 至少编写 API（组件）测试

大多数项目没有任何自动化测试。优先从 API 测试开始，这是最简单的编写方式，提供比单元测试更多的覆盖。

### 4.2 每个测试名称包含 3 个部分

在测试名称中说明正在测试什么（被测单元）、在什么情况下、预期结果是什么。

### 4.3 按 AAA 模式结构化测试

- **Arrange**：准备测试环境
- **Act**：执行被测操作
- **Assert**：验证结果

### 4.4 确保 Node 版本统一

使用 `.nvmrc` 文件和 `engines` 字段确保团队使用相同版本。

### 4.5 避免全局测试夹具和种子

按测试添加数据，而不是使用全局夹具。

### 4.6 标记测试

使用标签组织测试（如 `@slow`、`@integration`）。

### 4.7 检查测试覆盖率

测试覆盖率有助于识别错误的测试模式。

### 4.8 使用生产环境进行端到端测试

确保测试环境与生产环境尽可能相似。

### 4.9 使用静态分析工具定期重构

使用 ESLint、TypeScript 等工具保持代码质量。

### 4.10 模拟外部 HTTP 服务响应

在测试中模拟外部 HTTP 服务以提高测试速度和可靠性。

### 4.11 隔离测试中间件

单独测试中间件，不与整个应用集成。

### 4.12 生产中指定端口，测试中随机化

在生产环境中使用固定端口，在测试中使用随机端口。

### 4.13 测试五种可能结果

- 同步返回值
- 同步抛出
- 异步解决
- 异步拒绝
- 调用回调

---

## 5. 生产部署实践

### 5.1 监控

使用 PM2、Docker 或 Kubernetes 等工具确保进程正常运行。

### 5.2 使用智能日志增加可观察性

使用 Pino 或 Winston 等成熟日志记录器。

### 5.3 将一切可能委托给反向代理

将 gzip、SSL 等委托给 Nginx 或其他反向代理。

### 5.4 锁定依赖

使用 `npm ci` 而不是 `npm install` 以确保一致的依赖版本。

### 5.5 使用正确工具保护进程运行时间

使用 PM2、Docker 或 Kubernetes 等进程管理器。

### 5.6 利用所有 CPU 核心

使用集群模式或容器编排利用多核 CPU。

### 5.7 创建"维护端点"

创建用于健康检查和维护的专用端点。

### 5.8 使用 APM 产品发现未知问题

APM 可以自动发现性能瓶颈和错误。

### 5.9 使代码准备好投入生产

确保代码具有适当的日志记录、错误处理、监控等。

### 5.10 测量和保护内存使用

监控内存使用并设置限制。

### 5.11 将前端资产移出 Node

使用 CDN 或静态文件服务器提供前端资产。

### 5.12 努力实现无状态

避免在应用内存中存储会话状态。

### 5.13 使用自动检测漏洞的工具

使用 `npm audit`、Snyk 等工具检测依赖漏洞。

### 5.14 为每个日志语句分配事务 ID

使用请求 ID 跟踪跨服务的请求流程。

### 5.15 设置 NODE_ENV=production

生产环境必须设置 NODE_ENV=production。

### 5.16 设计自动化、原子化和零停机部署

使用蓝绿部署、滚动更新等策略。

### 5.17 使用 Node.js LTS 版本

在生产中使用长期支持版本。

### 5.18 日志写入 stdout，避免在应用内指定日志目标

让基础设施处理日志路由。

### 5.19 使用 npm ci 安装包

使用 `npm ci` 确保一致的安装。

---

## 6. 安全实践

### 6.1 采用 Linter 安全规则

使用 `eslint-plugin-security` 等插件。

### 6.2 使用中间件限制并发请求

防止 DoS 攻击。

### 6.3 从配置文件中提取密钥

使用环境变量或密钥管理服务。

### 6.4 使用 ORM/ODM 库防止查询注入

使用参数化查询。

### 6.5 通用安全最佳实践集合

遵循 OWASP 等安全指南。

### 6.6 调整 HTTP 响应头以增强安全性

使用 Helmet 等中间件。

### 6.7 持续自动检查漏洞依赖

定期运行 `npm audit`。

### 6.8 使用 bcrypt 或 scrypt 保护用户密码

永远不要存储明文密码。

### 6.9 转义 HTML、JS 和 CSS 输出

防止 XSS 攻击。

### 6.10 验证传入的 JSON 模式

使用 JSON Schema 验证请求数据。

### 6.11 支持 JWT 黑名单

实现 JWT 撤销机制。

### 6.12 防止针对授权的暴力攻击

使用速率限制和账户锁定。

### 6.13 以非 root 用户运行 Node.js

降低安全风险。

### 6.14 使用反向代理或中间件限制负载大小

防止大负载攻击。

### 6.15 避免 JavaScript eval 语句

eval 可执行任意代码。

### 6.16 防止邪恶正则表达式

避免 ReDoS 攻击。

### 6.17 避免使用变量加载模块

防止任意代码执行。

### 6.18 在沙箱中运行不安全代码

隔离不受信任的代码。

### 6.19 使用子进程时格外小心

避免命令注入。

### 6.20 向客户端隐藏错误详情

不要暴露堆栈跟踪或内部错误。

### 6.21 为 npm 或 Yarn 配置 2FA

保护账户安全。

### 6.22 修改会话中间件设置

使用安全的会话配置。

### 6.23 显式设置进程崩溃时机以避免 DoS 攻击

正确处理未捕获异常。

### 6.24 防止不安全重定向

验证和清理重定向 URL。

### 6.25 避免将密钥发布到 npm 注册表

使用 .npmignore 和 .gitignore。

### 6.26 检查过时包

定期更新依赖。

### 6.27 使用 'node:' 协议导入内置模块

```javascript
import fs from 'node:fs';
```

---

## 7. Docker 最佳实践

### 7.1 使用多阶段构建获得更精简更安全的 Docker 镜像

```dockerfile
# 构建阶段
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### 7.2 使用 node 命令启动，避免 npm start

直接使用 `node server.js` 而不是 `npm start`。

### 7.3 让 Docker 运行时处理复制和运行时间

使用 Docker 的重启策略和编排工具。

### 7.4 使用 .dockerignore 防止泄露密钥

```
node_modules
npm-debug.log
Dockerfile
.dockerignore
.env
.git
```

### 7.5 生产前清理依赖

使用 `npm prune --production`。

### 7.6 智能优雅关闭

正确处理 SIGTERM 信号。

### 7.7 使用 Docker 和 v8 设置内存限制

```bash
docker run --memory=512m -e NODE_OPTIONS="--max-old-space-size=400" app
```

### 7.8 计划高效缓存

优化 Dockerfile 层顺序以利用缓存。

### 7.9 使用显式镜像引用，避免 latest 标签

使用 `node:20.10.0-alpine` 而不是 `node:latest`。

### 7.10 优先使用较小的 Docker 基础镜像

使用 Alpine 版本减小镜像大小。

### 7.11 清理构建时密钥

不要在 Docker 镜像层中留下密钥。

### 7.12 扫描镜像多层漏洞

使用 Trivy、Clair 等工具。

### 7.13 清理 NODE_MODULE 缓存

```dockerfile
RUN npm ci && npm cache clean --force
```

### 7.14 通用 Docker 实践

遵循 Docker 安全和最佳实践指南。

### 7.15 Lint 你的 Dockerfile

使用 hadolint 检查 Dockerfile 最佳实践。

---

## 参考资料

- 原始仓库：https://github.com/goldbergyoni/nodebestpractices
- 中文版：https://github.com/goldbergyoni/nodebestpractices/blob/master/README.chinese.md
- 示例应用：https://github.com/practicajs/practica
