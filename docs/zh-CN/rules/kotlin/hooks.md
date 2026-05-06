---
paths:
  - "**/*.kt"
  - "**/*.kts"
  - "**/build.gradle.kts"
---

# Kotlin Hooks

> 此文件扩展了 [common/hooks.md](../common/hooks.md)，添加了 Kotlin 特定的内容。

## PostToolUse Hooks

在 `~/.claude/settings.json` 中配置：

* **ktfmt/ktlint**：编辑后自动格式化 `.kt` 和 `.kts` 文件
* **detekt**：编辑 Kotlin 文件后运行静态分析
* **./gradlew build**：更改后验证编译
