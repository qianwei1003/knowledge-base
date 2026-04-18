# Google Python 风格规范

> **来源**: [Google Python Style Guide](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/)
> **特点**: 基于 PEP 8，Google 内部实践

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **缩进** | 4空格，禁止 Tab |
| **行长度** | 最大 80 字符 |
| **模块名** | `lower_with_under` |
| **类名/异常** | `CapWords` |
| **函数/变量** | `lower_with_under` |
| **常量** | `CAPS_WITH_UNDER` |
| **字符串** | 使用 f-string，避免循环内拼接 |

---

## 一、缩进与格式

### 缩进规范

```python
# ✅ 使用 4 个空格缩进
def greet(name):
    if name == 'world':
        print('Hello!')
    else:
        print('Hi,', name)

# ✅ 使用圆括号实现隐式续行
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
)

# ❌ 禁止使用 Tab
# ❌ 禁止使用反斜杠续行
```

### 行长度

```python
# ✅ 每行最大 80 字符
# 例外：长导入语句、URL、模块级长字符串

# ✅ 使用括号续行
long_name = some_function(
    'argument one', 'argument two',
    'argument three',
)
```

---

## 二、命名规范

| 类型 | 风格 | 示例 |
|------|------|------|
| 模块/包 | `lower_with_under` | `my_module.py` |
| 类名/异常 | `CapWords` | `MyClass`, `ValueError` |
| 函数/方法/变量 | `lower_with_under` | `get_name()`, `user_id` |
| 全局常量/类常量 | `CAPS_WITH_UNDER` | `MAX_COUNT` |
| 私有变量 | `_internal_var` | `_private_helper()` |
| 受保护变量 | `_protected_var` | `_base_class` |

### 命名原则

```python
# ✅ 使用有意义的名称，避免缩写
user_name = 'John'           # ✅
usr = 'John'                 # ❌

# ❌ 避免单字符名（除特定情况）
for i in range(10):          # ✅ 计数器可用
for j in range(10):          # ✅ 嵌套循环可用
except e:                    # ✅ 异常可用
with open('f') as f:         # ✅ 文件句柄可用

# ✅ 文件名必须以 .py 结尾，不能含连字符
my_module.py                 # ✅
my-module.py                 # ❌
```

---

## 三、注释与文档字符串

### Docstring 格式

```python
# ✅ 必须使用三重双引号
"""模块描述。

这个模块提供了...
"""

# ✅ 函数文档字符串
def fetch_smalltalk(talk: FetchTalk) -> str:
    """Fetch the latest small talk about nothing.

    Args:
        talk: The talk object to fetch.

    Returns:
        The talk content as a string.

    Raises:
        FetchError: If the talk cannot be fetched.
    """
    return talk.content

# ❌ 不要仅描述代码（解释为什么）
# ✅ 注释应解释"为什么"，而非"做什么"
```

### 行注释

```python
# ✅ 复杂操作前加注释
# We assume a coordinate is in the valid range.
if value > 0:
    process()

# ✅ 不直观的代码加行注释
try:
    process()
except OSError:  # Devs didn't anticipate this error.
    pass
```

---

## 四、导入规范

```python
# ✅ 导入独占一行
import os
import sys

# ✅ 按分组顺序排列
import os              # 标准库
import sys             # 标准库

import aiohttp         # 第三方
import pytest          # 第三方

from typing import Dict, List  # 导入顺序：typing > collections.abc

from mymodule import MyClass   # 本地/子包

# ✅ 组内按字典序排列
```

### 导入顺序

1. `__future__` 导入
2. Python 标准库
3. 第三方模块
4. 代码仓库中的子包

---

## 五、字符串规范

```python
# ✅ 使用 f-string（Python 3.6+）
name = 'World'
print(f'Hello, {name}!')

# ✅ 或使用 % 运算符 / format
print('Hello, %s!' % name)
print('Hello, {}!'.format(name))

# ❌ 避免在循环内用 + 拼接字符串
# 会导致平方级时间复杂度
```

### 字符串拼接

```python
# ❌ 错误：循环内拼接
result = ''
for s in items:
    result += s          # ❌ 平方级复杂度

# ✅ 正确：列表 + join
parts = []
for s in items:
    parts.append(s)
result = ''.join(parts)  # ✅ 线性复杂度

# ✅ 或使用生成器
result = ''.join(s for s in items)
```

### 日志规范

```python
# ✅ 日志第一个参数必须是字符串字面量（含 % 占位符）
logger.info('User %s logged in', username)  # ✅

# ❌ 不要用 f-string
logger.info(f'User {username} logged in')   # ❌ 日志系统无法收集
```

---

## 六、其他重要规则

### 分号与括号

```python
# ❌ 不要加分号
x = 1; y = 2;                     # ❌

# ❌ 不要用括号连接语句
if x == 1: do_something();        # ❌

# ✅ 宁缺毋滥使用括号
if x == 1:
    do_something()                # ✅
```

### 主程序

```python
# ✅ 主要功能放在 main() 函数
def main():
    ...

if __name__ == '__main__':
    main()
```

### 资源管理

```python
# ✅ 使用 with 语句
with open('file.txt') as f:
    content = f.read()

# ❌ 避免手动关闭
f = open('file.txt')
content = f.read()
f.close()                        # ❌ 可能遗漏
```

### 类型注解

```python
# ✅ 公开 API 至少添加类型注解
def greet(name: str) -> str:
    return f'Hello, {name}!'

# ✅ Python 3.9+ 使用内置泛型
def process_items(items: list[int]) -> dict[str, int]:  # ✅

# ❌ 避免使用泛型别名
from typing import List, Dict      # ❌ Python 3.9+ 不推荐
def process(items: List[int]) -> Dict[str, int]:

# ✅ 使用 | 替代 Optional
def func(x: str | None) -> int:   # ✅
```

---

## AI 编程调用示例

```markdown
请遵循 Google Python 风格规范：
- 4 空格缩进，禁止 Tab
- 每行最大 80 字符
- 命名：模块 lower_with_under，类 CapWords，函数 lower_with_under
- 使用 f-string 格式化字符串
- 避免循环内用 + 拼接字符串
- 公开 API 添加类型注解
- 使用 with 语句管理资源
- 文档字符串用三重引号
```
