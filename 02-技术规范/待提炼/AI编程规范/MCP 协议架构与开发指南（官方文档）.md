---
来源: 官方文档
领域: AI编程
关键词: MCP, Model Context Protocol, AI Agent, 工具调用, 协议架构, JSON-RPC, 上下文管理, 标准化
质量: ⭐⭐⭐⭐⭐
提炼状态: 初提炼完成
提炼日期: 2026-04-19
提炼人: Claude
原始链接: https://modelcontextprotocol.io/docs/learn/architecture
---

# MCP 协议架构与开发指南

> 本文整理自 MCP 官方文档，系统介绍 Model Context Protocol 的架构设计、核心概念和开发实践。

---

## 核心要点速查

| 类别 | 规则 | 来源 |
|------|------|------|
| **MCP 定义** | AI 应用的 USB-C 接口，标准化连接外部系统 | 章节一 |
| **架构角色** | Host(宿主) → Client(客户端) → Server(服务端) | 章节四、4.1 |
| **传输方式** | STDIO(本地进程) / Streamable HTTP(远程) | 章节四、4.2 |
| **数据层协议** | 基于 JSON-RPC，含生命周期管理、原语、通知 | 章节四、4.2 |
| **服务器原语** | Tools(函数)、Resources(数据源)、Prompts(模板) | 章节五、5.2 |
| **客户端原语** | Sampling(LLM补全)、Elicitation(用户输入)、Logging(日志) | 章节五、5.2 |
| **生命周期** | 协议版本协商 → 能力发现 → 身份交换 | 章节五、5.1 |
| **能力协商** | 初始化握手时声明 capabilities，避免不支持调用 | 章节八 |
| **生态优势** | M+N 次实现代替 M×N，一次对接生态共享 | 章节八 |

---

## 一、MCP 是什么？

MCP（Model Context Protocol）是一个开源标准协议，用于连接 AI 应用与外部系统。

使用 MCP，AI 应用（如 Claude、ChatGPT）可以连接数据源（本地文件、数据库）、工具（搜索引擎、计算器）和工作流（专业提示模板），从而获取关键信息并执行任务。

**类比**：MCP 就像 AI 应用的 USB-C 接口。正如 USB-C 提供了连接电子设备的标准方式，MCP 提供了连接 AI 应用与外部系统的标准方式。

## 二、MCP 能实现什么？

- **个性化助手**：Agent 可访问 Google Calendar 和 Notion，成为更个性化的 AI 助手
- **设计到代码**：Claude Code 可基于 Figma 设计生成完整的 Web 应用
- **企业数据分析**：企业聊天机器人可连接组织内多个数据库，用对话分析数据
- **3D 创作**：AI 模型可在 Blender 上创建 3D 设计并使用 3D 打印机输出

## 三、为什么 MCP 重要？

- **开发者**：MCP 减少构建或集成 AI 应用/Agent 的开发时间和复杂度
- **AI 应用/Agent**：MCP 提供对数据源、工具和应用生态的访问，增强能力和用户体验
- **最终用户**：MCP 使 AI 应用更强大，能访问数据并代表用户执行操作

## 四、核心架构

> 来源：章节四 - 核心架构

### 4.1 参与者（Participants）

> 来源：章节四、4.1 - 参与者

MCP 采用客户端-服务器架构：

- **MCP Host（宿主）**：AI 应用（如 Claude Code、Claude Desktop），协调和管理一个或多个 MCP Client
- **MCP Client（客户端）**：与 MCP Server 保持连接的组件，从 Server 获取上下文供 Host 使用
- **MCP Server（服务端）**：向 MCP Client 提供上下文的程序

每个 MCP Client 与对应的 MCP Server 维持专用连接。使用 STDIO 传输的本地 MCP Server 通常服务单个 Client，而使用 Streamable HTTP 传输的远程 MCP Server 通常服务多个 Client。

**示例**：Visual Studio Code 作为 MCP Host，连接 Sentry MCP Server 时实例化一个 MCP Client 对象维持该连接；再连接本地文件系统 Server 时，再实例化一个额外的 MCP Client 对象。

### 4.2 分层设计

> 来源：章节四、4.2 - 分层设计

MCP 由两层组成：

#### 数据层（Data Layer）

定义基于 JSON-RPC 的客户端-服务器通信协议，包括：

- **生命周期管理**：处理连接初始化、能力协商和连接终止
- **服务器特性**：提供工具（AI 行为）、资源（上下文数据）、提示（交互模板）
- **客户端特性**：允许服务器请求 Host LLM 采样、获取用户输入、记录日志
- **工具特性**：支持实时更新通知和长时间操作的进度跟踪

#### 传输层（Transport Layer）

管理客户端与服务器之间的通信通道和认证：

- **STDIO 传输**：使用标准输入/输出流进行本地进程间通信，无网络开销，性能最优
- **Streamable HTTP 传输**：使用 HTTP POST 发送客户端消息，可选 Server-Sent Events 实现流式传输。支持远程通信和标准 HTTP 认证（Bearer Token、API Key、自定义 Headers）。推荐使用 OAuth 获取认证 Token

## 五、数据层协议详解

> 来源：章节五 - 数据层协议详解

### 5.1 生命周期管理

> 来源：章节五、5.1 - 生命周期管理

MCP 是有状态协议，需要生命周期管理。目的是协商客户端和服务器支持的能力（Capabilities），包括：

1. **协议版本协商**：`protocolVersion` 字段确保双方使用兼容的协议版本
2. **能力发现**：`capabilities` 对象声明各方支持的功能（tools、resources、prompts 等）
3. **身份交换**：`clientInfo` 和 `serverInfo` 提供标识和版本信息

初始化流程示例：

```json
// Client → Server: Initialize Request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "elicitation": {}
    },
    "clientInfo": {
      "name": "example-client",
      "version": "1.0.0"
    }
  }
}

// Server → Client: Initialize Response
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "tools": {"listChanged": true},
      "resources": {}
    },
    "serverInfo": {
      "name": "example-server",
      "version": "1.0.0"
    }
  }
}
```

### 5.2 原语（Primitives）

> 来源：章节五、5.2 - 原语

MCP 定义了三种核心服务器原语：

| 原语 | 说明 | 示例 |
|------|------|------|
| **Tools** | AI 应用可调用的可执行函数 | 文件操作、API 调用、数据库查询 |
| **Resources** | 为 AI 应用提供上下文信息的数据源 | 文件内容、数据库记录、API 响应 |
| **Prompts** | 帮助构建与语言模型交互的可复用模板 | 系统提示、少样本示例 |

每种原语都有对应的发现方法（`*/list`）、检索方法（`*/get`）和执行方法（`tools/call`）。

**客户端原语**：

| 原语 | 说明 |
|------|------|
| **Sampling** | 允许服务器请求客户端 AI 应用的 LLM 补全 |
| **Elicitation** | 允许服务器向用户请求额外信息或确认 |
| **Logging** | 允许服务器向客户端发送日志消息 |

### 5.3 通知机制

协议支持实时通知，当服务器可用工具变化时（如新增或修改工具），服务器可发送工具更新通知。通知作为 JSON-RPC 2.0 通知消息发送（不期望响应），实现实时更新。

## 六、MCP 生态支持

MCP 是开放协议，获得广泛生态支持：

- **AI 助手**：Claude、ChatGPT
- **开发工具**：Visual Studio Code、Cursor、MCPJam
- **传输协议**：STDIO（本地）、Streamable HTTP（远程）

## 七、开发入门

### 构建 MCP Server

MCP Server 暴露数据源和工具，供 AI 应用使用。

### 构建 MCP Client

开发连接到 MCP Server 的应用程序。

### 构建 MCP App

构建在 AI 客户端内运行的交互式应用。

## 八、关键设计理念

> 来源：章节八 - 关键设计理念

1. **协议专注**：MCP 仅关注上下文交换的协议，不规定 AI 应用如何使用 LLM 或管理上下文
2. **解耦设计**：工具提供方与 AI 应用通过标准化协议解耦，类似 Web 开发的前后端分离
3. **一次对接、生态共享**：M 个 AI 应用 + N 个工具，只需 M + N 次实现，而非 M × N
4. **能力协商**：通过初始化握手协商双方支持的功能，避免不支持的调用
5. **动态发现**：工具列表支持动态更新和通知机制

---

## 信息来源追溯

| 本文件内容 | 主要来源 | 辅助来源 |
|-----------|---------|---------|
| MCP 概述与应用场景 | MCP 官方文档 | - |
| 核心架构设计 | MCP 官方文档 | - |
| 数据层协议详解 | MCP 官方文档 | - |
| 关键设计理念 | MCP 官方文档 | - |

---

## 原始来源

- [MCP Official Documentation - Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [MCP Official Website](https://modelcontextprotocol.io/)

---
*初提炼完成时间: 2026-04-19 | 提炼人: Claude*
*L1 检查清单: ✅ frontmatter完整 | ✅ 核心要点速查(9条) | ✅ 关键段落标注来源 | ✅ 来源追溯表*
*原始收集时间: 2026-04-19*
