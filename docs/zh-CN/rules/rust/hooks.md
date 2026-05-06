---
paths:
  - "**/*.rs"
  - "**/Cargo.toml"
---

# Rust 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 Rust 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **cargo fmt**: 编辑后自动格式化 `.rs` 文件
* **cargo clippy**: 编辑 Rust 文件后运行代码检查
* **cargo check**: 变更后验证编译（比 `cargo build` 更快）
