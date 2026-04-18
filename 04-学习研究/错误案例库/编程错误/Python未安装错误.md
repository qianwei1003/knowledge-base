---
created: 2026-04-18
tags: [#错误案例, #编程/Python, #环境配置]
severity: 严重
status: 已解决
---

# Python未安装导致脚本执行失败

## 错误描述

在 Windows 环境下执行 Python 脚本时，命令行显示 `python` 不是内部或外部命令。

实际运行 `python --version` 时，Windows Store 弹出应用商店页面，说明系统只有 Python 的占位符，而不是真正的 Python 安装。

**影响范围：**
- 无法执行 `.py` 脚本
- qclaw-text-file Skill 的 `write_file.py` 脚本无法运行
- 依赖 Python 的自动化任务全部阻塞

## 错误原因

1. **Windows 10/11 默认行为**：系统没有安装 Python 时，`python` 命令会重定向到 Microsoft Store
2. **环境变量缺失**：即使安装了 Python，如果没有添加到 PATH，也会出现同样问题
3. **假设环境已就绪**：开发时假设用户环境已配置，没有做预检测

## 解决方案

### 方案一：安装 Python（推荐）

1. 从 [Python官网](https://www.python.org/downloads/) 下载安装包
2. 安装时勾选 **"Add Python to PATH"**（关键步骤）
3. 重启命令行，验证：`python --version`

### 方案二：使用 winget 安装

```powershell
winget install Python.Python.3.12
```

### 方案三：临时绕过

如果不需要 Python 环境，可以：
- 用其他工具替代（如 PowerShell 脚本）
- 使用在线 Python 解释器

## 避坑指南

### 1. 开发前检测环境
```powershell
# 检测 Python 是否可用
python --version 2>&1
if ($LASTEXITCODE -ne 0) {
    Write-Warning "Python 未安装或未添加到 PATH"
}
```

### 2. 文档明确依赖
在 Skill 文档或项目 README 中明确标注：
```
依赖：Python 3.8+
```

### 3. 提供降级方案
如果 Python 不是必需，提供替代方案：
- PowerShell 替代脚本
- 纯前端解决方案
- 在线服务

### 4. 用户环境假设
不要假设用户环境已配置：
- 新机器、新系统、重装系统都有可能
- 提供环境检测脚本
- 提供一键安装脚本

## 教训总结

| 类别 | 教训 |
|------|------|
| 环境依赖 | 必须做环境检测，不能假设用户已配置 |
| 文档规范 | 依赖项必须在文档中明确声明 |
| 降级方案 | 关键功能要有降级方案，不能单点依赖 |
| 测试覆盖 | 在纯净环境测试，验证首次体验 |

## 相关案例

- （待补充：类似的环境依赖问题）

## 关联文档

- [[../../06-AI协作区/qclaw-text-file|qclaw-text-file Skill]]
- [[../../00-Index/知识库使用指南|知识库使用指南]]
