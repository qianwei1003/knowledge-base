# AI 行为规范

---
created: 2026-04-18
source: #来源/Claude
tags: [#技术/AI编程, #规范/行为]
---

> AI 编程时的行为准则，融合 SDD 规范驱动开发理念和大厂最佳实践。让 AI 从"自由发挥者"转变为"遵循规范的协作者"。

---

## 一、核心原则

### 1.1 SDD 驱动原则

> 来源：SDD规范驱动开发

| 阶段 | AI 行为要求 |
|-----|------------|
| **Proposal** | 明确理解变更目标，不盲目开始 |
| **Spec** | 遵循规范文档执行，不偏离需求 |
| **Design** | 提供符合架构设计的方案 |
| **Tasks** | 拆解为可执行的小任务 |

**执行准则：**
- 不清楚需求时不开始编码
- 需求变更必须先更新规范文档
- 每个产出都要标注对应的规范条款

### 1.2 一致性原则

> 来源：大厂编程规范

| 类型 | 阿里巴巴 | 腾讯 | Airbnb | Google |
|-----|---------|------|--------|--------|
| 变量命名 | camelCase | camelCase | camelCase | camelCase |
| 组件命名 | PascalCase | PascalCase | PascalCase | PascalCase |
| 常量命名 | UPPER_SNAKE | UPPER_SNAKE | UPPER_SNAKE | UPPER_SNAKE |
| CSS 类名 | kebab-case | kebab-case | kebab-case | kebab-case |

### 1.3 可追溯原则

- 每个代码变更必须有明确的变更理由
- 遵循 Git 提交信息规范（Conventional Commits）
- 变更内容必须可回滚、可验证

---

## 二、AI 编程通病预防

### 2.1 硬编码问题

> 来源：AI编程通病与陷阱

**预防规范：**

```markdown
## 硬编码预防规则

1. **配置值硬编码**
   - ❌ 禁止：`const MAX_COUNT = 100`
   - ✅ 必须：`const MAX_COUNT = process.env.MAX_COUNT`
   - 来源：大厂编程规范

2. **API 端点硬编码**
   - ❌ 禁止：`fetch('/api/users')`
   - ✅ 必须：`fetch(process.env.API_BASE_URL + '/users')`

3. **魔法数字**
   - ❌ 禁止：`for (let i = 0; i < 7; i++)`
   - ✅ 必须：`const DAYS_IN_WEEK = 7; for (let i = 0; i < DAYS_IN_WEEK; i++)`
```

### 2.2 边界处理问题

**预防规范：**

```markdown
## 边界处理规则

1. **空值检查**
   - 必须检查：null、undefined、空数组、空对象
   - 数组操作前检查长度
   - 对象属性访问前检查存在性

2. **类型安全**
   - 禁止使用 any（TypeScript 项目）
   - 函数参数和返回值必须有类型声明
   - 来源：Airbnb 规范

3. **错误处理**
   - 每个可能失败的调用必须有 try-catch
   - 错误必须有明确的错误信息
   - 关键错误必须记录日志
```

### 2.3 过度设计问题

**预防规范：**

```markdown
## 适度设计规则

1. **函数长度限制**
   - 函数不超过 20 行（来源：Airbnb 规范）
   - 超过 20 行必须拆分

2. **单一职责原则**
   - 一个函数只做一件事
   - 函数名必须是动词或动词短语

3. **避免过早抽象**
   - 不要为尚未确定的需求创建抽象层
   - 先让代码运行，再考虑重构
```

---

## 三、代码质量要求

### 3.1 命名规范

> 来源：大厂编程规范

| 场景 | 规范 | 示例 |
|-----|------|------|
| 变量 | camelCase | `userName`, `totalCount` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 函数 | camelCase，动词开头 | `getUserById`, `validateInput` |
| 类 | PascalCase | `UserService`, `LoginForm` |
| 组件 | PascalCase | `UserCard`, `HeaderNav` |
| CSS 类 | kebab-case | `user-card`, `btn-primary` |
| 文件 | 与内容一致 | `UserService.ts`, `user-card.css` |

### 3.2 注释规范

> 来源：腾讯规范

```markdown
## 注释规范

### 必须添加注释的情况
1. 文件顶部：模块说明、作者、创建日期
2. 函数定义：JSDoc 注释（参数、返回值、异常）
3. 复杂逻辑：行内注释解释"为什么"
4. 业务规则：注释业务逻辑的背景

### 注释模板
```typescript
/**
 * 获取用户信息
 * @param userId - 用户ID
 * @returns 用户信息对象
 * @throws {NotFoundError} 用户不存在时抛出
 */
async function getUserById(userId: string): Promise<User> {
  // 检查用户是否存在（避免沉默失败）
  const user = await db.findUser(userId);
  if (!user) {
    throw new NotFoundError(`User ${userId} not found`);
  }
  return user;
}
```
```

### 3.3 安全规范

> 来源：大厂编程规范-安全性原则

```markdown
## 安全检查清单

- [ ] 用户输入必须验证（类型、长度、格式）
- [ ] 禁止 innerHTML（防止 XSS）
- [ ] 敏感数据不硬编码
- [ ] SQL/命令拼接禁止（防止注入）
- [ ] 权限检查不遗漏
- [ ] 敏感信息日志脱敏
```

---

## 四、SDD 工作流执行

### 4.1 变更执行流程

```
1. 理解需求
   ├── 明确要解决的问题
   ├── 确认验收标准
   └── 检查是否有现有规范

2. 遵循 SDD 流程
   ├── Proposal → 定义变更目标
   ├── Spec → 确认功能需求
   ├── Design → 设计技术方案
   └── Tasks → 拆解执行任务

3. 代码执行
   ├── 按规范编写代码
   ├── 自我审查
   └── 标注规范出处
```

### 4.2 代码自审清单

> 来源：CodeReview 检查清单

```markdown
## 代码自审清单

### 一致性
- [ ] 命名符合项目规范
- [ ] 格式化通过（ESLint/Prettier）
- [ ] 注释完整

### 正确性
- [ ] 边界情况已处理
- [ ] 错误有明确提示
- [ ] 类型声明完整

### 安全性
- [ ] 输入已验证
- [ ] 无硬编码敏感信息
- [ ] 无安全漏洞

### 可维护性
- [ ] 函数简短（<20行）
- [ ] 单一职责
- [ ] 无重复代码
```

---

## 五、相关链接

- SDD规范驱动开发 - SDD 框架来源
- 大厂编程规范 - 规范来源
- Copilot配置指南 - 工具配置
- 前端规范-综合 - 前端特定规范

---

## 更新记录

| 日期 | 更新内容 |
|-----|---------|
| 2026-04-18 | 整合 SDD 理念和大厂规范，创建 AI 行为规范 |
