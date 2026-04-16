---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 Python 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **black/ruff**：编辑后自动格式化 `.py` 文件
* **mypy/pyright**：编辑 `.py` 文件后运行类型检查

## 警告

* 对编辑文件中的 `print()` 语句发出警告（请改用 `logging` 模块）
