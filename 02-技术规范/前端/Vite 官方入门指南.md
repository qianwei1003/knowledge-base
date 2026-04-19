---
来源: 官方文档
原始链接: https://vite.dev/guide
---

# Vite 官方入门指南

## 概述

Vite（法语"快速"之意，发音 /viːt/）是一个构建工具，旨在为现代 Web 项目提供更快、更精简的开发体验。它由两部分组成：

- **开发服务器**：提供丰富的功能增强，例如极快的热模块替换（HMR）
- **构建命令**：使用 Rolldown 打包代码，输出高度优化的静态资源

Vite 开门即用，提供合理的默认配置。可通过插件扩展框架集成。

## 浏览器支持

开发时，Vite 假设使用现代浏览器。生产构建默认目标是 Widely Available 浏览器（发布至少 2.5 年前的浏览器）。

## 在线试用

可通过 StackBlitz 在线试用：访问 vite.new/{template} 选择框架模板。

支持模板：vanilla, vue, react, preact, lit, svelte, solid, qwik（各带 JS/TS 版本）

## 创建 Vite 项目

```bash
npm create vite@latest
# 或指定模板
npm create vite@latest my-vue-app -- --template vue
```

其他包管理器：yarn create vite, pnpm create vite, bun create vite

## index.html 与项目根目录

Vite 项目中，index.html 位于项目根目录（而非 public 文件夹），这是应用的入口点。

Vite 将 index.html 视为源代码的一部分，参与模块依赖图解析。

## 命令行接口

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

---

*来源: Vite 官方文档 (https://vite.dev/guide)*