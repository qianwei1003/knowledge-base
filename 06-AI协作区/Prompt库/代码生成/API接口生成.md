---
created: 2026-04-18
tags: [#Prompt, #代码生成, #后端, #API]
model: Claude
effectiveness: ⭐⭐⭐⭐
---

# API接口生成

## 适用场景

- 快速生成 RESTful API 接口代码
- 根据需求生成后端接口骨架
- 生成符合团队规范的接口实现

## Prompt内容

```
你是一个精通 Node.js/Express 或 NestJS 的后端工程师。

请根据以下需求生成 RESTful API 接口：

【需求描述】
{需求内容}

【技术栈】
- 框架：{Express / NestJS}
- 数据库：{PostgreSQL / MySQL / MongoDB}
- ORM：{TypeORM / Prisma / Mongoose}

【约束条件】
1. 遵循 RESTful 设计规范
2. 输入参数校验完整
3. 统一的错误处理
4. 接口注释完整（描述、参数、返回值）
5. 包含分页、排序、过滤（如果是列表接口）
6. 返回格式统一：{ code, data, message }

【输出格式】
1. 路由定义
2. Controller 层代码
3. Service 层代码（核心逻辑）
4. DTO/类型定义
5. 接口文档（简要说明请求参数和返回格式）
```

## 使用示例

**输入：**
```
{需求内容}
用户管理接口，包含：创建用户、获取用户列表、获取单个用户、更新用户、删除用户。

{框架}
Express

{数据库}
PostgreSQL + Prisma
```

**输出：**
```typescript
// routes/user.ts
import { Router } from 'express';
import { UserController } from '../controllers/user.controller';
import { validate } from '../middleware/validate';
import { createUserDto, updateUserDto } from '../dto/user.dto';

const router = Router();
const userController = new UserController();

// POST /api/users - 创建用户
router.post('/', validate(createUserDto), userController.create);

// GET /api/users - 获取用户列表
router.get('/', userController.findAll);

// GET /api/users/:id - 获取单个用户
router.get('/:id', userController.findOne);

// PUT /api/users/:id - 更新用户
router.put('/:id', validate(updateUserDto), userController.update);

// DELETE /api/users/:id - 删除用户
router.delete('/:id', userController.remove);

export default router;
```

（后续还有 Controller、Service、DTO 等代码，此处省略）

## 效果评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | ⭐⭐⭐⭐ | 结构清晰，分层合理 |
| 规范遵守 | ⭐⭐⭐⭐⭐ | 严格遵循 RESTful |
| 可维护性 | ⭐⭐⭐⭐ | 注释完整，易于扩展 |
| 实用性 | ⭐⭐⭐⭐ | 可直接使用，稍作调整 |

**优点：**
- 一次性生成完整的 CRUD 接口
- 参数校验、错误处理都包含
- 自动生成接口文档

**缺点：**
- 数据库模型需要单独定义
- 复杂业务逻辑需要手动补充
- 认证鉴权逻辑需要额外处理

## 变体

| 变体 | 变化点 | 链接 |
|------|--------|------|
| API接口-带认证 | 包含 JWT 鉴权 | （待创建） |
| API接口-GraphQL | 生成 GraphQL Resolver | （待创建） |

## 相关 Prompt

- [[React组件生成]]
- [[测试用例生成]]
