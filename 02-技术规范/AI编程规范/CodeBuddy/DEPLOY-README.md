# CodeBuddy 扩展文件快速部署

此文件包含可直接复制到 `.codebuddy` 目录使用的 Skills 和 Agents 文件。

## 📦 包含内容

### Skills
- `skills/codebuddy-doc-fetcher/` - 官方文档获取工具

### Agents
- `agents/codebuddy-doc-fetcher.md` - 文档获取助手

## 🚀 使用方法

### 方法 1：复制到用户目录（推荐）

```powershell
# 复制 Skills
Copy-Item -Recurse "skills" -Destination "$env:USERPROFILE\.codebuddy\"

# 复制 Agents
Copy-Item "agents\codebuddy-doc-fetcher.md" -Destination "$env:USERPROFILE\.codebuddy\agents\"
```

### 方法 2：复制到项目目录

```powershell
# 在项目根目录执行
Copy-Item -Recurse "skills" -Destination ".\.codebuddy\"
Copy-Item "agents\codebuddy-doc-fetcher.md" -Destination ".\.codebuddy\agents\"
```

## 📝 文件说明

| 文件 | 用途 | 位置 |
|:---|:---|:---|
| `SKILL.md` | 定义 Skill 的核心配置和工作流 | `skills/codebuddy-doc-fetcher/` |
| `codebuddy-doc-fetcher.md` | 定义 Agent 的角色和行为 | `agents/` |

## ⚙️ 验证安装

安装后重启 CodeBuddy，在对话中输入以下内容测试：

```
"帮我查一下 CodeBuddy 的 skills 文件格式"
```

如果 Skill 正常工作，系统会自动获取并展示相关文档。
