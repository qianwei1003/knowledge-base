---
来源: 官方文档
原始链接: https://prettier.io/docs/configuration.html
---

# Prettier 配置与选项完整指南（官方文档）

> 整合自 Prettier 官方 Configuration、Options 文档。

## 一、配置文件格式（按优先级排序）

| 文件名 | 格式 |
|--------|------|
| `package.json` 中的 `"prettier"` 键 | JSON |
| `.prettierrc` | JSON 或 YAML |
| `.prettierrc.json` / `.yml` / `.yaml` / `.json5` | 对应格式 |
| `.prettierrc.js` / `prettier.config.js` | ES Modules / CommonJS |
| `.prettierrc.ts` / `prettier.config.ts` | TypeScript（需 Node >= 22.6） |
| `.prettierrc.mjs` / `.prettierrc.cjs` | ES Modules / CommonJS |
| `.prettierrc.toml` | TOML |

> Prettier **不支持全局配置**，确保项目复制到其他机器时行为一致。

---

## 二、基础配置示例

### JSON 格式

```json
{
  "trailingComma": "es5",
  "tabWidth": 4,
  "semi": false,
  "singleQuote": true
}
```

### YAML 格式

```yaml
trailingComma: "es5"
tabWidth: 4
semi: false
singleQuote: true
```

### JavaScript (ES Modules)

```javascript
/** @type {import("prettier").Config} */
const config = {
  trailingComma: "es5",
  tabWidth: 4,
  semi: false,
  singleQuote: true,
};
export default config;
```

### TypeScript (ES Modules)

```typescript
import { type Config } from "prettier";

const config: Config = {
  trailingComma: "none",
};
export default config;
```

---

## 三、配置覆盖（Overrides）

针对不同文件类型/目录使用不同配置：

```json
{
  "semi": false,
  "overrides": [
    {
      "files": "*.test.js",
      "options": { "semi": true }
    },
    {
      "files": ["*.html", "legacy/**/*.js"],
      "options": { "tabWidth": 4 }
    }
  ]
}
```

> `files` 是必填项，可选 `excludeFiles` 排除文件。

---

## 四、Parser 配置

默认按文件扩展名自动推断 parser。**不要在顶级配置中设置 parser**，仅在 overrides 中使用：

```json
{
  "overrides": [
    {
      "files": ".prettierrc",
      "options": { "parser": "json" }
    }
  ]
}
```

---

## 五、完整选项参考

### 格式化核心选项

| 选项 | 默认值 | CLI 参数 | 说明 |
|------|--------|----------|------|
| `printWidth` | `80` | `--print-width <int>` | 行宽（建议 80，不超 100） |
| `tabWidth` | `2` | `--tab-width <int>` | 缩进空格数 |
| `useTabs` | `false` | `--use-tabs` | 用 Tab 代替空格 |
| `semi` | `true` | `--no-semi` | 语句末尾分号 |
| `singleQuote` | `false` | `--single-quote` | 使用单引号 |
| `quoteProps` | `"as-needed"` | `--quote-props` | 对象属性引号策略 |
| `jsxSingleQuote` | `false` | `--jsx-single-quote` | JSX 中使用单引号 |
| `trailingComma` | `"all"` (v3+) | `--trailing-comma` | 尾逗号策略 |
| `bracketSpacing` | `true` | `--no-bracket-spacing` | 对象大括号内空格 |
| `bracketSameLine` | `false` | `--bracket-same-line` | `>` 是否与最后属性同行 |
| `arrowParens` | `"always"` | `--arrow-parens` | 箭头函数单参数括号 |
| `endOfLine` | `"lf"` | `--end-of-line` | 换行符风格 |
| `proseWrap` | `"preserve"` | `--prose-wrap` | Markdown 换行 |

### 实验性选项

| 选项 | 默认值 | CLI 参数 | 说明 |
|------|--------|----------|------|
| `experimentalTernaries` | `false` | `--experimental-ternaries` | 新三元表达式格式 |
| `experimentalOperatorPosition` | `"end"` | `--experimental-operator-position` | 运算符位置 |

### v3.0 新增/变更选项

| 选项 | 说明 |
|------|------|
| `objectWrap` (`"preserve"`/`"collapse"`) | 对象字面量换行策略，v3.5.0+ |
| `trailingComma` 默认值从 `"es5"` 改为 `"all"` | |

---

## 六、重要选项详解

### printWidth（行宽）

> ⚠️ Prettier 的 `printWidth` 不同于 ESLint 的 `max-len`。
> - `max-len`：硬性上限
> - `printWidth`：期望的**大致**行宽，Prettier 会灵活处理

建议值：
- 库/开源项目：`80`
- 业务项目：`80`（Prettier 默认，推荐不超过 100）

### trailingComma（尾逗号）

| 值 | 说明 |
|----|------|
| `"all"` | 所有位置加尾逗号（v3 默认），需 ES2017+ |
| `"es5"` | 仅 ES5 合法位置（对象、数组） |
| `"none"` | 不加尾逗号 |

### endOfLine（换行符）

| 值 | 说明 |
|----|------|
| `"lf"` | Unix 风格（推荐） |
| `"crlf"` | Windows 风格 |
| `"cr"` | 旧 Mac 风格 |
| `"auto"` | 保持原文件换行符 |

### quoteProps（对象属性引号）

| 值 | 说明 |
|----|------|
| `"as-needed"` | 仅必要时加引号（默认） |
| `"consistent"` | 一个属性需要引号则全部加 |
| `"preserve"` | 保留输入中的引号 |

### objectWrap（对象换行，v3.5+）

| 值 | 说明 |
|----|------|
| `"preserve"` | 如果 `{` 后有换行则保持多行（默认） |
| `"collapse"` | 尽可能压缩为单行 |

---

## 七、EditorConfig 集成

Prettier 会读取项目中的 `.editorconfig` 文件，但 `.prettierrc` 优先级更高。

推荐的 `.editorconfig`（与 Prettier 默认值一致）：

```ini
[*]
charset = utf-8
insert_final_newline = true
end_of_line = lf
indent_style = space
indent_size = 2
max_line_length = 80
```

> ⚠️ `trim_trailing_whitespace` 由编辑器处理，Prettier 不处理模板字符串内的尾部空白。

---

## 八、配置验证

使用 JSON Schema 验证配置文件：[schemastore.org/prettierrc.json](https://www.schemastore.org/prettierrc.json)

---

## 九、最佳实践总结

1. **项目根目录放 `.prettierrc.json`**，不用 `package.json` 中的 key（避免文件膨胀）
2. **`printWidth` 保持 80**，不要设太高
3. **不要设全局 Prettier 配置**，每个项目独立
4. **与 ESLint 配合**：用 `eslint-config-prettier` 关闭冲突规则
5. **配合 `.editorconfig`**：保持编辑器和格式化工具一致
6. **CI 中加格式检查**：`prettier --check .`，防止未格式化代码提交

---

*文档来源：prettier.io | 最后更新：2026-04*
