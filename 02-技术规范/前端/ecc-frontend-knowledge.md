# 前端开发相关知识汇总（everything-claude-code）

## 主要技术栈
- **主语言**: JavaScript
- **架构**: 混合模块化组织

## 代码规范
- 文件命名: camelCase
- 函数命名: camelCase
- 类命名: PascalCase
- 常量命名: SCREAMING_SNAKE_CASE
- 导入: 使用相对路径，如 `import { Button } from '../components/Button'`
- 导出: 支持多种风格，优先混合导出
- 推荐 *.test.js 测试文件命名模式

## 组件与测试
- 组件开发与测试推荐单元测试与集成测试（单独 test 文件，每个 *.test.js 负责对应功能或组件）
- 采用 80%+ 覆盖率标准（检测覆盖率配置已启用，需保持高代码可测性）

## 错误处理模式
- 推荐 `try-catch` 块处理错误，严格日志与用户友好错误上报

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('User-friendly message')
}
```

## 提交规范
- 遵循 Conventional Commits 格式（feat, fix, test, docs 等前缀）
- 提交说明语言精炼明确，首行为祈使句
- commit 例子：
  - `feat(rules): add C# language support`
  - `fix: sync catalog counts with filesystem (27 agents, 113 skills, 58 commands)`

## 常用工作流
- 标准 Feature 开发流程： 实现 → 测试 → 文档
- 详细的测试与验证步骤，强调不可跳过测试
- 常用配置文件有 package.json、eslint.config.js、tsconfig.json 等
