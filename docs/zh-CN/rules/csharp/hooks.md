---
paths:
  - "**/*.cs"
  - "**/*.csx"
  - "**/*.csproj"
  - "**/*.sln"
  - "**/Directory.Build.props"
  - "**/Directory.Build.targets"
---

# C# 钩子

> 本文档在 [common/hooks.md](../common/hooks.md) 的基础上扩展了 C# 相关的内容。

## PostToolUse 钩子

在 `~/.claude/settings.json` 中配置：

* **dotnet format**：自动格式化编辑过的 C# 文件并应用分析器修复
* **dotnet build**：验证编辑后解决方案或项目是否仍能编译
* **dotnet test --no-build**：在行为更改后重新运行最相关的测试项目

## Stop 钩子

* 在结束涉及广泛 C# 更改的会话前，运行一次最终的 `dotnet build`
* 对修改过的 `appsettings*.json` 文件发出警告，以防密钥被提交
