# AI 编程相关知识（everything-claude-code）

## AI 编程主题覆盖
- 智能体（Agent）添加与开发支持（coding 工作流、代码生成、AI 技能集成等）
- Prompt 工程和编码自动化可通过扩展/增加技能与代理实现
- 相关代码风格、架构和测试均统一纳入项目规范

## 技能与代码生成
- 新增 AI/技能/自动化开发统一放在 `skills/{skill-name}` 下
- 有独立的 SKILL.md 进行文档描述，包括触发条件、工作流、示例
- 支持脚本自动化和多平台兼容（如 .agents/skills/*/SKILL.md, .cursor/skills/*/SKILL.md）
- 标准开发流程：实现 → 编写测试 → 文档完善 → 审核

## 工作流配置要点
- 所有自动化、CI、测试等流程统一在 `.github/workflows/` 目录下以 YML 文件管理
- 强调规范、审查、多人协作和可维护
- 工作流涉及：CI，测试校验，发布自动化，月度指标等

