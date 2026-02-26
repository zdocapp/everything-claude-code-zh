**语言：** 英语 | [繁體中文](../zh-TW/README.md) | [简体中文](../../README.md)

# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash\&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript\&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python\&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go\&logoColor=white)
![Java](https://img.shields.io/badge/-Java-ED8B00?logo=openjdk\&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown\&logoColor=white)

> **42K+ stars** | **5K+ forks** | **24 contributors** | **6 languages supported** | **Anthropic Hackathon Winner**

***

<div align="center">

**🌐 语言 / 语言 / 語言**

[**English**](../../README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md)

</div>

***

**Anthropic 黑客马拉松获胜者提供的完整 Claude Code 配置集合。**

经过 10 多个月的密集日常使用，在构建真实产品的过程中演化出的生产就绪的智能体、技能、钩子、命令、规则和 MCP 配置。

***

## 指南

此仓库仅包含原始代码。指南解释了一切。

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="The Shorthand Guide to Everything Claude Code" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="The Longform Guide to Everything Claude Code" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>Shorthand Guide</b><br/>Setup, foundations, philosophy. <b>Read this first.</b></td>
<td align="center"><b>Longform Guide</b><br/>Token optimization, memory persistence, evals, parallelization.</td>
</tr>
</table>

| 主题 | 你将学到什么 |
|-------|-------------------|
| 令牌优化 | 模型选择，系统提示精简，后台进程 |
| 内存持久化 | 自动跨会话保存/加载上下文的钩子 |
| 持续学习 | 从会话中自动提取模式为可重用技能 |
| 验证循环 | 检查点与持续评估，评分器类型，pass@k 指标 |
| 并行化 | Git 工作树，级联方法，何时扩展实例 |
| 子智能体编排 | 上下文问题，迭代检索模式 |

***

## 最新动态

### v1.4.1 — 错误修复 (2026年2月)

* **修复了直觉导入内容丢失问题** — `parse_instinct_file()` 在 `/instinct-import` 期间会静默丢弃 frontmatter 之后的所有内容（Action, Evidence, Examples 部分）。已由社区贡献者 @ericcai0814 修复 ([#148](https://github.com/affaan-m/everything-claude-code/issues/148), [#161](https://github.com/affaan-m/everything-claude-code/pull/161))

### v1.4.0 — 多语言规则、安装向导 & PM2 (2026年2月)

* **交互式安装向导** — 新的 `configure-ecc` 技能提供了带有合并/覆盖检测的引导式设置
* **PM2 & 多智能体编排** — 6 个新命令 (`/pm2`, `/multi-plan`, `/multi-execute`, `/multi-backend`, `/multi-frontend`, `/multi-workflow`) 用于管理复杂的多服务工作流
* **多语言规则架构** — 规则从扁平文件重组为 `common/` + `typescript/` + `python/` + `golang/` 目录。仅安装您需要的语言
* **中文 (zh-CN) 翻译** — 所有智能体、命令、技能和规则的完整翻译 (80+ 个文件)
* **GitHub Sponsors 支持** — 通过 GitHub Sponsors 赞助项目
* **增强的 CONTRIBUTING.md** — 针对每种贡献类型的详细 PR 模板

### v1.3.0 — OpenCode 插件支持 (2026年2月)

* **完整的 OpenCode 集成** — 12 个智能体，24 个命令，16 个技能，通过 OpenCode 的插件系统支持钩子 (20+ 种事件类型)
* **3 个原生自定义工具** — run-tests, check-coverage, security-audit
* **LLM 文档** — `llms.txt` 用于获取全面的 OpenCode 文档

### v1.2.0 — 统一的命令和技能 (2026年2月)

* **Python/Django 支持** — Django 模式、安全、TDD 和验证技能
* **Java Spring Boot 技能** — Spring Boot 的模式、安全、TDD 和验证
* **会话管理** — `/sessions` 命令用于查看会话历史
* **持续学习 v2** — 基于直觉的学习，带有置信度评分、导入/导出、进化

完整的更新日志请参见 [Releases](https://github.com/affaan-m/everything-claude-code/releases)。

***

## 🚀 快速开始

在 2 分钟内启动并运行：

### 步骤 1：安装插件

```bash
# Add marketplace
/plugin marketplace add affaan-m/everything-claude-code

# Install plugin
/plugin install everything-claude-code@everything-claude-code
```

### 步骤 2：安装规则（必需）

> ⚠️ **重要提示：** Claude Code 插件无法自动分发 `rules`。请手动安装它们：

```bash
# Clone the repo first
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# Recommended: use the installer (handles common + language rules safely)
./install.sh typescript    # or python or golang
# You can pass multiple languages:
# ./install.sh typescript python golang
# or target cursor:
# ./install.sh --target cursor typescript
```

手动安装说明请参阅 `rules/` 文件夹中的 README。

### 步骤 3：开始使用

```bash
# Try a command
/plan "Add user authentication"

# Check available commands
/plugin list everything-claude-code@everything-claude-code
```

✨ **就这样！** 你现在可以访问 13 个代理、37 个技能和 31 个命令。

***

## 🌐 跨平台支持

此插件现已完全支持 **Windows、macOS 和 Linux**。所有钩子和脚本都已用 Node.js 重写，以实现最大的兼容性。

### 包管理器检测

插件会自动检测您首选的包管理器（npm、pnpm、yarn 或 bun），优先级如下：

1. **环境变量**：`CLAUDE_PACKAGE_MANAGER`
2. **项目配置**：`.claude/package-manager.json`
3. **package.json**：`packageManager` 字段
4. **锁文件**：从 package-lock.json、yarn.lock、pnpm-lock.yaml 或 bun.lockb 检测
5. **全局配置**：`~/.claude/package-manager.json`
6. **回退方案**：第一个可用的包管理器

要设置您首选的包管理器：

```bash
# Via environment variable
export CLAUDE_PACKAGE_MANAGER=pnpm

# Via global config
node scripts/setup-package-manager.js --global pnpm

# Via project config
node scripts/setup-package-manager.js --project bun

# Detect current setting
node scripts/setup-package-manager.js --detect
```

或者在 Claude Code 中使用 `/setup-pm` 命令。

***

## 📦 包含内容

此仓库是一个 **Claude Code 插件** - 可以直接安装或手动复制组件。

```
everything-claude-code/
|-- .claude-plugin/   # 插件和市场清单
|   |-- plugin.json         # 插件元数据和组件路径
|   |-- marketplace.json    # 用于 /plugin marketplace add 的市场目录
|
|-- agents/           # 用于委托的专用子代理
|   |-- planner.md           # 功能实现规划
|   |-- architect.md         # 系统设计决策
|   |-- tdd-guide.md         # 测试驱动开发
|   |-- code-reviewer.md     # 质量与安全审查
|   |-- security-reviewer.md # 漏洞分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright 端到端测试
|   |-- refactor-cleaner.md  # 无用代码清理
|   |-- doc-updater.md       # 文档同步
|   |-- go-reviewer.md       # Go 代码审查
|   |-- go-build-resolver.md # Go 构建错误修复
|   |-- python-reviewer.md   # Python 代码审查（新增）
|   |-- database-reviewer.md # 数据库/Supabase 审查（新增）
|
|-- skills/           # 工作流定义和领域知识
|   |-- coding-standards/           # 语言最佳实践
|   |-- clickhouse-io/              # ClickHouse 分析、查询与数据工程
|   |-- backend-patterns/           # API、数据库、缓存模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从会话中自动提取模式（长文指南）
|   |-- continuous-learning-v2/     # 基于直觉的学习与置信度评分
|   |-- iterative-retrieval/        # 子代理渐进式上下文优化
|   |-- strategic-compact/          # 手动压缩建议（长文指南）
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- eval-harness/               # 验证循环评估（长文指南）
|   |-- verification-loop/          # 持续验证（长文指南）
|   |-- golang-patterns/            # Go 习惯用法和最佳实践
|   |-- golang-testing/             # Go 测试模式、TDD、基准测试
|   |-- cpp-testing/                # 使用 GoogleTest 与 CMake/CTest 的 C++ 测试（新增）
|   |-- django-patterns/            # Django 模式、模型与视图（新增）
|   |-- django-security/            # Django 安全最佳实践（新增）
|   |-- django-tdd/                 # Django TDD 工作流（新增）
|   |-- django-verification/        # Django 验证循环（新增）
|   |-- python-patterns/            # Python 习惯用法和最佳实践（新增）
|   |-- python-testing/             # 使用 pytest 的 Python 测试（新增）
|   |-- springboot-patterns/        # Java Spring Boot 模式（新增）
|   |-- springboot-security/        # Spring Boot 安全（新增）
|   |-- springboot-tdd/             # Spring Boot TDD（新增）
|   |-- springboot-verification/    # Spring Boot 验证（新增）
|   |-- configure-ecc/              # 交互式安装向导（新增）
|   |-- security-scan/              # AgentShield 安全审计集成（新增）
|   |-- java-coding-standards/     # Java 编码规范（新增）
|   |-- jpa-patterns/              # JPA/Hibernate 模式（新增）
|   |-- postgres-patterns/         # PostgreSQL 优化模式（新增）
|   |-- nutrient-document-processing/ # 使用 Nutrient API 的文档处理（新增）
|   |-- project-guidelines-example/   # 项目专用技能模板
|   |-- database-migrations/         # 迁移模式（Prisma、Drizzle、Django、Go）（新增）
|   |-- api-design/                  # REST API 设计、分页与错误响应（新增）
|   |-- deployment-patterns/         # CI/CD、Docker、健康检查与回滚（新增）
|   |-- docker-patterns/            # Docker Compose、网络、卷与容器安全（新增）
|   |-- e2e-testing/                 # Playwright 端到端模式与页面对象模型（新增）
|
|-- commands/         # 快速执行的斜杠命令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现规划
|   |-- e2e.md              # /e2e - 端到端测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 修复构建错误
|   |-- refactor-clean.md   # /refactor-clean - 无用代码清理
|   |-- learn.md            # /learn - 会话中提取模式（长文指南）
|   |-- checkpoint.md       # /checkpoint - 保存验证状态（长文指南）
|   |-- verify.md           # /verify - 执行验证循环（长文指南）
|   |-- setup-pm.md         # /setup-pm - 配置包管理器
|   |-- go-review.md        # /go-review - Go 代码审查（新增）
|   |-- go-test.md          # /go-test - Go TDD 工作流（新增）
|   |-- go-build.md         # /go-build - 修复 Go 构建错误（新增）
|   |-- skill-create.md     # /skill-create - 从 git 历史生成技能（新增）
|   |-- instinct-status.md  # /instinct-status - 查看学习到的直觉（新增）
|   |-- instinct-import.md  # /instinct-import - 导入直觉（新增）
|   |-- instinct-export.md  # /instinct-export - 导出直觉（新增）
|   |-- evolve.md           # /evolve - 将直觉聚类为技能
|   |-- pm2.md              # /pm2 - PM2 服务生命周期管理（新增）
|   |-- multi-plan.md       # /multi-plan - 多代理任务拆解（新增）
|   |-- multi-execute.md    # /multi-execute - 编排的多代理工作流（新增）
|   |-- multi-backend.md    # /multi-backend - 后端多服务编排（新增）
|   |-- multi-frontend.md   # /multi-frontend - 前端多服务编排（新增）
|   |-- multi-workflow.md   # /multi-workflow - 通用多服务工作流（新增）
|   |-- orchestrate.md      # /orchestrate - 多代理协调
|   |-- sessions.md         # /sessions - 会话历史管理
|   |-- eval.md             # /eval - 按标准评估
|   |-- test-coverage.md    # /test-coverage - 测试覆盖率分析
|   |-- update-docs.md      # /update-docs - 更新文档
|   |-- update-codemaps.md  # /update-codemaps - 更新代码映射
|   |-- python-review.md    # /python-review - Python 代码审查（新增）
|
|-- rules/            # 必须遵循的规则（复制到 ~/.claude/rules/）
|   |-- README.md            # 结构说明与安装指南
|   |-- common/              # 与语言无关的原则
|   |   |-- coding-style.md    # 不可变性与文件组织
|   |   |-- git-workflow.md    # 提交格式与 PR 流程
|   |   |-- testing.md         # TDD 与 80% 覆盖率要求
|   |   |-- performance.md     # 模型选择与上下文管理
|   |   |-- patterns.md        # 设计模式与骨架项目
|   |   |-- hooks.md           # Hook 架构与 TodoWrite
|   |   |-- agents.md          # 何时委托给子代理
|   |   |-- security.md        # 强制安全检查
|   |-- typescript/          # TypeScript/JavaScript 专用
|   |-- python/              # Python 专用
|   |-- golang/              # Go 专用
|
|-- hooks/            # 基于触发器的自动化
|   |-- README.md                 # Hook 文档、示例与自定义指南
|   |-- hooks.json                # 所有 Hook 配置（PreToolUse、PostToolUse、Stop 等）
|   |-- memory-persistence/       # 会话生命周期 Hook（长文指南）
|   |-- strategic-compact/        # 压缩建议（长文指南）
|
|-- scripts/          # 跨平台 Node.js 脚本（新增）
|   |-- lib/                     # 公共工具
|   |   |-- utils.js             # 跨平台文件/路径/系统工具
|   |   |-- package-manager.js   # 包管理器检测与选择
|   |-- hooks/                   # Hook 实现
|   |   |-- session-start.js     # 会话开始时加载上下文
|   |   |-- session-end.js       # 会话结束时保存状态
|   |   |-- pre-compact.js       # 压缩前状态保存
|   |   |-- suggest-compact.js   # 战略压缩建议
|   |   |-- evaluate-session.js  # 从会话提取模式
|   |-- setup-package-manager.js # 交互式包管理器设置
|
|-- tests/            # 测试套件（新增）
|   |-- lib/                     # 库测试
|   |-- hooks/                   # Hook 测试
|   |-- run-all.js               # 运行所有测试
|
|-- contexts/         # 动态系统提示上下文（长文指南）
|   |-- dev.md              # 开发模式上下文
|   |-- review.md           # 代码审查模式上下文
|   |-- research.md         # 研究/探索模式上下文
|
|-- examples/         # 示例配置与会话
|   |-- CLAUDE.md             # 项目级配置示例
|   |-- user-CLAUDE.md        # 用户级配置示例
|   |-- saas-nextjs-CLAUDE.md   # 实际 SaaS 示例（Next.js + Supabase + Stripe）
|   |-- go-microservice-CLAUDE.md # 实际 Go 微服务示例（gRPC + PostgreSQL）
|   |-- django-api-CLAUDE.md      # 实际 Django REST API 示例（DRF + Celery）
|   |-- rust-api-CLAUDE.md        # 实际 Rust API 示例（Axum + SQLx + PostgreSQL）（新增）
|
|-- mcp-configs/      # MCP 服务器配置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- marketplace.json  # 自托管市场配置（用于 /plugin marketplace add）
```

***

## 🛠️ 生态系统工具

### 技能创建器

从您的仓库生成 Claude Code 技能的两种方式：

#### 选项 A：本地分析（内置）

使用 `/skill-create` 命令进行本地分析，无需外部服务：

```bash
/skill-create                    # Analyze current repo
/skill-create --instincts        # Also generate instincts for continuous-learning
```

这会在本地分析您的 git 历史记录并生成 SKILL.md 文件。

#### 选项 B：GitHub 应用（高级）

适用于高级功能（10k+ 提交、自动 PR、团队共享）：

[安装 GitHub 应用](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# Comment on any issue:
/skill-creator analyze

# Or auto-triggers on push to default branch
```

两种选项都会创建：

* **SKILL.md 文件** - 可供 Claude Code 使用的即用型技能
* **Instinct 集合** - 用于 continuous-learning-v2
* **模式提取** - 从您的提交历史中学习

### AgentShield — 安全审计器

> 在 Claude Code 黑客松（Cerebral Valley x Anthropic，2026 年 2 月）上构建。912 个测试，98% 覆盖率，102 条静态分析规则。

扫描您的 Claude Code 配置，查找漏洞、错误配置和注入风险。

```bash
# Quick scan (no install needed)
npx ecc-agentshield scan

# Auto-fix safe issues
npx ecc-agentshield scan --fix

# Deep analysis with three Opus 4.6 agents
npx ecc-agentshield scan --opus --stream

# Generate secure config from scratch
npx ecc-agentshield init
```

**它扫描什么：** CLAUDE.md、settings.json、MCP 配置、钩子、代理定义以及 5 个类别的技能 —— 密钥检测（14 种模式）、权限审计、钩子注入分析、MCP 服务器风险剖析和代理配置审查。

**`--opus` 标志** 在红队/蓝队/审计员管道中运行三个 Claude Opus 4.6 代理。攻击者寻找利用链，防御者评估保护措施，审计员将两者综合成优先风险评估。对抗性推理，而不仅仅是模式匹配。

**输出格式：** 终端（按颜色分级的 A-F）、JSON（CI 管道）、Markdown、HTML。在关键发现时退出代码 2，用于构建门控。

在 Claude Code 中使用 `/security-scan` 来运行它，或者通过 [GitHub Action](https://github.com/affaan-m/agentshield) 添加到 CI。

[GitHub](https://github.com/affaan-m/agentshield) | [npm](https://www.npmjs.com/package/ecc-agentshield)

### 🧠 持续学习 v2

基于本能的学习系统会自动学习您的模式：

```bash
/instinct-status        # Show learned instincts with confidence
/instinct-import <file> # Import instincts from others
/instinct-export        # Export your instincts for sharing
/evolve                 # Cluster related instincts into skills
```

完整文档请参阅 `skills/continuous-learning-v2/`。

***

## 📋 要求

### Claude Code CLI 版本

**最低版本：v2.1.0 或更高版本**

此插件需要 Claude Code CLI v2.1.0+，因为插件系统处理钩子的方式发生了变化。

检查您的版本：

```bash
claude --version
```

### 重要提示：钩子自动加载行为

> ⚠️ **对于贡献者：** 请勿向 `.claude-plugin/plugin.json` 添加 `"hooks"` 字段。这由回归测试强制执行。

Claude Code v2.1+ **会自动加载** 任何已安装插件中的 `hooks/hooks.json`（按约定）。在 `plugin.json` 中显式声明会导致重复检测错误：

```
Duplicate hooks file detected: ./hooks/hooks.json resolves to already-loaded file
```

**历史背景：** 这已导致此仓库中多次修复/还原循环（[#29](https://github.com/affaan-m/everything-claude-code/issues/29), [#52](https://github.com/affaan-m/everything-claude-code/issues/52), [#103](https://github.com/affaan-m/everything-claude-code/issues/103)）。Claude Code 版本之间的行为发生了变化，导致了混淆。我们现在有一个回归测试来防止这种情况再次发生。

***

## 📥 安装

### 选项 1：作为插件安装（推荐）

使用此仓库的最简单方式 - 作为 Claude Code 插件安装：

```bash
# Add this repo as a marketplace
/plugin marketplace add affaan-m/everything-claude-code

# Install the plugin
/plugin install everything-claude-code@everything-claude-code
```

或者直接添加到您的 `~/.claude/settings.json`：

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

这将使您能够立即访问所有命令、代理、技能和钩子。

> **注意：** Claude Code 插件系统不支持通过插件分发 `rules`（[上游限制](https://code.claude.com/docs/en/plugins-reference)）。您需要手动安装规则：
>
> ```bash
> # 首先克隆仓库
> git clone https://github.com/affaan-m/everything-claude-code.git
>
> # 选项 A：用户级规则（适用于所有项目）
> mkdir -p ~/.claude/rules
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择您的技术栈
> cp -r everything-claude-code/rules/python/* ~/.claude/rules/
> cp -r everything-claude-code/rules/golang/* ~/.claude/rules/
>
> # 选项 B：项目级规则（仅适用于当前项目）
> mkdir -p .claude/rules
> cp -r everything-claude-code/rules/common/* .claude/rules/
> cp -r everything-claude-code/rules/typescript/* .claude/rules/     # 选择您的技术栈
> ```

***

### 🔧 选项 2：手动安装

如果您希望对安装的内容进行手动控制：

```bash
# Clone the repo
git clone https://github.com/affaan-m/everything-claude-code.git

# Copy agents to your Claude config
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Copy rules (common + language-specific)
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # pick your stack
cp -r everything-claude-code/rules/python/* ~/.claude/rules/
cp -r everything-claude-code/rules/golang/* ~/.claude/rules/

# Copy commands
cp everything-claude-code/commands/*.md ~/.claude/commands/

# Copy skills
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### 将钩子添加到 settings.json

将 `hooks/hooks.json` 中的钩子复制到你的 `~/.claude/settings.json`。

#### 配置 MCPs

将 `mcp-configs/mcp-servers.json` 中所需的 MCP 服务器复制到你的 `~/.claude.json`。

**重要：** 将 `YOUR_*_HERE` 占位符替换为你实际的 API 密钥。

***

## 🎯 关键概念

### 智能体

子智能体处理具有有限范围的委托任务。示例：

```markdown
---
name: code-reviewer
description: 审查代码的质量、安全性和可维护性
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

您是一位资深代码审查员...

```

### 技能

技能是由命令或智能体调用的工作流定义：

```markdown
# TDD Workflow

1. Define interfaces first
2. Write failing tests (RED)
3. Implement minimal code (GREEN)
4. Refactor (IMPROVE)
5. Verify 80%+ coverage
```

### 钩子

钩子在工具事件上触发。示例 - 警告关于 console.log：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### 规则

规则是始终遵循的指导原则，组织成 `common/`（与语言无关）+ 语言特定目录：

```
rules/
  common/          # Universal principles (always install)
  typescript/      # TS/JS specific patterns and tools
  python/          # Python specific patterns and tools
  golang/          # Go specific patterns and tools
```

有关安装和结构详情，请参阅 [`rules/README.md`](../../rules/README.md)。

***

## 🗺️ 我应该使用哪个代理？

不确定从哪里开始？使用这个快速参考：

| 我想要... | 使用此命令 | 使用的代理 |
|--------------|-----------------|------------|
| 规划新功能 | `/plan "Add auth"` | planner |
| 设计系统架构 | `/plan` + architect agent | architect |
| 先编写代码和测试 | `/tdd` | tdd-guide |
| 审查我刚写的代码 | `/code-review` | code-reviewer |
| 修复失败的构建 | `/build-fix` | build-error-resolver |
| 运行端到端测试 | `/e2e` | e2e-runner |
| 查找安全漏洞 | `/security-scan` | security-reviewer |
| 移除死代码 | `/refactor-clean` | refactor-cleaner |
| 更新文档 | `/update-docs` | doc-updater |
| 审查 Go 代码 | `/go-review` | go-reviewer |
| 审查 Python 代码 | `/python-review` | python-reviewer |
| 审计数据库查询 | *(自动委派)* | database-reviewer |

### 常见工作流

**开始新功能：**

```
/plan "Add user authentication with OAuth"   → planner creates implementation blueprint
/tdd                                          → tdd-guide enforces write-tests-first
/code-review                                  → code-reviewer checks your work
```

**修复错误：**

```
/tdd                                          → tdd-guide: write a failing test that reproduces it
                                              → implement the fix, verify test passes
/code-review                                  → code-reviewer: catch regressions
```

**准备生产环境：**

```
/security-scan                                → security-reviewer: OWASP Top 10 audit
/e2e                                          → e2e-runner: critical user flow tests
/test-coverage                                → verify 80%+ coverage
```

***

## ❓ 常见问题

<details>
<summary><b>How do I check which agents/commands are installed?</b></summary>

```bash
/plugin list everything-claude-code@everything-claude-code
```

这会显示插件中所有可用的代理、命令和技能。

</details>

<details>
<summary><b>My hooks aren't working / I see "Duplicate hooks file" errors</b></summary>

这是最常见的问题。**不要在 `.claude-plugin/plugin.json` 中添加 `"hooks"` 字段。** Claude Code v2.1+ 会自动从已安装的插件加载 `hooks/hooks.json`。显式声明它会导致重复检测错误。参见 [#29](https://github.com/affaan-m/everything-claude-code/issues/29), [#52](https://github.com/affaan-m/everything-claude-code/issues/52), [#103](https://github.com/affaan-m/everything-claude-code/issues/103)。

</details>

<details>
<summary><b>My context window is shrinking / Claude is running out of context</b></summary>

太多的 MCP 服务器会消耗你的上下文。每个 MCP 工具描述都会消耗你 200k 窗口的令牌，可能将其减少到约 70k。

**修复：** 按项目禁用未使用的 MCP：

```json
// In your project's .claude/settings.json
{
  "disabledMcpServers": ["supabase", "railway", "vercel"]
}
```

保持启用的 MCP 少于 10 个，活动工具少于 80 个。

</details>

<details>
<summary><b>Can I use only some components (e.g., just agents)?</b></summary>

是的。使用选项 2（手动安装）并仅复制你需要的部分：

```bash
# Just agents
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Just rules
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
```

每个组件都是完全独立的。

</details>

<details>
<summary><b>Does this work with Cursor / OpenCode?</b></summary>

是的。ECC 是跨平台的：

* **Cursor**：`.cursor/` 中的预翻译配置。参见 [Cursor IDE 支持](#cursor-ide-支持)。
* **OpenCode**：`.opencode/` 中的完整插件支持。参见 [OpenCode 支持](#-opencode-支持)。
* **Claude Code**：原生支持 —— 这是主要目标。

</details>

<details>
<summary><b>How do I contribute a new skill or agent?</b></summary>

参见 [CONTRIBUTING.md](CONTRIBUTING.md)。简短版本：

1. Fork 仓库
2. 在 `skills/your-skill-name/SKILL.md` 中创建你的技能（带有 YAML 前言）
3. 或在 `agents/your-agent.md` 中创建代理
4. 提交 PR，清晰描述其功能和使用时机

</details>

***

## 🧪 运行测试

该插件包含一个全面的测试套件：

```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

***

## 🤝 贡献

**欢迎并鼓励贡献。**

此仓库旨在成为社区资源。如果你有：

* 有用的智能体或技能
* 巧妙的钩子
* 更好的 MCP 配置
* 改进的规则

请贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

### 贡献想法

* 特定语言技能 (Rust, C#, Swift, Kotlin) — Go, Python, Java 已包含
* 特定框架配置 (Rails, Laravel, FastAPI, NestJS) — Django, Spring Boot 已包含
* DevOps 智能体 (Kubernetes, Terraform, AWS, Docker)
* 测试策略 (不同框架，视觉回归)
* 领域特定知识 (ML, 数据工程, 移动端)

***

## Cursor IDE 支持

ecc-universal 包含为 [Cursor IDE](https://cursor.com) 预翻译的配置。`.cursor/` 目录包含适用于 Cursor 格式的规则、智能体、技能、命令和 MCP 配置。

### 快速开始 (Cursor)

```bash
# Install the package
npm install ecc-universal

# Install for your language(s)
./install.sh --target cursor typescript
./install.sh --target cursor python golang
```

### 已翻译内容

| 组件 | Claude Code → Cursor | 对等性 |
|-----------|---------------------|--------|
| 规则 | 添加了 YAML frontmatter，路径扁平化 | 完全 |
| 智能体 | 模型 ID 已扩展，工具 → 只读标志 | 完全 |
| 技能 | 无需更改 (标准相同) | 相同 |
| 命令 | 路径引用已更新，多-\* 已存根 | 部分 |
| MCP 配置 | 环境变量插值语法已更新 | 完全 |
| 钩子 | Cursor 中无等效项 | 参见替代方案 |

详情请参阅 [.cursor/README.md](../../.cursor/README.md)，完整迁移指南请参阅 [.cursor/MIGRATION.md](../../.cursor/MIGRATION.md)。

***

## 🔌 OpenCode 支持

ECC 提供 **完整的 OpenCode 支持**，包括插件和钩子。

### 快速开始

```bash
# Install OpenCode
npm install -g opencode

# Run in the repository root
opencode
```

配置会自动从 `.opencode/opencode.json` 检测。

### 功能对等

| 功能 | Claude Code | OpenCode | 状态 |
|---------|-------------|----------|--------|
| 代理 | ✅ 13 个代理 | ✅ 12 个代理 | **Claude Code 领先** |
| 命令 | ✅ 31 个命令 | ✅ 24 个命令 | **Claude Code 领先** |
| 技能 | ✅ 37 个技能 | ✅ 16 个技能 | **Claude Code 领先** |
| 钩子 | ✅ 3 个阶段 | ✅ 20+ 个事件 | **OpenCode 更多！** |
| 规则 | ✅ 8 条规则 | ✅ 8 条规则 | **完全对等** |
| MCP 服务器 | ✅ 完整 | ✅ 完整 | **完全对等** |
| 自定义工具 | ✅ 通过钩子 | ✅ 原生支持 | **OpenCode 更好** |

### 通过插件实现的钩子支持

OpenCode 的插件系统比 Claude Code 更复杂，有 20 多种事件类型：

| Claude Code 钩子 | OpenCode 插件事件 |
|-----------------|----------------------|
| PreToolUse | `tool.execute.before` |
| PostToolUse | `tool.execute.after` |
| Stop | `session.idle` |
| SessionStart | `session.created` |
| SessionEnd | `session.deleted` |

**额外的 OpenCode 事件**：`file.edited`、`file.watcher.updated`、`message.updated`、`lsp.client.diagnostics`、`tui.toast.show` 等等。

### 可用命令 (31)

| 命令 | 描述 |
|---------|-------------|
| `/plan` | 创建实施计划 |
| `/tdd` | 强制执行 TDD 工作流 |
| `/code-review` | 审查代码变更 |
| `/build-fix` | 修复构建错误 |
| `/e2e` | 生成 E2E 测试 |
| `/refactor-clean` | 移除死代码 |
| `/orchestrate` | 多代理工作流 |
| `/learn` | 从会话中提取模式 |
| `/checkpoint` | 保存验证状态 |
| `/verify` | 运行验证循环 |
| `/eval` | 根据标准进行评估 |
| `/update-docs` | 更新文档 |
| `/update-codemaps` | 更新代码地图 |
| `/test-coverage` | 分析覆盖率 |
| `/go-review` | Go 代码审查 |
| `/go-test` | Go TDD 工作流 |
| `/go-build` | 修复 Go 构建错误 |
| `/python-review` | Python 代码审查（PEP 8、类型提示、安全性） |
| `/multi-plan` | 多模型协作规划 |
| `/multi-execute` | 多模型协作执行 |
| `/multi-backend` | 后端聚焦的多模型工作流 |
| `/multi-frontend` | 前端聚焦的多模型工作流 |
| `/multi-workflow` | 完整的多模型开发工作流 |
| `/pm2` | 自动生成 PM2 服务命令 |
| `/sessions` | 管理会话历史 |
| `/skill-create` | 从 git 生成技能 |
| `/instinct-status` | 查看已学习的本能 |
| `/instinct-import` | 导入本能 |
| `/instinct-export` | 导出本能 |
| `/evolve` | 将本能聚类为技能 |
| `/setup-pm` | 配置包管理器 |

### 插件安装

**选项 1：直接使用**

```bash
cd everything-claude-code
opencode
```

**选项 2：作为 npm 包安装**

```bash
npm install ecc-universal
```

然后添加到您的 `opencode.json`：

```json
{
  "plugin": ["ecc-universal"]
}
```

### 文档

* **迁移指南**：`.opencode/MIGRATION.md`
* **OpenCode 插件 README**：`.opencode/README.md`
* **整合的规则**：`.opencode/instructions/INSTRUCTIONS.md`
* **LLM 文档**：`llms.txt`（完整的 OpenCode 文档，供 LLM 使用）

***

## 📖 背景

我从实验性推出以来就一直在使用 Claude Code。在 2025 年 9 月，与 [@DRodriguezFX](https://x.com/DRodriguezFX) 一起使用 Claude Code 构建 [zenith.chat](https://zenith.chat)，赢得了 Anthropic x Forum Ventures 黑客马拉松。

这些配置已在多个生产应用程序中经过实战测试。

***

## 令牌优化

如果不管理令牌消耗，使用 Claude Code 可能会很昂贵。这些设置能在不牺牲质量的情况下显著降低成本。

### 推荐设置

添加到 `~/.claude/settings.json`：

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| 设置 | 默认值 | 推荐值 | 影响 |
|---------|---------|-------------|--------|
| `model` | opus | **sonnet** | 约 60% 的成本降低；处理 80%+ 的编码任务 |
| `MAX_THINKING_TOKENS` | 31,999 | **10,000** | 每个请求的隐藏思考成本降低约 70% |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 95 | **50** | 更早压缩 —— 在长会话中质量更好 |

仅在需要深度架构推理时切换到 Opus：

```
/model opus
```

### 日常工作流命令

| 命令 | 何时使用 |
|---------|-------------|
| `/model sonnet` | 大多数任务的默认选择 |
| `/model opus` | 复杂架构、调试、深度推理 |
| `/clear` | 在不相关的任务之间（免费，即时重置） |
| `/compact` | 在逻辑任务断点处（研究完成，里程碑达成） |
| `/cost` | 在会话期间监控令牌花费 |

### 策略性压缩

`strategic-compact` 技能（包含在此插件中）建议在逻辑断点处进行 `/compact`，而不是依赖在 95% 上下文时的自动压缩。完整决策指南请参见 `skills/strategic-compact/SKILL.md`。

**何时压缩：**

* 研究/探索之后，实施之前
* 完成一个里程碑之后，开始下一个之前
* 调试之后，继续功能工作之前
* 失败的方法之后，尝试新方法之前

**何时不压缩：**

* 实施过程中（你会丢失变量名、文件路径、部分状态）

### 上下文窗口管理

**关键：** 不要一次性启用所有 MCP。每个 MCP 工具描述都会消耗你 200k 窗口的令牌，可能将其减少到约 70k。

* 每个项目保持启用的 MCP 少于 10 个
* 保持活动工具少于 80 个
* 在项目配置中使用 `disabledMcpServers` 来禁用未使用的 MCP

### 代理团队成本警告

代理团队会生成多个上下文窗口。每个团队成员独立消耗令牌。仅用于并行性能提供明显价值的任务（多模块工作、并行审查）。对于简单的顺序任务，子代理更节省令牌。

***

## ⚠️ 重要说明

### 令牌优化

达到每日限制？参见 **[令牌优化指南](../token-optimization.md)** 获取推荐设置和工作流提示。

快速见效的方法：

```json
// ~/.claude/settings.json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

在不相关的任务之间使用 `/clear`，在逻辑断点处使用 `/compact`，并使用 `/cost` 来监控花费。

### 定制化

这些配置适用于我的工作流。你应该：

1. 从引起共鸣的部分开始
2. 根据你的技术栈进行修改
3. 移除你不使用的部分
4. 添加你自己的模式

***

## 💜 赞助商

这个项目是免费和开源的。赞助商帮助保持其维护和发展。

[**成为赞助商**](https://github.com/sponsors/affaan-m) | [赞助商等级](SPONSORS.md)

***

## 🌟 Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code\&type=Date)](https://star-history.com/#affaan-m/everything-claude-code\&Date)

***

## 🔗 链接

* **速查指南 (从此开始):** [Claude Code 万事速查指南](https://x.com/affaanmustafa/status/2012378465664745795)
* **详细指南 (进阶):** [Claude Code 万事详细指南](https://x.com/affaanmustafa/status/2014040193557471352)
* **关注:** [@affaanmustafa](https://x.com/affaanmustafa)
* **zenith.chat:** [zenith.chat](https://zenith.chat)
* **技能目录:** awesome-agent-skills（社区维护的智能体技能目录）

***

## 📄 许可证

MIT - 自由使用，根据需要修改，如果可以请回馈贡献。

***

**如果此仓库对你有帮助，请点星。阅读两份指南。构建伟大的东西。**
