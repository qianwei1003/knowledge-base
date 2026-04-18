---
created: 2026-04-18
tags: [#Prompt, #代码生成, #React]
model: Claude
effectiveness: ⭐⭐⭐⭐⭐
---

# React组件生成

## 适用场景

- 快速生成 React 函数组件
- 根据需求描述生成组件骨架
- 生成符合团队规范的组件代码

## Prompt内容

```
你是一个精通 React 和 TypeScript 的高级前端工程师。

请根据以下需求生成一个 React 函数组件：

【需求描述】
{需求内容}

【约束条件】
1. 使用 TypeScript，类型定义完整
2. 使用函数组件 + Hooks，不用 class
3. 遵循单一职责，组件只做一件事
4. Props 类型导出，便于其他文件引用
5. 添加必要的注释，说明组件用途和关键逻辑
6. 样式使用 CSS Modules 或 styled-components（二选一）

【输出格式】
1. 组件代码（完整文件内容）
2. Props 类型定义
3. 使用示例（如何调用这个组件）
```

## 使用示例

**输入：**
```
{需求内容}
一个用户头像组件，支持显示头像图片或默认占位符，支持 sizes 属性控制大小（small/medium/large），点击可触发 onClick 事件。
```

**输出：**
```tsx
// UserAvatar.tsx
import React from 'react';
import styles from './UserAvatar.module.css';

/** 头像尺寸 */
export type AvatarSize = 'small' | 'medium' | 'large';

/** UserAvatar 组件 Props */
export interface UserAvatarProps {
  /** 头像图片地址，为空则显示默认占位符 */
  src?: string;
  /** 头像尺寸 */
  size?: AvatarSize;
  /** 点击事件 */
  onClick?: () => void;
  /** alt 文本 */
  alt?: string;
}

/**
 * 用户头像组件
 *
 * 支持三种尺寸：small(32px)、medium(48px)、large(64px)
 * 未提供 src 时显示默认占位符
 */
export const UserAvatar: React.FC<UserAvatarProps> = ({
  src,
  size = 'medium',
  onClick,
  alt = '用户头像',
}) => {
  const sizeMap: Record<AvatarSize, string> = {
    small: styles.small,
    medium: styles.medium,
    large: styles.large,
  };

  return (
    <img
      src={src || '/default-avatar.png'}
      alt={alt}
      className={`${styles.avatar} ${sizeMap[size]}`}
      onClick={onClick}
      role={onClick ? 'button' : undefined}
      tabIndex={onClick ? 0 : undefined}
    />
  );
};

// 使用示例
// <UserAvatar src="https://example.com/avatar.jpg" size="large" onClick={() => navigate('/profile')} />
```

## 效果评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | ⭐⭐⭐⭐⭐ | 类型完整，结构清晰 |
| 可维护性 | ⭐⭐⭐⭐⭐ | 注释充分，命名规范 |
| 可复用性 | ⭐⭐⭐⭐⭐ | Props 设计合理，覆盖常见场景 |
| 约束遵守 | ⭐⭐⭐⭐⭐ | 严格遵守所有约束条件 |

**优点：**
- 输出结构一致，易于复制使用
- 类型定义完整，TypeScript 支持好
- 自动生成使用示例

**缺点：**
- 复杂组件可能需要多轮迭代
- 样式部分需要手动补充

**改进方向：**
- 可追加要求生成单元测试
- 可追加要求生成 Storybook 文档

## 变体

| 变体 | 变化点 | 链接 |
|------|--------|------|
| React组件-带测试 | 额外生成测试用例 | （待创建） |
| React组件-带状态 | 包含 useState/useEffect | （待创建） |

## 相关 Prompt

- [[API接口生成]]
- [[测试用例生成]]
