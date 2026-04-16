---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 Swift 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **SwiftFormat**：编辑后自动格式化 `.swift` 文件
* **SwiftLint**：编辑 `.swift` 文件后运行代码检查
* **swift build**：编辑后对修改过的包进行类型检查

## 警告

标记 `print()` 语句 —— 在生产代码中请改用 `os.Logger` 或结构化日志记录。
