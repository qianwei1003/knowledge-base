# ECC 质量保障体系

> 来源：ECC `eval-harness` + `tdd-workflow` + `verification-loop` 技能 | 提炼时间：2026-04-20

---

## 目录

- [Eval 驱动开发（EDD）](#eval-驱动开发edd)
- [TDD 工作流](#tdd-工作流)
- [验证循环](#验证循环)

---

# Eval 驱动开发（EDD）

## 核心理念

Eval-Driven Development 将 evals 视为"AI 开发的单元测试"：
- 在实现之前定义期望行为
- 开发期间持续运行 evals
- 每次变更跟踪回归
- 使用 pass@k 指标衡量可靠性

## Eval 类型

### 能力 Eval
测试 Claude 能否做到之前做不到的事：
```markdown
[CAPABILITY EVAL: feature-name]
Task: Description of what Claude should accomplish
Success Criteria:
  - [ ] Criterion 1
  - [ ] Criterion 2
Expected Output: Description of expected result
```

### 回归 Eval
确保变更不会破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
Baseline: SHA or checkpoint name
Tests:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
Result: X/Y passed (previously Y/Y)
```

## Grader 类型

### 1. 代码 Grader（确定性）
```bash
# 检查文件是否包含预期模式
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# 检查测试是否通过
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"
```

### 2. 模型 Grader（开放式）
```markdown
[MODEL GRADER PROMPT]
Evaluate the following code change:
1. Does it solve the stated problem?
2. Is it well-structured?
3. Are edge cases handled?

Score: 1-5
Reasoning: [explanation]
```

## 指标

### pass@k
"k 次尝试中至少一次成功"
- pass@1: 首次成功率
- pass@3: 3 次内成功率
- 目标：pass@3 > 90%

### pass^k
"k 次试验全部成功"
- 更严格
- 关键路径用 pass^3

## 工作流

### 1. 定义（编码前）
```markdown
## EVAL DEFINITION: feature-xyz
Capability Evals:
- [ ] Can create new user account
- [ ] Can validate email format
Regression Evals:
- [ ] Existing login still works
Success Metrics:
- pass@3 > 90% for capability evals
```

### 2. 实现
编写代码通过定义的 evals。

### 3. 评估
```bash
npm test
```

### 4. 报告
```markdown
EVAL REPORT: feature-xyz
Capability: 5/5 passed (pass@3: 100%)
Regression: 3/3 passed (pass^3: 100%)
Status: READY FOR REVIEW
```

---

# TDD 工作流

## 核心原则

1. **测试先于代码** - 始终 TDD
2. **覆盖率要求** - 最低 80%（单元 + 集成 + E2E）
3. **测试类型** - 单元测试、集成测试、E2E 测试

## 工作流步骤

### Step 1: 写用户旅程
```
As a user, I want to search for markets semantically,
so that I can find relevant markets even without exact keywords.
```

### Step 2: 生成测试用例
```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => { })
  it('handles empty query gracefully', async () => { })
  it('falls back to substring search when Redis unavailable', async () => { })
})
```

### Step 3: 运行测试（应该失败）
```bash
npm test
# 测试应该失败 - 还没实现
```

### Step 4: 实现代码
编写最小代码让测试通过。

### Step 5: 再次运行测试
```bash
npm test
# 测试现在应该通过
```

### Step 6: 重构
保持测试绿色，改善代码质量。

### Step 7: 验证覆盖率
```bash
npm run test:coverage
# 验证达到 80%+ 覆盖率
```

## 测试模式

### 单元测试（Jest/Vitest）
```typescript
import { render, screen, fireEvent } from '@testing-library/react'

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)
    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### API 集成测试
```typescript
describe('GET /api/markets', () => {
  it('returns markets successfully', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    expect(response.status).toBe(200)
  })
})
```

### E2E 测试（Playwright）
```typescript
test('user can search and filter markets', async ({ page }) => {
  await page.goto('/')
  await page.fill('input[placeholder="Search markets"]', 'election')
  await page.waitForTimeout(600)
  const results = page.locator('[data-testid="market-card"]')
  await expect(results.first()).toContainText(/election/i, { ignoreCase: true })
})
```

## 测试文件组织

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── Button.stories.tsx
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/api/markets/
│   ├── route.ts
│   └── route.test.ts
└── e2e/
    ├── markets.spec.ts
    └── trading.spec.ts
```

## 常见测试错误

### ❌ 测试实现细节
```typescript
// 不要测试内部状态
expect(component.state.count).toBe(5)
```

### ✅ 测试用户可见行为
```typescript
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ 脆弱选择器
```typescript
// 容易破坏
await page.click('.css-class-xyz')
```

### ✅ 语义化选择器
```typescript
// 稳定
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

---

# 验证循环

## 使用时机

- 完成功能或重大代码变更后
- 创建 PR 前
- 确保质量门通过时
- 重构后

## 验证阶段

### Phase 1: 构建验证
```bash
npm run build 2>&1 | tail -20
```
构建失败则停止，修复后再继续。

### Phase 2: 类型检查
```bash
npx tsc --noEmit 2>&1 | head -30
```

### Phase 3: Lint 检查
```bash
npm run lint 2>&1 | head -30
```

### Phase 4: 测试套件
```bash
npm run test -- --coverage 2>&1 | tail -50
```
目标：80% 最低覆盖率。

### Phase 5: 安全扫描
```bash
# 检查 secrets
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# 检查 console.log
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### Phase 6: Diff 审查
```bash
git diff --stat
```
审查每个变更文件。

## 输出格式

```
VERIFICATION REPORT
==================

Build:     [PASS/FAIL]
Types:     [PASS/FAIL] (X errors)
Lint:      [PASS/FAIL] (X warnings)
Tests:     [PASS/FAIL] (X/Y passed, Z% coverage)
Security:  [PASS/FAIL] (X issues)
Diff:      [X files changed]

Overall:   [READY/NOT READY] for PR

Issues to Fix:
1. ...
2. ...
```

## 持续模式

长时间会话中，每 15 分钟或重大变更后运行验证：
```
Set a mental checkpoint:
- After completing each function
- After finishing a component
- Before moving to next task

Run: /verify
```

---

**记住**：质量是每个人的责任。EDD + TDD + Verification 构成完整质量保障体系。
