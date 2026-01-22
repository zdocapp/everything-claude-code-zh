# Everything Claude Code

**Anthropic 黑客马拉松获胜者提供的完整 Claude Code 配置集合。**

经过 10 多个月的密集日常使用，在构建真实产品的过程中演化出的生产就绪的智能体、技能、钩子、命令、规则和 MCP 配置。

***

## 指南

此仓库仅包含原始代码。指南解释了一切。

### 从这里开始：简明指南

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)**

基础 - 每种配置类型的作用，如何构建您的设置，上下文窗口管理，以及这些配置背后的哲学。**请先阅读此指南。**

***

### 然后：详细指南

<img width="609" height="428" alt="image" src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" />

**[Everything Claude Code 详细指南](https://x.com/affaanmustafa/status/2014040193557471352)**

高级技术 - 令牌优化、跨会话内存持久化、验证循环与评估、并行化策略、子智能体编排和持续学习。本指南中的所有内容在此仓库中都有可运行的代码。

| 主题 | 你将学到什么 |
|-------|-------------------|
| 令牌优化 | 模型选择，系统提示精简，后台进程 |
| 内存持久化 | 自动跨会话保存/加载上下文的钩子 |
| 持续学习 | 从会话中自动提取模式为可重用技能 |
| 验证循环 | 检查点与持续评估，评分器类型，pass@k 指标 |
| 并行化 | Git 工作树，级联方法，何时扩展实例 |
| 子智能体编排 | 上下文问题，迭代检索模式 |

***

## 内容概览

```
everything-claude-code/
|-- agents/           # Specialized subagents for delegation
|   |-- planner.md           # Feature implementation planning
|   |-- architect.md         # System design decisions
|   |-- tdd-guide.md         # Test-driven development
|   |-- code-reviewer.md     # Quality and security review
|   |-- security-reviewer.md # Vulnerability analysis
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E testing
|   |-- refactor-cleaner.md  # Dead code cleanup
|   |-- doc-updater.md       # Documentation sync
|
|-- skills/           # Workflow definitions and domain knowledge
|   |-- coding-standards.md         # Language best practices
|   |-- backend-patterns.md         # API, database, caching patterns
|   |-- frontend-patterns.md        # React, Next.js patterns
|   |-- continuous-learning/        # Auto-extract patterns from sessions (Longform Guide)
|   |-- strategic-compact/          # Manual compaction suggestions (Longform Guide)
|   |-- tdd-workflow/               # TDD methodology
|   |-- security-review/            # Security checklist
|
|-- commands/         # Slash commands for quick execution
|   |-- tdd.md              # /tdd - Test-driven development
|   |-- plan.md             # /plan - Implementation planning
|   |-- e2e.md              # /e2e - E2E test generation
|   |-- code-review.md      # /code-review - Quality review
|   |-- build-fix.md        # /build-fix - Fix build errors
|   |-- refactor-clean.md   # /refactor-clean - Dead code removal
|   |-- learn.md            # /learn - Extract patterns mid-session (Longform Guide)
|
|-- rules/            # Always-follow guidelines
|   |-- security.md         # Mandatory security checks
|   |-- coding-style.md     # Immutability, file organization
|   |-- testing.md          # TDD, 80% coverage requirement
|   |-- git-workflow.md     # Commit format, PR process
|   |-- agents.md           # When to delegate to subagents
|   |-- performance.md      # Model selection, context management
|
|-- hooks/            # Trigger-based automations
|   |-- hooks.json                # All hooks config (PreToolUse, PostToolUse, Stop, etc.)
|   |-- memory-persistence/       # Session lifecycle hooks (Longform Guide)
|   |   |-- pre-compact.sh        # Save state before compaction
|   |   |-- session-start.sh      # Load previous context
|   |   |-- session-end.sh        # Persist learnings on end
|   |-- strategic-compact/        # Compaction suggestions (Longform Guide)
|
|-- contexts/         # Dynamic system prompt injection contexts (Longform Guide)
|   |-- dev.md              # Development mode context
|   |-- review.md           # Code review mode context
|   |-- research.md         # Research/exploration mode context
|
|-- examples/         # Example configurations and sessions
|   |-- CLAUDE.md           # Example project-level config
|   |-- user-CLAUDE.md      # Example user-level config
|   |-- sessions/           # Example session log files (Longform Guide)
|
|-- mcp-configs/      # MCP server configurations
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway, etc.
|
|-- plugins/          # Plugin ecosystem documentation
    |-- README.md           # Plugins, marketplaces, skills guide
```

***

## 快速开始

### 1. 复制你需要的部分

```bash
# Clone the repo
git clone https://github.com/affaan-m/everything-claude-code.git

# Copy agents to your Claude config
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Copy rules
cp everything-claude-code/rules/*.md ~/.claude/rules/

# Copy commands
cp everything-claude-code/commands/*.md ~/.claude/commands/

# Copy skills
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. 将钩子添加到 settings.json

将 `hooks/hooks.json` 中的钩子复制到你的 `~/.claude/settings.json`。

### 3. 配置 MCPs

将 `mcp-configs/mcp-servers.json` 中所需的 MCP 服务器复制到你的 `~/.claude.json`。

**重要：** 将 `YOUR_*_HERE` 占位符替换为你实际的 API 密钥。

### 4. 阅读指南

说真的，阅读指南。有了上下文，这些配置会好理解 10 倍。

1. **[简明指南](https://x.com/affaanmustafa/status/2012378465664745795)** - 设置和基础
2. **[详细指南](https://x.com/affaanmustafa/status/2014040193557471352)** - 高级技术（令牌优化，内存持久化，评估，并行化）

***

## 关键概念

### 智能体

子智能体处理具有有限范围的委托任务。示例：

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer...
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

规则是始终遵循的指导原则。保持其模块化：

```
~/.claude/rules/
  security.md      # No hardcoded secrets
  coding-style.md  # Immutability, file limits
  testing.md       # TDD, coverage requirements
```

***

## 贡献

**欢迎并鼓励贡献。**

此仓库旨在成为社区资源。如果你有：

* 有用的智能体或技能
* 巧妙的钩子
* 更好的 MCP 配置
* 改进的规则

请贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

### 贡献想法

* 语言特定技能（Python，Go，Rust 模式）
* 框架特定配置（Django，Rails，Laravel）
* DevOps 智能体（Kubernetes，Terraform，AWS）
* 测试策略（不同框架）
* 领域特定知识（ML，数据工程，移动端）

***

## 背景

我从实验性推出以来就一直在使用 Claude Code。在 2025 年 9 月，与 [@DRodriguezFX](https://x.com/DRodriguezFX) 一起使用 Claude Code 构建 [zenith.chat](https://zenith.chat)，赢得了 Anthropic x Forum Ventures 黑客马拉松。

这些配置已在多个生产应用程序中经过实战测试。

***

## 重要说明

### 上下文窗口管理

**关键：** 不要一次性启用所有 MCP。启用过多工具后，你的 200k 上下文窗口可能会缩小到 70k。

经验法则：

* 配置 20-30 个 MCP
* 每个项目保持启用少于 10 个
* 活动工具少于 80 个

在项目配置中使用 `disabledMcpServers` 来禁用未使用的工具。

### 定制化

这些配置适用于我的工作流。你应该：

1. 从引起共鸣的部分开始
2. 根据你的技术栈进行修改
3. 移除你不使用的部分
4. 添加你自己的模式

***

## 链接

* **简明指南（从此开始）：** [Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)
* **详细指南（高级）：** [Everything Claude Code 详细指南](https://x.com/affaanmustafa/status/2014040193557471352)
* **关注：** [@affaanmustafa](https://x.com/affaanmustafa)
* **zenith.chat：** [zenith.chat](https://zenith.chat)

***

## 许可证

MIT - 自由使用，根据需要修改，如果可以请回馈贡献。

***

**如果此仓库对你有帮助，请点星。阅读两份指南。构建伟大的东西。**
