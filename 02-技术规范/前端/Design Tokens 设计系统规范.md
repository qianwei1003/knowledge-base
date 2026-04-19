---
来源: CSDN博客
原始链接: https://blog.csdn.net/leopold_man/article/details/159836087
---

# Design Tokens：设计系统的 DNA

## 核心要点

- Tokens 是设计系统的最小可复用单元（颜色、字体、间距、圆角）
- **三层架构**：基础 Tokens → 语义 Tokens → 组件 Tokens
- 用 JSON 定义，自动生成 CSS 变量、Flutter/React 代码
- 主题切换只需改语义层引用，无需改动组件代码

## 三层架构

### 1. 基础 Tokens（原始值）
```json
{
  "color": {
    "base": {
      "gray": {
        "100": { "value": "#f7fafc" },
        "900": { "value": "#1a202c" }
      },
      "blue": {
        "500": { "value": "#4299e1" }
      }
    }
  },
  "space": {
    "4": { "value": "1rem" },
    "6": { "value": "1.5rem" }
  }
}
```

### 2. 语义 Tokens（用途定义）
```json
{
  "color": {
    "semantic": {
      "background": {
        "primary": { "value": "{color.base.white}" }
      },
      "text": {
        "primary": { "value": "{color.base.gray.900}" }
      }
    }
  }
}
```

### 3. 组件 Tokens
```json
{
  "component": {
    "button": {
      "primary": {
        "background": { "value": "{color.semantic.action.primary.background}" },
        "padding": { "value": "{space.3} {space.6}" }
      }
    }
  }
}
```

## CSS 变量实现

```css
:root {
  /* Base */
  --color-base-gray-900: #1a202c;
  --color-base-blue-500: #4299e1;

  /* Semantic */
  --color-text-primary: var(--color-base-gray-900);
  --color-action-primary: var(--color-base-blue-500);

  /* Spacing */
  --space-4: 1rem;
  --space-6: 1.5rem;
}

/* Dark Mode — 只改语义层 */
[data-theme="dark"] {
  --color-text-primary: var(--color-base-white);
}
```

## Flutter 实现

```dart
class AppColors {
  // Base
  static const Color baseGray900 = Color(0xFF1A202C);
  static const Color baseBlue500 = Color(0xFF4299E1);

  // Semantic
  static const Color textPrimary = baseGray900;
  static const Color actionPrimary = baseBlue500;
}

class AppSpacing {
  static const double space4 = 16.0;
  static const double space6 = 24.0;
}
```

## 工具链：Style Dictionary

```json
{
  "source": ["tokens/**/*.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "files": [{ "destination": "variables.css", "format": "css/variables" }]
    },
    "flutter": {
      "files": [{ "destination": "tokens.dart", "format": "flutter/class.dart" }]
    }
  }
}
```

## 最佳实践

- ✅ Tokens 纳入版本控制
- ✅ 每个 Token 有明确用途说明
- ✅ 自动化生成各平台代码
- ✅ 语义层命名体现用途（`text-primary`），而非外观（`gray-900`）
