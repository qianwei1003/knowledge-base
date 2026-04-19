---
来源: 博客园
领域: AI编程
关键词: Cursor, AI规则, Rules, Commands, Skills, .cursorrules, 编程规范, 代码审查
质量: ⭐⭐⭐⭐⭐
提炼状态: 初提炼完成
提炼日期: 2026-04-19
提炼人: Claude
原始链接: https://www.cnblogs.com/ljbguanli/p/19748680
---

# 深度解析Cursor AI规则：从概念到实践，打造你的智能编程副驾驶

在人工智能驱动的编程新时代，Cursor作为一款强大的AI代码助手，正改变着开发者的工作流。其中，Rules（规则）功能是掌控AI行为、实现个性化与团队标准化的核心。本文将深入剖析Cursor Rules的设计哲学、工作机制与最佳实践。

---

## 核心要点速查

| 类别 | 规则 | 来源 |
|------|------|------|
| **Rules** | `.cursor/rules/*.mdc` - 定义代码标准和约束，自动应用于所有AI交互 | 章节一 |
| **Commands** | `.cursor/commands/*.md` - 手动触发的任务模板，封装具体执行步骤 | 章节一 |
| **Skills** | `.cursor/skills/*/SKILL.md` - 封装复杂可重用工作流程的技能包 | 章节一 |
| **核心区别** | Rules管风格约束，Commands管做什么，Skills管如何复杂地做 | 章节一 |
| **项目级规则** | `.cursor/rules/` 目录，随Git版本控制，团队协作基石 | 章节三、1 |
| **用户级规则** | 个人Cursor配置，管个人工作习惯 | 章节三、2 |
| **规则本质** | 隐藏的Prompt，自动拼接到系统提示前 | 章节四 |
| **编写原则** | 用自然语言（非配置语法）、明确范围、提供正面范例 | 章节五 |
| **黄金法则** | 像给聪明实习生写备忘录一样写规则，保持简洁聚焦 | 章节五 |

---

## 一、核心概念辨析：Rules、Commands与Skills

> 来源：章节一 - 核心概念辨析

Cursor三大配置文件的区别：

- **Rules** (`.cursor/rules/*.mdc`)：定义代码标准和约束的"宪法"。它们自动应用于所有AI代码生成与修改过程，是被动生效的全局策略。你可以将其理解为注入到每个AI思考过程中的上下文提示（prompt）。
- **Commands** (`.cursor/commands/*.md`)：可手动触发的"任务模板"。它们封装了具体的执行步骤，比如"重构这个函数"或"添加单元测试"，是主动调用的一次性指令。
- **Skills** (`.cursor/skills/*/SKILL.md`)：封装了更复杂、可重用工作流程的"技能包"。通常涉及多个步骤、条件判断甚至外部工具调用，用于处理如"搭建一个REST API端点"之类的复合任务。

简而言之，**Rules管"风格和约束"，Commands管"做什么"，Skills管"如何复杂地做"**。

## 二、Rules的价值与应用场景

> 来源：章节二 - Rules的价值与应用场景

Cursor Rules的本质，是将你的开发偏好、团队规范或项目要求，转化为AI能理解的"自然语言指令"，并在每次交互中自动执行。其主要价值体现在：

- **统一编码风格**：强制使用特定的命名规范（如驼峰命名）、缩进（2空格 vs 4空格）、引号类型（单引号 vs 双引号）。
- **定制文档与注释格式**：规定函数注释必须包含@param和@return，或要求为复杂逻辑添加行内解释。
- **强化团队协作与流程**：确保所有成员生成的代码遵循相同的项目结构、技术栈约束和目录规范。
- **实施个性化代码审查**：在AI生成代码的同时进行第一轮审查，要求避免使用某些已废弃的API，或强制进行错误处理。
- **设置全局偏好**：如默认响应语言、代码解释的详细程度等。

## 三、如何创建与管理Rules

> 来源：章节三 - 如何创建与管理Rules

### 1. Project Rules（项目级规则 - 强烈推荐）

> 来源：章节三、1 - Project Rules

作用范围：对整个项目生效，影响所有文件和所有AI提示。它非常稳定，且可随项目代码一起进行版本控制（Git），是团队协作的基石。

创建后，Cursor会自动在项目根目录下生成 `.cursor/rules/` 目录及对应的规则文件。

### 2. User Rules（用户级规则）

作用范围：仅对当前Cursor配置或会话生效。更偏向个人工作习惯。

**核心区别与选择建议：**
- User Rules：管"你这个人怎么用AI"。放个人偏好、通用风格。
- Project Rules：管"这个项目怎么写代码"。放项目结构、技术栈约束、团队共识。
- 优先级上，当两者冲突时，项目级规则通常更高。

## 四、Rules的底层机制

> 来源：章节四 - Rules的底层机制

Rules的本质是"隐藏的Prompt"。当你触发AI操作时，Cursor会在后台自动将当前激活的所有Rules内容作为系统提示拼接到你的原始请求之前：

```
最终发送给模型的 prompt =
  系统 prompt
+ 当前文件 / 选中代码
+ 你的自然语言指令
+ 所有匹配到的 rules（拼接成文本）
```

Rules的效果完全依赖于底层大语言模型的自然语言理解（NLP）能力。

## 五、最佳实践：如何编写高效、可执行的Rules

> 来源：章节五 - 最佳实践

### ⚠️ 常见误区1：写成"配置文件思维"

> 来源：章节五 - 常见误区1

Cursor不会像程序一样解析JSON或YAML的结构。需要用自然语言明确说明：

```
This is a Spring Boot project using MyBatis XML.
When adding a new database field:
- Update controller request/response DTOs
- Update service layer
- Update mapper interface
- Update mapper XML
- Do not modify unrelated files
```

### ⚠️ 常见误区2：指令模糊，缺乏上下文

过于笼统的指令会让AI困惑。"Always update all layers."中的"all"指哪些文件？规则必须明确触发条件和影响范围。

### ✅ 高效规则的黄金法则

1. **使用清晰的自然语言**：像给一个聪明的实习生写备忘录一样写规则
2. **列表化与结构化**：使用"-"或数字列表，让要点清晰
3. **明确范围**：例如，"当生成或修改Java Controller类时，必须……"
4. **提供正面范例**：给出"应该怎么做"的例子，比单纯说"不要怎么做"更有效
5. **保持简洁与聚焦**：一条规则解决一个问题。过于冗长的规则可能会被模型忽略部分内容

### 示例：代码审查规则

可用于指导AI在生成代码时进行自我审查，包括：
- 禁止代码中出现中文字符或符号
- 禁止继承MyBatis-Plus ServiceImpl/IService
- Wrapper仅限Repository层使用
- Mapper注入仅限Repository层
- 禁止直接使用RedisTemplate
- 禁止使用System.out/System.err
- Main方法仅存在于启动类
- 禁止直接打印堆栈跟踪
- 日志中禁止手动序列化对象
- 禁止注释掉的旧代码残留
- 敏感字段类必须使用@Sensitizable注解

### 示例：OpenAPI规范生成规则

```
## Project Conventions
### OpenSpec Authoring Conventions
- 语言策略:
  - 对话与生成文档: 使用中文
  - 代码与代码注释: 使用英文
- Requirement 描述格式:
  - 每条 Requirement 的描述行末尾需包含至少一个 MUST/SHALL 关键字
- Changes 目录命名规范:
  - 格式: `YYYY-mm-dd-{{Feature name}}`
  - 示例: `2026-01-27-user-export`
```

## 六、总结

> 来源：章节六 - 总结

Cursor Rules是一个强大而精巧的设计，它将项目规范与个人偏好通过NLP无缝注入AI的决策流程。关键三点：

1. 用自然语言，而非配置语法
2. 项目级规则用于团队协作，用户级规则用于个人习惯
3. Rules是"隐形的Prompt"，其效力取决于描述的清晰度

---

## 信息来源追溯

| 本文件内容 | 主要来源 | 辅助来源 |
|-----------|---------|---------|
| Rules/Commands/Skills 概念辨析 | 博客园文章 | - |
| Rules 价值与应用场景 | 博客园文章 | - |
| 创建与管理 Rules | 博客园文章 | - |
| 底层机制 | 博客园文章 | - |
| 最佳实践与黄金法则 | 博客园文章 | - |

---

## 原始来源

- [博客园 - 深度解析Cursor AI规则](https://www.cnblogs.com/ljbguanli/p/19748680)

---
*初提炼完成时间: 2026-04-19 | 提炼人: Claude*
*L1 检查清单: ✅ frontmatter完整 | ✅ 核心要点速查(9条) | ✅ 关键段落标注来源 | ✅ 来源追溯表*
