---
来源: GitHub官方文档
领域: 工程化
关键词: GitHub Actions, CI/CD, 工作流, 自动化, DevOps, 持续集成, 持续部署
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://docs.github.com/en/actions
---

# GitHub Actions CI/CD 工作流完全指南

> 基于 GitHub 官方文档整理，覆盖工作流语法、安全加固、依赖缓存、Job 编排等核心内容。
> 来源：https://docs.github.com/en/actions

---

## 一、工作流基础

### 1.1 什么是工作流

工作流是一个可配置的自动化过程，由一个或多个 Job 组成。工作流通过 YAML 文件定义，必须存放在仓库的 `.github/workflows/` 目录下。

### 1.2 基本结构

```yaml
name: 工作流名称          # 显示在 Actions 标签页的名称
run-name: 部署到 ${{ inputs.deploy_target }}  # 单次运行的名称（支持表达式）

on:                       # 触发条件
  push:
    branches: [main]

jobs:                     # 一个或多个 Job
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
```

---

## 二、触发条件（on）

### 2.1 单事件触发

```yaml
on: push    # push 到任意分支时触发
```

### 2.2 多事件触发

```yaml
on: [push, fork]   # 任一事件触发即可
```

### 2.3 活动类型（Activity Types）

某些事件有更细粒度的活动类型：

```yaml
on:
  issues:
    types:
      - opened
      - labeled
  label:
    types:
      - created
```

### 2.4 过滤器（Filters）

#### 分支过滤

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
  pull_request:
    branches-ignore:
      - 'releases/**-alpha'
```

#### 同时包含和排除分支

```yaml
on:
  pull_request:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'    # ! 排除
```

#### 路径过滤

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'tests/**'
    paths-ignore:
      - 'docs/**'
      - '*.md'
```

### 2.5 定时触发

```yaml
on:
  schedule:
    - cron: '30 5 * * 1'    # 每周一 05:30 UTC
```

### 2.6 工作流调用触发

```yaml
on:
  workflow_dispatch:
    inputs:
      deploy_target:
        description: '部署目标环境'
        required: true
        default: 'staging'
```

### 2.7 多事件混合配置

每个事件必须单独配置（即使没有额外配置也需要冒号）：

```yaml
on:
  label:
    types:
      - created
  push:
    branches:
      - main
  page_build:         # 无额外配置，但需要冒号分隔
```

---

## 三、Job 配置

### 3.1 Job ID 和名称

```yaml
jobs:
  my_first_job:
    name: 我的第一个任务
  my_second_job:
    name: 我的第二个任务
```

> **注意**：`job_id` 必须以字母或 `_` 开头，只能包含字母、数字、`-` 和 `_`。

### 3.2 依赖关系（needs）

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
  job2:
    needs: job1              # 等待 job1 完成
    runs-on: ubuntu-latest
  job3:
    needs: [job1, job2]      # 等待两者都完成
    runs-on: ubuntu-latest
```

执行顺序：job1 → job2 → job3（串行）

### 3.3 始终执行

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
  job2:
    needs: job1
    runs-on: ubuntu-latest
  job3:
    if: ${{ always() }}       # 即使 job1/job2 失败也执行
    needs: [job1, job2]
    runs-on: ubuntu-latest
```

### 3.4 并行执行

没有 `needs` 关系的 Job 会并行运行：

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
  test:
    runs-on: ubuntu-latest
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]     # lint 和 test 并行，都完成后执行 build
```

### 3.5 矩阵策略

自动以不同变量组合运行 Job：

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        node-version: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
```

---

## 四、安全加固

### 4.1 Secret 管理最佳实践

#### 最小权限原则

- 仓库有写权限的用户可读取所有 Secret，因此应确保凭据仅具有所需的最小权限
- 默认将 `GITHUB_TOKEN` 设为只读，在 Job 中按需提升

#### 掩码敏感数据

```yaml
steps:
  - name: Mask sensitive value
    run: echo "::add-mask::${{ secrets.MY_SECRET }}"
```

#### 不要使用结构化数据存储 Secret

```yaml
# ❌ 错误 — JSON 结构化数据会导致掩码失败
secret: '{"token":"abc123","key":"xyz789"}'

# ✅ 正确 — 为每个敏感值创建独立的 Secret
secrets:
  API_TOKEN: abc123
  API_KEY: xyz789
```

#### 注册工作流中生成的敏感值

如果 Secret 用于在工作流中生成其他敏感值，必须注册该值：

```yaml
steps:
  - name: Generate token
    id: generate
    run: |
      TOKEN=$(generate_token)
      echo "::add-mask::$TOKEN"
      echo "token=$TOKEN" >> $GITHUB_OUTPUT
```

#### Secret 审计与轮换

- 定期审查已注册的 Secret，移除不再需要的
- 定期轮换 Secret，减少被泄露后的有效时间窗口
- 审查工作流源码和 Action，确保 Secret 未发送到非预期主机

### 4.2 防止脚本注入

#### 方式一：使用 Action（推荐）

```yaml
# ✅ 安全 — 通过参数传递，不构成注入
uses: fakeaction/checktitle@v3
with:
  title: ${{ github.event.pull_request.title }}
```

#### 方式二：使用中间环境变量

```yaml
# ✅ 安全 — 值存储在环境变量中，不参与脚本生成
- name: Check PR title
  env:
    TITLE: ${{ github.event.pull_request.title }}
  run: |
    if [[ "$TITLE" =~ ^octocat ]]; then
      echo "PR title starts with 'octocat'"
    fi
```

#### ❌ 危险写法

```yaml
# ❌ 危险 — 表达式直接嵌入 shell 命令
- run: echo "${{ github.event.pull_request.title }}"
```

### 4.3 第三方 Action 安全

#### Pin 到完整 Commit SHA

```yaml
# ✅ 安全 — Pin 到完整 SHA（唯一不可变的方式）
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11

# ⚠️ 风险 — Tag 可被移动或删除
- uses: actions/checkout@v4

# ❌ 高风险 — 分支引用可变
- uses: actions/checkout@main
```

#### 审查 Action 源码

确保 Action 正确处理仓库内容和 Secret：
- 检查 Secret 是否被发送到非预期主机
- 检查是否存在日志输出敏感信息

### 4.4 其他安全特性

| 特性 | 说明 |
|------|------|
| **CODEOWNERS** | 控制工作流文件的变更需经指定审查者批准 |
| **OpenID Connect** | 直接向云提供商认证，无需存储长期凭据 |
| **Dependabot** | 自动更新 Action 和可复用工作流的引用 |
| **Environment 审查** | 要求审查者批准后才能访问环境 Secret |

---

## 五、依赖缓存

### 5.1 缓存 Action 使用

```yaml
- name: Cache node modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-build-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-build-
      ${{ runner.os }}-
```

### 5.2 缓存匹配逻辑

1. 首先搜索 `key` 的精确匹配
2. 无精确匹配 → 搜索 `key` 的部分匹配
3. 仍无匹配 → 按顺序搜索 `restore-keys`
4. 当前分支无匹配 → 搜索默认分支

### 5.3 检测缓存命中

```yaml
- if: ${{ steps.cache-npm.outputs.cache-hit != 'true' }}
  name: List the state of node modules
  continue-on-error: true
  run: npm list
```

### 5.4 setup-* Actions 内置缓存

| 包管理器 | 内置缓存的 Action |
|----------|-------------------|
| npm, Yarn, pnpm | `setup-node` |
| pip, pipenv, Poetry | `setup-python` |
| Gradle, Maven | `setup-java` |
| RubyGems | `setup-ruby` |
| Go (go.sum) | `setup-go` |
| .NET NuGet | `setup-dotnet` |

### 5.5 缓存限制

- 默认总大小限制：**10 GB/仓库**
- 超出 7 天未访问的缓存自动删除
- 超出限制时按最近访问时间从旧到新淘汰

---

## 六、GITHUB_TOKEN 权限管理

### 6.1 最小权限配置

```yaml
# 工作流级别
permissions:
  contents: read

jobs:
  deploy:
    permissions:
      contents: read
      deployments: write    # 仅 deploy job 需要写权限
```

### 6.2 权限类型

| 权限 | 说明 |
|------|------|
| `contents` | 仓库内容读写 |
| `issues` | Issue 管理 |
| `pull-requests` | PR 管理 |
| `deployments` | 部署管理 |
| `packages` | 包管理 |
| `id-token` | OIDC 令牌 |
| `security-events` | 安全事件 |

---

## 七、常见工作流模板

### 7.1 Node.js CI

```yaml
name: Node.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - run: npm test
```

### 7.2 构建 + 部署

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - name: Deploy
        run: ./deploy.sh
```

### 7.3 定时健康检查

```yaml
name: Health Check

on:
  schedule:
    - cron: '0 */6 * * *'    # 每 6 小时

jobs:
  health-check:
    runs-on: ubuntu-latest
    steps:
      - name: Check API
        run: curl -f https://api.example.com/health || exit 1
```

---

## 八、最佳实践总结

### 工作流设计
1. **保持 Job 小而专注** — 每个 Job 做一件事
2. **善用并行** — 无依赖的 Job 并行执行，缩短总时间
3. **使用矩阵** — 多版本/多平台测试用矩阵策略
4. **合理使用缓存** — 依赖缓存可显著缩短构建时间

### 安全
5. **最小权限** — GITHUB_TOKEN 和 Secret 仅赋予所需最小权限
6. **Pin Action 到 SHA** — 永远 pin 到完整 commit SHA
7. **不使用结构化 Secret** — 每个敏感值独立存储
8. **防止脚本注入** — 用 Action 或环境变量处理用户输入

### 可维护性
9. **使用可复用工作流** — 将通用流程抽为可复用工作流
10. **使用环境** — 区分 staging/production 等部署环境
11. **善用 Dependabot** — 自动更新 Action 版本
12. **清晰的命名** — Job、Step、工作流名称要有意义

---

> 📚 完整文档：https://docs.github.com/en/actions
> 🔒 安全参考：https://docs.github.com/en/actions/reference/security/secure-use
> 📦 缓存参考：https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching
