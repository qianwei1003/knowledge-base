# ECC Claude API 使用指南

> 来源：ECC `claude-api` 技能 | 提炼时间：2026-04-20

---

## 目录

- [激活条件](#激活条件)
- [模型选择](#模型选择)
- [Python SDK](#python-sdk)
- [TypeScript SDK](#typescript-sdk)
- [工具调用](#工具调用)
- [视觉能力](#视觉能力)
- [扩展思考](#扩展思考)
- [提示缓存](#提示缓存)
- [批量 API](#批量-api)
- [Claude Agent SDK](#claude-agent-sdk)
- [成本优化](#成本优化)
- [错误处理](#错误处理)

---

## 激活条件

- 构建调用 Claude API 的应用
- 代码引入了 `anthropic` (Python) 或 `@anthropic-ai/sdk` (TypeScript)
- 询问 Claude API 模式、工具调用、流式输出或视觉能力
- 使用 Claude Agent SDK 实现 agent 工作流
- 优化 API 成本、token 使用或延迟

---

## 模型选择

| 模型 | ID | 最佳用途 |
|------|-----|---------|
| Opus 4.6 | `claude-opus-4-6` | 复杂推理、架构、研究 |
| Sonnet 4.6 | `claude-sonnet-4-6` | 平衡编码、大多数开发任务 |
| Haiku 4.5 | `claude-haiku-4-5-20251001` | 快速响应、高容量、成本敏感 |

默认使用 Sonnet 4.6，除非任务需要深度推理（Opus）或速度/成本优化（Haiku）。

---

## Python SDK

### 安装

```bash
pip install anthropic
```

### 基础消息

```python
import anthropic

client = anthropic.Anthropic()  # 从环境变量读取 ANTHROPIC_API_KEY

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Explain async/await in Python"}
    ]
)
print(message.content[0].text)
```

### 流式输出

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a haiku about coding"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### 系统提示

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a senior Python developer. Be concise.",
    messages=[{"role": "user", "content": "Review this function"}]
)
```

---

## TypeScript SDK

### 安装

```bash
npm install @anthropic-ai/sdk
```

### 基础消息

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic(); // 从环境变量读取 ANTHROPIC_API_KEY

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Explain async/await in TypeScript" }
  ],
});
console.log(message.content[0].text);
```

### 流式输出

```typescript
const stream = client.messages.stream({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku" }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

---

## 工具调用

定义工具让 Claude 调用：

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["location"]
        }
    }
]

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in SF?"}]
)

# 处理工具调用响应
for block in message.content:
    if block.type == "tool_use":
        result = get_weather(**block.input)
        follow_up = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "What's the weather in SF?"},
                {"role": "assistant", "content": message.content},
                {"role": "user", "content": [
                    {"type": "tool_result", "tool_use_id": block.id, "content": str(result)}
                ]}
            ]
        )
```

---

## 视觉能力

发送图片进行分析：

```python
import base64

with open("diagram.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": image_data}},
            {"type": "text", "text": "Describe this diagram"}
        ]
    }]
)
```

---

## 扩展思考

用于复杂推理任务：

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000
    },
    messages=[{"role": "user", "content": "Solve this math problem step by step..."}]
)

for block in message.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Answer: {block.text}")
```

---

## 提示缓存

缓存大型系统提示或上下文以降低成本：

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {"type": "text", "text": large_system_prompt, "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": "Question about the cached context"}]
)

print(f"Cache read: {message.usage.cache_read_input_tokens}")
print(f"Cache creation: {message.usage.cache_creation_input_tokens}")
```

---

## 批量 API

以 50% 成本降低异步处理大量任务：

```python
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": f"request-{i}",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": prompt}]
            }
        }
        for i, prompt in enumerate(prompts)
    ]
)

# 轮询完成状态
while True:
    status = client.messages.batches.retrieve(batch.id)
    if status.processing_status == "ended":
        break
    time.sleep(30)

# 获取结果
for result in client.messages.batches.results(batch.id):
    print(result.result.message.content[0].text)
```

---

## Claude Agent SDK

构建多步骤 agent：

```python
import anthropic

tools = [{
    "name": "search_codebase",
    "description": "Search the codebase for relevant code",
    "input_schema": {
        "type": "object",
        "properties": {"query": {"type": "string"}},
        "required": ["query"]
    }
}]

client = anthropic.Anthropic()
messages = [{"role": "user", "content": "Review the auth module for security issues"}]

while True:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        tools=tools,
        messages=messages,
    )
    if response.stop_reason == "end_turn":
        break
    messages.append({"role": "assistant", "content": response.content})
    # ... 执行工具并追加 tool_result 消息
```

---

## 成本优化

| 策略 | 节省 | 使用场景 |
|------|------|----------|
| 提示缓存 | 缓存 token 最高 90% | 重复系统提示或上下文 |
| 批量 API | 50% | 非时间敏感的批量处理 |
| 用 Haiku 替代 Sonnet | ~75% | 简单任务、分类、提取 |
| 缩短 max_tokens | 可变 | 已知输出较短时 |
| 流式输出 | 无（价格相同） | 更好的 UX |

---

## 错误处理

```python
import time
from anthropic import APIError, RateLimitError, APIConnectionError

try:
    message = client.messages.create(...)
except RateLimitError:
    # 退避并重试
    time.sleep(60)
except APIConnectionError:
    # 网络问题，退避重试
    pass
except APIError as e:
    print(f"API error {e.status_code}: {e.message}")
```

---

**记住**：Claude API 是构建强大 AI 应用的核心。选择合适的模型、使用工具调用、缓存提示，可以显著降低成本。
