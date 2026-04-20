# ECC Agent 自检与调试

> 来源：ECC `agent-introspection-debugging` 技能 | 提炼时间：2026-04-20

---

## 目录

- [何时激活](#何时激活)
- [范围边界](#范围边界)
- [四阶段循环](#四阶段循环)
- [恢复启发式](#恢复启发式)
- [ECC 集成](#ecc-集成)
- [输出标准](#输出标准)

---

## 何时激活

- 最大工具调用 / 循环限制失败
- 重复重试无进展
- 上下文增长或提示漂移导致输出质量下降
- 文件系统或环境状态不匹配
- 工具失败可能是可恢复的

---

## 范围边界

**适用：**
- 在盲目重试前捕获失败状态
- 诊断常见 agent 特定失败模式
- 应用受限的恢复操作
- 生成结构化人类可读 debug 报告

**不适用：**
- 代码变更后的功能验证 → 使用 `verification-loop`
- 已有更窄 ECC 技能的框架特定调试
- 当前 harness 无法自动执行的运行时承诺

---

## 四阶段循环

### Phase 1: 失败捕获

重试前精确记录失败：

```markdown
## Failure Capture
- Session / task:
- Goal in progress:
- Error:
- Last successful step:
- Last failed tool / command:
- Repeated pattern seen:
- Environment assumptions to verify:
```

### Phase 2: 根因诊断

| 模式 | 可能原因 | 检查 |
|------|----------|------|
| 最大工具调用 / 重复同一命令 | 循环或无退出路径 | 检查最后 N 个工具调用是否有重复 |
| 上下文溢出 / 推理降级 | 无界笔记、重复计划、超大日志 | 检查近期上下文是否有重复和低信号 |
| `ECONNREFUSED` / 超时 | 服务不可用或端口错误 | 验证服务健康、URL、端口 |
| `429` / 配额耗尽 | 重试风暴或缺少退避 | 统计重复调用并检查重试间隔 |
| 写入后文件缺失 / 过时 diff | 竞态、错误 cwd 或分支漂移 | 重新检查路径、cwd、git status |
| "修复"后测试仍失败 | 错误的假设 | 隔离具体失败的测试并重新推导 bug |

诊断问题：
- 这是逻辑失败、状态失败、环境失败还是策略失败？
- Agent 是否丢失了真实目标并开始优化错误的子任务？
- 失败是确定性的还是瞬态的？
- 最小的可逆动作是什么，可以验证诊断？

### Phase 3: 受限恢复

用改变诊断范围的最小动作恢复：

**安全的恢复动作：**
- 停止重复重试并重述假设
- 修剪低信号上下文，只保留活跃目标、阻塞点和证据
- 重新检查实际文件系统 / 分支 / 进程状态
- 将任务缩小到一个失败命令、一个文件或一个测试
- 从推测性推理切换到直接观察
- 当失败高风险或外部阻塞时升级给人类

**恢复检查清单：**

```markdown
## Recovery Action
- Diagnosis chosen:
- Smallest action taken:
- Why this is safe:
- What evidence would prove the fix worked:
```

### Phase 4: 内省报告

以报告结束，使恢复对下一个 agent 或人类可读：

```markdown
## Agent Self-Debug Report
- Session / task:
- Failure:
- Root cause:
- Recovery action:
- Result: success | partial | blocked
- Token / time burn risk:
- Follow-up needed:
- Preventive change to encode later:
```

---

## 恢复启发式

按此顺序优先干预：

1. **用一句话重述真实目标**
2. **验证世界状态而非信任记忆**
3. **缩小失败范围**
4. **运行一个鉴别性检查**
5. **仅在那之后重试**

**坏模式：**
用稍有不同的措辞重试同一动作三次

**好模式：**
捕获失败 → 分类模式 → 运行一个直接检查 → 仅在检查支持时更改计划

---

## ECC 集成

- 恢复后如代码有变更，使用 `verification-loop`
- 失败模式值得转化为本能或后续技能时，使用 `continuous-learning-v2`
- 问题不是技术失败而是决策模糊时，使用 `council`
- 失败来自冲突的本地状态或仓库漂移时，使用 `workspace-surface-audit`

---

## 输出标准

当此技能激活时，不要仅以"我修复了它"结束。

始终提供：
- 失败模式
- 根因假设
- 恢复动作
- 证明情况现在更好或仍然被阻塞的证据

---

**记住**：系统化自调试是 AI agent 的核心能力。在升级给人类前，先用结构化方法诊断问题。
