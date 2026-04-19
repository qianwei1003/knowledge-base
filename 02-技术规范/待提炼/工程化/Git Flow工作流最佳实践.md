---
来源: 官方文档/博客
领域: 工程化
关键词: Git,Git Flow,分支管理,版本控制,工作流
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://zhuanlan.zhihu.com/p/398684084
---

# Git Flow 工作流最佳实践

## 一、Git Flow 概述

Git Flow 是由 Vincent Driessen 在 2010 年提出的一种 Git 分支管理策略。它不是 Git 提供的工具，而是一种关于如何使用分支的约定。Git Flow 已经被认为是现代连续软件开发和 DevOps 实践的最佳实践。

## 二、Git Flow 工作流结构

### 1. 永久分支
- **master/main** - 生产环境分支，仅包含稳定代码
- **develop** - 集成环境分支，下一个发布版本

### 2. 支持性分支
- **feature/\*** - 新功能开发
- **release/\*** - 为生产环境做准备
- **hotfix/\*** - 生产环境中的紧急修复

## 三、Git Flow 工作流详解

### 总体规范建议
统一以版本号命名，各分支以版本号对应，比如：
- feature/v1.0
- release/v1.0
- tag/v1.0

### 1. 主分支 Master
- 稳定版本代码分支
- 不能在此分支直接修改代码
- 只接受 hotfix、release 分支的代码合并
- 每次从 release/hotfix 分支合并必须打版本号 tag，以方便后续的代码追溯

### 2. 主开发分支 Develop
- 每个 feature 迭代从 develop 拉取分支
- 该分支只接受 hotfix、release 分支的代码合并
- 该分支禁止直接合并到 master

### 3. 新功能分支 Feature
- 新功能迭代开发分支
- 开发人员开发完成后合并生成预发布分支 release/xxx（与 feature 分支一一对应）
- 此分支禁止测试、禁止发布上线
- 禁止直接合并到 develop
- 禁止直接合并到 master

### 4. 预发布分支 Release
- feature 分支由开发自测完成后合并到 develop 分支
- 测试人员再从 develop 分支新建对应的 release 分支
- 测试部进行集成测试、开发部修改 bug、产品验收
- 测试通过后（发布上线前）将此分支合并到 develop 和 master 分支
- 然后可将 release 分支删除

### 5. 热修复分支 Hotfix
- 生产环境出现紧急 bug 时使用
- 从 master 拉取
- 修复完成后合并到 master 和 develop
- 打 tag 标记

## 四、分步工作流及真实示例

### 开发新功能场景
**场景**：团队需要为应用添加 OAuth2 身份验证。

```bash
# 1. 从 develop 分支创建功能分支
git checkout develop
git pull origin develop
git checkout -b feature/oauth2-integration

# 2. 在功能分支上工作（多次提交）
git add .
git commit -m "Add Google OAuth2 basic setup"
git commit -m "Implement token refresh logic"
git commit -m "Add user profile sync"

# 3. 完成功能
git checkout develop
git merge --no-ff feature/oauth2-integration
git branch -d feature/oauth2-integration
git push origin develop
```

### 发布新版本场景
**场景**：准备发布 v1.2.0 版本。

```bash
# 1. 从 develop 创建 release 分支
git checkout develop
git checkout -b release/v1.2.0

# 2. 准备发布（修复bug、更新版本号等）
git commit -m "Bump version to 1.2.0"
git commit -m "Fix minor bugs"

# 3. 完成发布
git checkout master
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"

git checkout develop
git merge --no-ff release/v1.2.0

git branch -d release/v1.2.0
git push origin master --tags
git push origin develop
```

### 紧急修复场景
**场景**：生产环境发现严重 bug 需要立即修复。

```bash
# 1. 从 master 创建 hotfix 分支
git checkout master
git checkout -b hotfix/critical-login-bug

# 2. 修复 bug
git commit -m "Fix critical login validation bug"

# 3. 完成修复
git checkout master
git merge --no-ff hotfix/critical-login-bug
git tag -a v1.2.1 -m "Hotfix version 1.2.1"

git checkout develop
git merge --no-ff hotfix/critical-login-bug

git branch -d hotfix/critical-login-bug
git push origin master --tags
git push origin develop
```

## 五、Git Flow 与其他工作流对比

### Git Flow
**优势**：
- 在项目开发周期内，所有分支都保持了一个干净的状态
- 分支名称采用语义命名方式便于理解
- 生产环境多版本支持

**劣势**：
- 分支比较复杂
- Git 提交历史可读性差
- master/develop 分支拆分有点冗余，不利于 CI/CD
- 如果生产环境中只需要维护一个版本不推荐使用

### GitHub Flow
**特点**：
- 只有一个长期分支 master
- 新功能从 master 创建分支
- 开发完成后创建 Pull Request
- 代码审查通过后合并到 master

**优势**：
- 简单易懂
- 适合持续部署
- 分支管理简单

**劣势**：
- 不适合多版本并行开发
- 缺少明确的发布流程

### GitLab Flow
**特点**：
- 结合了 Git Flow 和 GitHub Flow 的优点
- 支持环境分支（如 production、staging）
- 支持基于发布分支的流程

## 六、Git Flow 最佳实践

### 1. 分支命名规范
- feature/功能描述
- release/版本号
- hotfix/bug描述

### 2. 提交信息规范
- feat: 新功能
- fix: 修复bug
- docs: 文档更新
- style: 代码格式调整
- refactor: 重构
- test: 测试相关
- chore: 构建/工具相关

### 3. 代码审查
- 所有合并到 master 和 develop 的代码必须经过 Pull Request
- 至少一人审查通过才能合并
- 使用 --no-ff 保留分支历史

### 4. 自动化
- 使用 CI/CD 工具自动化测试和部署
- 合并到 master 自动触发生产环境部署
- 合并到 develop 自动触发测试环境部署

## 七、Git Flow 工具支持

### 安装 git-flow
```bash
# Mac OS X
brew install git-flow-avh

# Linux
apt-get install git-flow

# Windows
下载安装包或使用 Git Bash
```

### 初始化 Git Flow
```bash
git flow init
```

### 使用 Git Flow 命令
```bash
# 开始新功能
git flow feature start oauth2-integration

# 完成功能
git flow feature finish oauth2-integration

# 开始发布
git flow release start v1.2.0

# 完成发布
git flow release finish v1.2.0

# 开始热修复
git flow hotfix start critical-bug

# 完成热修复
git flow hotfix finish critical-bug
```

## 八、总结

Git Flow 是一种成熟的分支管理策略，特别适合：
- 有明确发布周期的项目
- 需要维护多个生产版本的项目
- 大型团队协作开发

选择工作流时应根据团队规模、项目特点和发布频率来决定，没有一种工作流适合所有场景。
