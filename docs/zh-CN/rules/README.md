# 规则

## 结构

规则组织为 **通用** 层加上 **语言特定** 目录：

```
rules/
├── common/          # 语言无关原则（始终安装）
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   └── security.md
├── typescript/      # TypeScript/JavaScript 特定
├── python/          # Python 特定
├── golang/          # Go 特定
├── web/             # Web 和前端特定
├── swift/           # Swift 特定
└── php/             # PHP 特定
```

* **common/** 包含通用原则 —— 没有语言特定的代码示例。
* **语言目录** 通过框架特定的模式、工具和代码示例扩展通用规则。每个文件都引用其对应的通用规则文件。

## 安装

### 选项 1：安装脚本（推荐）

```bash
# Install common + one or more language-specific rule sets
./install.sh typescript
./install.sh python
./install.sh golang
./install.sh web
./install.sh swift
./install.sh php

# Install multiple languages at once
./install.sh typescript python
```

### 选项 2：手动安装

> **重要：** 复制整个目录 —— 请勿使用 `/*` 将其扁平化。
> 通用目录和语言特定目录包含同名文件。
> 将它们扁平化到一个目录中会导致语言特定文件覆盖通用规则，并破坏语言特定文件使用的相对 `../common/` 引用。

```bash
# Install common rules (required for all projects)
cp -r rules/common ~/.claude/rules/common

# Install language-specific rules based on your project's tech stack
cp -r rules/typescript ~/.claude/rules/typescript
cp -r rules/python ~/.claude/rules/python
cp -r rules/golang ~/.claude/rules/golang
cp -r rules/web ~/.claude/rules/web
cp -r rules/swift ~/.claude/rules/swift
cp -r rules/php ~/.claude/rules/php

# Attention ! ! ! Configure according to your actual project requirements; the configuration here is for reference only.
```

## 规则与技能

* **规则** 定义广泛适用的标准、约定和检查清单（例如，“80% 的测试覆盖率”、“无硬编码的密钥”）。
* **技能**（`skills/` 目录）为特定任务提供深入、可操作的参考材料（例如，`python-patterns`、`golang-testing`）。

语言特定的规则文件在适当的地方引用相关技能。规则告诉你*要做什么*；技能告诉你*如何去做*。

## 添加新语言

要添加对新语言（例如 `rust/`）的支持：

1. 创建一个 `rules/rust/` 目录
2. 添加扩展通用规则的文件：
   * `coding-style.md` —— 格式化工具、惯用法、错误处理模式
   * `testing.md` —— 测试框架、覆盖率工具、测试组织
   * `patterns.md` —— 语言特定的设计模式
   * `hooks.md` —— 用于格式化程序、linter、类型检查器的 PostToolUse 钩子
   * `security.md` —— 密钥管理、安全扫描工具
3. 每个文件应以以下内容开头：
   ```
   > 本文件通过 <语言> 特定内容扩展了 [common/xxx.md](../common/xxx.md)。
   ```
4. 如果现有技能可用，则引用它们，或者在 `skills/` 下创建新技能。

对于非语言领域，如 `web/`，当有足够多的可重用领域特定指导来证明独立的规则集合理时，遵循相同的分层模式。

## 规则优先级

当语言特定规则与通用规则冲突时，**语言特定规则优先**（具体覆盖通用）。这遵循标准的分层配置模式（类似于 CSS 特异性或 `.gitignore` 优先级）。

* `rules/common/` 定义了适用于所有项目的通用默认值。
* `rules/golang/`、`rules/python/`、`rules/swift/`、`rules/php/`、`rules/typescript/` 等在语言惯用法不同的地方覆盖这些默认值。

### 示例

`common/coding-style.md` 建议将不可变性作为默认原则。语言特定的 `golang/coding-style.md` 可以覆盖此规则：

> 惯用的 Go 使用指针接收器进行结构体变更 —— 有关通用原则，请参阅 [common/coding-style.md](../../../common/coding-style.md)，但此处优先使用 Go 惯用的变更方式。

### 带有覆盖说明的通用规则

`rules/common/` 中可能被语言特定文件覆盖的规则标记为：

> **语言说明：** 对于此模式不符合惯用法的语言，此规则可能被语言特定规则覆盖。
