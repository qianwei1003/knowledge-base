---
来源: 官方文档
原始链接: https://react.dev/reference/rsc/server-components
---

# React Server Components 官方指南

## 核心要点

- Server Components 在服务器/构建时执行，**零 JS 发送到浏览器**
- 无需构建 API 层，直接在组件中 `await` 数据库/文件系统
- 异步组件配合 Suspense 实现**流式渲染**，渐进式加载
- 用 `"use client"` 标记客户端组件边界；**没有** `"use server"` 指令
- Promise 可在服务器创建、在客户端通过 `use()` 等待

---

## 一、无需服务器的 Server Components

Server Components 可在构建时运行，读取静态内容，无需 Web 服务器。

```js
// ✅ 构建时渲染，marked/sanitizeHtml 不打入 bundle
import marked from 'marked'; // 不包含在 bundle 中
import sanitizeHtml from 'sanitize-html';

async function Page({page}) {
  const content = await file.readFile(`${page}.md`);
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

```js
// ❌ 传统方式：用户需下载 75K 库 + 等待第二次请求
function Page({page}) {
  const [content, setContent] = useState('');
  useEffect(() => {
    fetch(`/api/content/${page}`).then(data => setContent(data.content));
  }, [page]);
  return <div dangerouslySetInnerHTML={{__html: content}} />;
}
```

---

## 二、带服务器的 Server Components（页面请求时）

直接访问数据层，无 API 即可获取数据，支持 SSR。

```js
// ✅ 消除客户端-服务器瀑布流
import db from './database';

async function Note({id}) {
  const note = await db.notes.get(id);
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

async function Author({id}) {
  const author = await db.authors.get(id);
  return <span>By: {author.name}</span>;
}
```

```js
// ❌ 传统方式：客户端瀑布流，每次 useEffect 等待上一次渲染完成后才请求
function Note({id}) {
  const [note, setNote] = useState('');
  useEffect(() => {
    fetch(`/api/notes/${id}`).then(data => setNote(data.note));
  }, [id]);
  // Author 又需要等待 Note 的 authorId 才能发请求...
}
```

---

## 三、为 Server Components 添加交互性

用 `"use client"` 将组件与 Client Component 组合。

> **常见误解**：`"use server"` 标记 Server Components——这是错的！该指令用于 **Server Functions**。

```js
// Server Component（无 "use server" 指令）
import Expandable from './Expandable';

async function Notes() {
  const notes = await db.notes.getAll();
  return (
    <div>
      {notes.map(note => (
        <Expandable key={note.id}><p>{note}</p></Expandable>
      ))}
    </div>
  )
}
```

```js
// Client Component（必须 "use client"）
"use client"
export default function Expandable({children}) {
  const [expanded, setExpanded] = useState(false);
  return (
    <div>
      <button onClick={() => setExpanded(!expanded)}>Toggle</button>
      {expanded && children}
    </div>
  )
}
```

---

## 四、Async Components（异步组件）

Server Components 支持 `async/await`，React 配合 Suspense 流式渲染。

```js
// 服务器创建 Promise，客户端用 use() 等待（不阻塞主内容渲染）
async function Page({id}) {
  const note = await db.notes.get(id);           // 服务器端同步等待
  const commentsPromise = db.comments.get(id);   // 启动 Promise
  return (
    <div>
      {note}
      <Suspense fallback={<p>Loading Comments...</p>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </div>
  );
}
```

```js
"use client"
// 客户端用 use() 恢复 Promise，不会阻塞首屏
function Comments({commentsPromise}) {
  const comments = use(commentsPromise);
  return comments.map(c => <p>{c}</p>);
}
