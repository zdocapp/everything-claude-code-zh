---
paths:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
---

# Java 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 Java 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **google-java-format**：在编辑后自动格式化 `.java` 文件
* **checkstyle**：在编辑 Java 文件后运行代码风格检查
* **./mvnw compile** 或 **./gradlew compileJava**：在更改后验证编译
