---
来源: 官方文档
领域: 工程化
关键词: EditorConfig, 编辑器配置, 缩进, 编码, 换行符, 代码风格统一
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://editorconfig.org/
---

# EditorConfig - 跨编辑器代码风格统一

> 来源：EditorConfig 官方 https://editorconfig.org/

## 概述

EditorConfig 帮助多个开发者在不同编辑器和 IDE 上保持一致的编码风格。它由文件格式定义 + 编辑器插件组成。

## 示例配置文件

```ini
# EditorConfig is awesome: https://editorconfig.org

# 顶层 EditorConfig 文件
root = true

# 所有文件 - Unix 换行 + 文件末尾空行
[*]
end_of_line = lf
insert_final_newline = true

# JS/Python 文件 - UTF-8 编码
[*.{js,py}]
charset = utf-8

# Python 文件 - 4 空格缩进
[*.py]
indent_style = space
indent_size = 4

# Makefile - Tab 缩进
[Makefile]
indent_style = tab

# lib 目录下的 JS - 2 空格缩进
[lib/**.js]
indent_style = space
indent_size = 2

# 特定文件
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2
```

## 文件查找规则

1. 从打开文件所在目录**向上**搜索 `.editorconfig`
2. 找到 `root = true` 或到达根目录时停止
3. **最近的规则优先**（自上而下覆盖）

## 通配符

| 通配符 | 匹配规则 |
|--------|---------|
| `*` | 除路径分隔符外的任意字符串 |
| `**` | 任意字符串（含路径分隔符） |
| `?` | 任意单个字符 |
| `[name]` | name 中的任意单个字符 |
| `[!name]` | 不在 name 中的任意单个字符 |
| `{s1,s2,s3}` | 任意给定字符串 |
| `{num1..num2}` | num1 到 num2 之间的整数 |

## 支持的属性

| 属性 | 说明 | 可选值 |
|------|------|--------|
| `indent_style` | 缩进方式 | `tab` / `space` |
| `indent_size` | 缩进宽度（空格数） | 整数 |
| `tab_width` | Tab 显示宽度 | 整数（默认=indent_size） |
| `end_of_line` | 换行符 | `lf` / `cr` / `crlf` |
| `charset` | 文件编码 | `latin1` / `utf-8` / `utf-8-bom` / `utf-16be` / `utf-16le` |
| `trim_trailing_whitespace` | 删除行末空格 | `true` / `false` |
| `insert_final_newline` | 文件末尾添加空行 | `true` / `false` |
| `root` | 停止向上搜索 | `true` |

## 原生支持的编辑器

VS Code、IntelliJ 系列（WebStorm/PyCharm 等）、Vim/Neovim、GitHub、GitLab、Xcode 等 30+ 编辑器。

## 需要插件的编辑器

Atom、Sublime Text、Eclipse、Notepad++、Emacs 等。

## 最佳实践

```ini
# 推荐的通用 .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false  # Markdown 中保留行末空格（换行）
```

> 💡 **建议**：不确定的属性可以不写，让编辑器使用默认设置。`indent_size` 在 `indent_style = tab` 时可不写，让读者使用自己偏好的宽度。
