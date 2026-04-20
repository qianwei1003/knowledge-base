# ECC MCP 服务端模式

> 来源：ECC `mcp-server-patterns` 技能 | 提炼时间：2026-04-20

---

## 目录

- [何时使用](#何时使用)
- [核心概念](#核心概念)
- [连接模式](#连接模式)
- [最佳实践](#最佳实践)

---

## 何时使用

- 实现新的 MCP 服务器
- 添加工具或资源
- 选择 stdio vs HTTP
- 升级 SDK
- 调试 MCP 注册和传输问题

---

## 核心概念

| 概念 | 描述 | 注册方式 |
|------|------|----------|
| **Tools** | 模型可调用的动作（如搜索、运行命令） | `registerTool()` 或 `tool()` |
| **Resources** | 模型可获取的只读数据（如文件内容、API 响应） | `registerResource()` 或 `resource()` |
| **Prompts** | 可复用参数化提示模板（如 Claude Desktop 中的提示） | `registerPrompt()` |
| **Transport** | stdio（本地，如 Claude Desktop）；Streamable HTTP（远程，如 Cursor、云） | 传输层 |

---

## 连接模式

### 本地（stdio）

适用于 Claude Desktop 等本地客户端。创建 stdio 传输并传递给服务器 connect 方法。

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });
```

### 远程（Streamable HTTP）

适用于 Cursor、云或其他远程客户端。使用 Streamable HTTP（当前规范的单 MCP HTTP 端点）。

> 注意：SDK API 随版本变化。始终检查 [MCP 官方文档](https://modelcontextprotocol.io) 或 Context7 获取当前方法名和签名。

### 安装

```bash
npm install @modelcontextprotocol/sdk zod
```

---

## 最佳实践

- **Schema 优先**：为每个工具定义输入 schema；记录参数和返回形状
- **错误处理**：返回模型可解释的结构化错误或消息；避免原始堆栈跟踪
- **幂等性**：优先使工具幂等，以便重试安全
- **限流和成本**：对于调用外部 API 的工具，考虑限流和成本；在工具描述中说明
- **版本管理**：在 package.json 中固定 SDK 版本；升级时检查发布说明

---

**记住**：MCP 服务器使 AI 助手能够调用工具、读取资源和使用提示。保持传输层与业务逻辑分离。
