---
来源: 官方文档
原始链接: https://typescript-eslint.io/getting-started/
---

# ESLint + typescript-eslint 配置最佳实践（官方文档）

> 整合自 typescript-eslint 官方 Getting Started、Shared Configs 文档及 ESLint 官方文档。

## 一、快速开始（Flat Config 格式）

### Step 1: 安装依赖

```bash
npm install --save-dev eslint @eslint/js typescript typescript-eslint
```

### Step 2: 创建配置文件

在项目根目录创建 `eslint.config.mjs`：

```javascript
// @ts-check
import js from '@eslint/js';
import { defineConfig } from 'eslint/config';
import tseslint from 'typescript-eslint';

export default defineConfig(
  js.configs.recommended,
  tseslint.configs.recommended,
);
```

### Step 3: 运行

```bash
npx eslint .
```

---

## 二、配置层级详解

typescript-eslint 提供多个预设配置，按严格度递增：

| 配置名 | 适用场景 | 稳定性 |
|--------|----------|--------|
| `recommended` | 代码正确性，零配置即用 | ✅ 稳定（semver 保护） |
| `recommended-type-checked` | 需要类型信息的正确性规则 | ✅ 稳定 |
| `strict` | 比 recommended 更严格，可捕获更多 bug | ⚠️ 非稳定 |
| `strict-type-checked` | strict + 类型感知的严格规则 | ⚠️ 非稳定 |
| `stylistic` | 代码风格一致性，不影响逻辑 | ✅ 稳定 |
| `stylistic-type-checked` | stylistic + 类型感知的风格规则 | ✅ 稳定 |

---

## 三、项目配置推荐方案

### 方案 A：无类型检查的项目

适合快速启动或迁移中的项目：

```javascript
// eslint.config.mjs
export default defineConfig(
  js.configs.recommended,
  tseslint.configs.recommended,
  tseslint.configs.stylistic,
);
```

### 方案 B：有类型检查的项目

需要配置 `languageOptions.parserOptions.project`：

```javascript
// eslint.config.mjs
export default defineConfig(
  js.configs.recommended,
  tseslint.configs.recommendedTypeChecked,
  tseslint.configs.stylisticTypeChecked,
);
```

### 方案 C：高级/成熟团队

如果团队大多数开发者 TypeScript 熟练度高：

```javascript
// eslint.config.mjs
export default defineConfig(
  js.configs.recommended,
  tseslint.configs.strictTypeChecked,
  tseslint.configs.stylisticTypeChecked,
);
```

> ⚠️ `strict` 和 `strict-type-checked` 配置在 semver 下**不保证稳定性**，规则可能在 minor 版本中变更。

---

## 四、推荐配置规则说明

### recommended

- 报告几乎总是不良实践或可能 bug 的规则
- 同时禁用与 typescript-eslint 冲突的核心 ESLint 规则
- **所有项目都建议启用**

### strict

- 包含 recommended 的全部规则
- 额外启用更多可能捕获 bug 但更具主观性的规则
- **仅建议 TS 熟练度高的团队使用**

### stylistic

- 现代 TypeScript 代码库的最佳实践风格
- 不影响程序逻辑
- **与 recommended/strict 互补，不是替代关系**

---

## 五、规则分类概览

ESLint 规则分为四大类：

### Possible Problems（可能的问题）
- 逻辑错误相关：`no-const-assign`、`no-dupe-keys`、`no-unreachable`、`no-unused-vars` 等
- TypeScript 特有：类型相关的可能错误

### Suggestions（建议）
- 命名规范：`camelcase`、`capitalized-comments`
- 代码质量：`max-depth`、`max-lines`、`max-params`、`complexity`
- 最佳实践：`eqeqeq`（强制 ===）、`dot-notation` 等

### Layout & Formatting（布局与格式）
- 缩进、换行、间距等（建议由 Prettier 处理）
- 使用 `eslint-config-prettier` 关闭与 Prettier 冲突的规则

### Deprecated/Removed（已废弃/移除）
- 已不推荐使用的旧规则

---

## 六、TypeScript 配置文件支持

Prettier/ESLint 均支持 TypeScript 配置文件（需 Node.js >= 22.6.0）：

```javascript
// eslint.config.ts
import { type Config } from "prettier";
const config: Config = {
  trailingComma: "none",
};
export default config;
```

---

## 七、常用规则速查

| 类别 | 规则 | 说明 |
|------|------|------|
| 安全 | `no-eval` | 禁止 eval() |
| 安全 | `no-implied-eval` | 禁止类似 eval 的隐式调用 |
| 质量 | `no-console` | 禁止 console（可配 warn） |
| 质量 | `complexity` | 圈复杂度限制 |
| 质量 | `max-depth` | 嵌套深度限制 |
| 质量 | `max-lines-per-function` | 函数最大行数 |
| 质量 | `max-params` | 参数最大数量 |
| 风格 | `eqeqeq` | 强制 === 和 !== |
| 风格 | `camelcase` | 强制驼峰命名 |
| TS 特有 | `no-explicit-any` | 禁止显式 any |
| TS 特有 | `no-unused-vars` | 禁止未使用变量 |
| TS 特有 | `explicit-function-return-type` | 要求显式返回类型 |

---

## 八、ESLint + Prettier 协作

使用 `eslint-config-prettier` 关闭与 Prettier 冲突的 ESLint 规则：

```javascript
// eslint.config.mjs
import eslintConfigPrettier from 'eslint-config-prettier';

export default defineConfig(
  js.configs.recommended,
  tseslint.configs.recommended,
  tseslint.configs.stylistic,
  eslintConfigPrettier,  // 必须放最后
);
```

> `eslint-config-prettier` 必须放在配置数组的**最后**，否则前面的配置会重新启用被关闭的规则。

---

*文档来源：typescript-eslint.io | 最后更新：2026-04*
