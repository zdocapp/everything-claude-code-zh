---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 Go 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **gofmt/goimports**：编辑后自动格式化 `.go` 文件
* **go vet**：编辑 `.go` 文件后运行静态分析
* **staticcheck**：对修改过的包运行扩展静态检查
