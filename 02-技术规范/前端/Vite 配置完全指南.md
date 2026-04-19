---
来源: 官方文档
原始链接: https://vite.dev/config
---

# Vite 配置完全指南

## 配置文件

Vite 会自动在项目根目录查找 `vite.config.js`（也支持 .ts 等扩展名）。

```js
import { defineConfig } from 'vite'

export default defineConfig({
  // 配置选项
})
```

## 配置智能提示

使用 JSDoc 类型提示或 `defineConfig` 辅助函数获取 IDE 智能提示：

```js
/** @type {import('vite').UserConfig} */
export default { /* ... */ }

// 或
import { defineConfig } from 'vite'
export default defineConfig({ /* ... */ })
```

也支持 TypeScript 配置文件 `vite.config.ts`：

```ts
import type { UserConfig } from 'vite'

export default {
  // ...
} satisfies UserConfig
```

## 条件配置

根据命令（serve/build）、模式、SSR 构建来条件化配置：

```js
export default defineConfig(({ command, mode, isSsrBuild, isPreview }) => {
  if (command === 'serve') {
    return { /* 开发专用配置 */ }
  } else {
    return { /* 生产构建配置 */ }
  }
})
```

## 异步配置

支持导出异步函数：

```js
export default defineConfig(async ({ mode }) => {
  const data = await asyncFunction()
  return { /* 配置 */ }
})
```

## 环境变量

配置评估时的环境变量只能是已存在于当前进程中的 `process.env`。

`.env` 文件在配置解析后自动加载，通过 `import.meta.env` 暴露给应用代码。

如需在配置中读取 `.env` 文件，使用 `loadEnv` 辅助函数：

```js
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV),
    },
    server: {
      port: env.APP_PORT ? Number(env.APP_PORT) : 5173,
    }
  }
})
```

---

*来源: Vite 官方文档 (https://vite.dev/config)*