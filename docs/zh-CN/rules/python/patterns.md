---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 模式

> 本文档在 [common/patterns.md](../common/patterns.md) 的基础上扩展了 Python 特有的内容。

## 协议（鸭子类型）

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## 数据类作为 DTO

```python
from dataclasses import dataclass

@dataclass
class CreateUserRequest:
    name: str
    email: str
    age: int | None = None
```

## 上下文管理器与生成器

* 使用上下文管理器（`with` 语句）进行资源管理
* 使用生成器实现惰性求值和内存高效的迭代

## 参考

有关全面的模式（包括装饰器、并发和包组织），请参阅技能：`python-patterns`。
