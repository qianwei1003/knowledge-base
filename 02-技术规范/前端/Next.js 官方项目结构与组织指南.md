---
来源: 官方文档
原始链接: https://nextjs.org/docs/app/getting-started/project-structure
---

# Next.js 官方项目结构与组织指南

## 核心要点

- **文件夹即路由**：`app` 目录结构直接映射 URL 路径
- **特殊文件约定**：`layout`、`page`、`loading`、`error`、`route` 等定义路由行为
- **路由组 `(group)`**：组织代码但不影响 URL 结构
- **私有文件夹 `_folder`**：放置非路由文件（组件、工具函数），不会被路由
- **元数据文件约定**：`icon`、`sitemap`、`robots` 等自动处理 SEO

---

## 顶层结构

### 顶层文件夹

| 文件夹 | 用途 |
|--------|------|
| `app` | App Router（应用路由） |
| `pages` | Pages Router（页面路由） |
| `public` | 静态资源 |
| `src` | 可选的应用源代码文件夹 |

### 顶层文件

| 文件 | 用途 |
|------|------|
| `next.config.js` | Next.js 配置 |
| `package.json` | 项目依赖和脚本 |
| `.env.*` | 环境变量 |
| `tsconfig.json` | TypeScript 配置 |

## 路由文件约定

| 文件 | 用途 |
|------|------|
| `layout` | 布局（共享 UI） |
| `page` | 页面（使路由公开可访问） |
| `loading` | 加载 UI（Skeleton） |
| `not-found` | 404 页面 UI |
| `error` | 错误 UI（React Error Boundary） |
| `route` | API 端点 |
| `template` | 重新渲染的布局 |

### 嵌套路由示例

| 路径 | URL 模式 |
|------|---------|
| `app/layout.tsx` | 根布局，包裹所有路由 |
| `app/page.tsx` | `/` |
| `app/blog/page.tsx` | `/blog` |
| `app/blog/[slug]/page.tsx` | `/blog/my-first-post` |

## 动态路由

| 模式 | 示例 URL |
|------|---------|
| `[segment]` | `/blog/[slug]` → `/blog/my-post` |
| `[...segment]` | `/shop/[...slug]` → `/shop/clothing/shirts` |
| `[[...segment]]` | `/docs/[[...slug]]` → `/docs` 或 `/docs/api/hooks` |

## 路由组和私有文件夹

### 路由组 `(group)`

组织代码但不改变 URL：

```
app/
  (marketing)/
    page.tsx        → URL: /
  (shop)/
    cart/page.tsx   → URL: /cart
```

### 私有文件夹 `_folder`

放置非路由文件：

```
app/
  blog/
    page.tsx              → 可路由
    _components/Post.tsx  → 不可路由
    _lib/data.ts          → 不可路由
```

## 并行路由和拦截路由

### 并行路由 `@folder`

命名槽位，由父布局渲染（如侧边栏 + 主内容）。

### 拦截路由

| 模式 | 含义 | 用例 |
|------|------|------|
| `(.)folder` | 拦截同级 | 模态框预览兄弟路由 |
| `(..)folder` | 拦截父级 | 覆盖方式打开父级子路由 |
| `(...)folder` | 从根拦截 | 当前视图显示任意路由 |

## 元数据文件约定

| 文件 | 用途 |
|------|------|
| `favicon.ico` | Favicon |
| `icon.png` | App 图标 |
| `opengraph-image.png` | OG 图片 |
| `sitemap.xml` | 站点地图 |
| `robots.txt` | Robots 文件 |

## 组件层级

```
layout.js       ← 最外层
  template.js
    error.js
      loading.js
        not-found.js
          page.js     ← 最内层
```

## 推荐目录结构

```
app/
  (marketing)/
    layout.tsx
    page.tsx
    _components/
  (shop)/
    layout.tsx
    cart/
      page.tsx
      _components/
  layout.tsx            # 根布局
  loading.tsx
  error.tsx
  not-found.tsx
components/             # 共享组件
lib/                    # 共享工具函数
public/
  images/
```
