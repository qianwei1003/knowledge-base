# Agent 智能体配置与使用（everything-claude-code）

## 核心知识
- 智能体类型包括代码审查、构建修复、自动化测试、文档等
- 新增 agent 做法：
  1. 在 agents/{agent-name}.md 下新建 agent 描述文件
  2. 在 AGENTS.md 完成注册
  3. 可选：更新 README.md、docs/COMMAND-AGENT-MAP.md，补充说明

## Agent 标准开发与注册流程
- agent 描述应涵盖用途、触发场景、适配脚本/命令
- 支持同步 catalog 机制（agents、skills、commands）
- 工作流支持多平台适配与扩展
- 推荐以 markdown 方式完整描述 agent 配置和用法，例：
  - 注册流程：新建 agents/{name}.md → AGENTS.md 登记
  - 目录/文件结构：“agents/*.md”, “docs/COMMAND-AGENT-MAP.md”

## 典型 Agent 相关文件与维护流程
- 相关 agent 配置与实现按模块和文档分布并有同步机制（如 AGENTS.md 自动校对新增/删除后的统计）
- 支持多人协作、文档先行、与技能集成紧密
- 大型提交建议分步骤详细列明 agent 增删改同步过程
