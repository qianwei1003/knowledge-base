---
来源: 博客园
领域: AI编程
关键词: OpenAI, Codex CLI, AI编程, 终端工具, MCP, 命令行, Agent, 沙箱
质量: ⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://www.cnblogs.com/ljbguanli/p/19439518
---

# OpenAI Codex CLI 完全指南：AI 编程助手的终端革命

> 来源：博客园 | 作者：ljbguanli | 发布时间：2026-01-04
> 原文链接：https://www.cnblogs.com/ljbguanli/p/19439518

## 一、什么是 Codex CLI？

Codex CLI 是 OpenAI 开发的本地运行编码代理工具，结合 ChatGPT 级别推理能力和本地代码执行能力。

### 核心特点

| 特性 | 说明 |
|------|------|
| 本地运行 | 直接在电脑上运行，保护代码隐私 |
| 开源免费 | Apache 2.0 许可，使用 Rust 构建 |
| 跨平台 | macOS、Linux、Windows（WSL2） |
| MCP 支持 | 可连接 Model Context Protocol 服务器 |
| 多种模式 | suggest、auto-edit、full-auto 三种操作模式 |

## 二、安装要求

- **操作系统**：macOS、Linux（Windows 需通过 WSL2）
- **Node.js**：v22 或更高版本
- **网络**：需要访问 OpenAI API

```bash
node --version  # 输出应 >= v22.0.0
```

## 三、安装方法

### 方式一：npm 全局安装（推荐）
```bash
npm install -g @openai/codex
```

### 方式二：Homebrew 安装（macOS/Linux）
```bash
brew install --cask codex
```

### 方式三：下载二进制文件
访问 [GitHub Releases](https://github.com/openai/codex/releases/latest) 下载：
- macOS (Apple Silicon): `codex-aarch64-apple-darwin.tar.gz`
- macOS (Intel): `codex-x86_64-apple-darwin.tar.gz`
- Linux (x86_64): `codex-x86_64-unknown-linux-musl.tar.gz`

### 验证安装
```bash
codex --version
```

## 四、认证配置

### 方式一：ChatGPT 账号登录（推荐）
运行 `codex` 后选择 Sign in with ChatGPT，支持：
- ✅ ChatGPT Plus（获 5 美元免费 API 额度）
- ✅ ChatGPT Pro（获 50 美元免费 API 额度，30天有效）
- ✅ ChatGPT Team / Edu / Enterprise

### 方式二：API Key 配置
```bash
# Linux/macOS
export OPENAI_API_KEY="sk-your-api-key-here"
# Windows PowerShell
$env:OPENAI_API_KEY="sk-your-api-key-here"
```

## 五、基本使用

### 启动交互模式
```bash
codex
```

### 直接提问模式
```bash
codex "解释这个项目的结构"
codex "找出所有的 TODO 注释"
codex "帮我写一个单元测试"
```

### 三种操作模式

| 模式 | 说明 | 自主程度 |
|------|------|----------|
| **suggest** | 只提供建议不执行 | 最安全 |
| **auto-edit** | 自动编辑，但执行需确认 | 平衡 |
| **full-auto** | 全自动模式，自动执行所有操作 | 最快 |

```bash
codex -a suggest "重构这个函数"
codex -a auto-edit "添加错误处理"
codex -a full-auto "运行测试并修复错误"
```

## 六、常用命令和参数

```bash
codex [options] [prompt]
# 选项：
#   -m, --model <model>         指定使用的模型
#   -a, --approval-mode <mode>  设置操作模式 (suggest/auto-edit/full-auto)
#   -q, --quiet                 安静模式
#   --profile <name>            使用指定的配置文件
```

### 实用示例

```bash
# 代码审查
codex "审查最近的代码更改并提供改进建议"
# 生成文档
codex "为这个项目生成 README 文档"
# Debug 帮助
codex "这个函数为什么会返回 null？"
# 重构代码
codex "将这个类重构为更小的模块"
```

## 七、配置文件

配置存储在 `~/.codex/config.toml`：

```toml
# 默认模型
model = "o4-mini"
# 默认操作模式
approval_mode = "auto-edit"

# 启用 MCP 服务器
[mcp_servers]
github = { command = "npx", args = ["-y", "@modelcontextprotocol/server-github"] }
```

## 八、MCP 服务器集成

Codex 支持 Model Context Protocol (MCP)，可连接外部工具：

```toml
[mcp_servers]
# GitHub 集成
github = { command = "npx", args = ["-y", "@modelcontextprotocol/server-github"] }
# 数据库集成
postgres = { command = "npx", args = ["-y", "@modelcontextprotocol/server-postgres"] }
```

## 九、IDE 集成

- **VS Code**：[安装 Codex 扩展](https://developers.openai.com/codex/ide)
- **Cursor**：内置支持
- **Windsurf**：内置支持

## 十、常见问题

| 问题 | 解决方案 |
|------|----------|
| `command not found` | 确保 npm 全局安装目录在 PATH 中 |
| Node 版本过低 | `nvm install 22 && nvm use 22` |
| API Key 无效 | `echo $OPENAI_API_KEY` 检查 |
| Homebrew 升级失败 | `brew uninstall --cask codex && brew install --cask codex` |

## 十一、最新动态

2025 年 12 月推出 **GPT-5.1-Codex-Max**（专门为编码任务优化）：
- ✅ 更快的响应速度
- ✅ 更好的代码理解能力
- ✅ 首个支持 Windows 环境训练的模型

## 快速开始三步走

1. **安装**：`npm install -g @openai/codex`
2. **配置**：登录 ChatGPT 账号或设置 API Key
3. **使用**：运行 `codex` 开始对话

## 参考资源

- [Codex CLI 官方文档](https://developers.openai.com/codex/cli/)
- [GitHub 仓库](https://github.com/openai/codex)
- [OpenAI Codex 介绍](https://openai.com/codex/)

---

## 关键启示

1. **三种操作模式**是安全与效率的平衡设计：suggest 最安全，full-auto 最快
2. **MCP 协议支持**使其可扩展连接 GitHub、数据库等外部工具
3. **开源 + Rust 构建**确保性能和隐私安全
4. **ChatGPT 账号直接登录**降低了使用门槛（Plus/Pro 用户有免费额度）
