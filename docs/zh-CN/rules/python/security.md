---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 安全

> 本文档在 [common/security.md](../common/security.md) 的基础上扩展了 Python 相关的内容。

## 密钥管理

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # Raises KeyError if missing
```

## 安全扫描

* 使用 **bandit** 进行静态安全分析：
  ```bash
  bandit -r src/
  ```

## 参考

如需 Django 特定的安全指南（如适用），请参阅技能：`django-security`。
