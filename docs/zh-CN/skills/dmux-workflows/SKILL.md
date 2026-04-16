---
name: dmux-workflows
description: 使用dmux（AI代理的tmux窗格管理器）进行多代理编排。跨Claude Code、Codex、OpenCode和其他框架的并行代理工作流模式。适用于并行运行多个代理会话或协调多代理开发工作流时。
origin: ECC
---

# dmux 工作流

使用 dmux（一个用于智能体工具的 tmux 窗格管理器）来编排并行的 AI 智能体会话。

## 何时激活

* 并行运行多个智能体会话时
* 跨 Claude Code、Codex 和其他工具协调工作时
* 受益于分治并行策略的复杂任务
* 用户提及“并行运行”、“拆分此工作”、“使用 dmux”或“多智能体”时

## 什么是 dmux

dmux 是一个基于 tmux 的编排工具，用于管理 AI 智能体窗格：

* 按 `n` 创建一个带有提示的新窗格
* 按 `m` 将窗格输出合并回主会话
* 支持：Claude Code、Codex、OpenCode、Cline、Gemini、Qwen

**安装：** 在审查软件包后，从其代码仓库安装 dmux。参见 [github.com/standardagents/dmux](https://github.com/standardagents/dmux)

## 快速开始

```bash
# Start dmux session
dmux

# Create agent panes (press 'n' in dmux, then type prompt)
# Pane 1: "Implement the auth middleware in src/auth/"
# Pane 2: "Write tests for the user service"
# Pane 3: "Update API documentation"

# Each pane runs its own agent session
# Press 'm' to merge results back
```

## 工作流模式

### 模式 1：研究 + 实现

将研究和实现拆分为并行轨道：

```
Pane 1 (研究): "研究 Node.js 中速率限制的最佳实践。
  检查现有库，比较不同方法，并将研究结果写入
  /tmp/rate-limit-research.md"

Pane 2 (实现): "为我们的 Express API 实现速率限制中间件。
  从基本的令牌桶开始，待研究完成后我们将进行优化。"

# 在 Pane 1 完成后，将研究结果合并到 Pane 2 的上下文中
```

### 模式 2：多文件功能

跨独立文件并行化工作：

```
Pane 1: "创建计费功能的数据库架构和迁移"
Pane 2: "在 src/api/billing/ 中构建计费 API 端点"
Pane 3: "创建计费仪表板 UI 组件"

# 合并所有内容，然后在主面板中进行集成
```

### 模式 3：测试 + 修复循环

在一个窗格中运行测试，在另一个窗格中修复：

```
窗格 1 (观察者): "在监视模式下运行测试套件。当测试失败时，
  总结失败情况。"

窗格 2 (修复者): "根据窗格 1 的错误输出修复失败的测试"
```

### 模式 4：跨工具

对不同任务使用不同的 AI 工具：

```
窗格 1 (Claude Code): "审查认证模块的安全性"
窗格 2 (Codex): "重构工具函数以提升性能"
窗格 3 (Claude Code): "为结账流程编写端到端测试"
```

### 模式 5：代码审查流水线

并行审查视角：

```
Pane 1: "审查 src/api/ 中的安全漏洞"
Pane 2: "审查 src/api/ 中的性能问题"
Pane 3: "审查 src/api/ 中的测试覆盖缺口"

# 将所有审查合并为一份报告
```

## 最佳实践

1. **仅限独立任务。** 不要并行化那些彼此输出有依赖的任务。
2. **清晰的边界。** 每个窗格应处理不同的文件或关注点。
3. **策略性合并。** 在合并前审查窗格输出，以避免冲突。
4. **使用 git worktree。** 对于容易产生文件冲突的工作，为每个窗格使用独立的工作树。
5. **资源意识。** 每个窗格都会消耗 API 令牌 — 将总窗格数控制在 5-6 个以下。

## Git Worktree 集成

对于涉及重叠文件的任务：

```bash
# Create worktrees for isolation
git worktree add -b feat/auth ../feature-auth HEAD
git worktree add -b feat/billing ../feature-billing HEAD

# Run agents in separate worktrees
# Pane 1: cd ../feature-auth && claude
# Pane 2: cd ../feature-billing && claude

# Merge branches when done
git merge feat/auth
git merge feat/billing
```

## 互补工具

| 工具 | 功能 | 使用时机 |
|------|-------------|-------------|
| **dmux** | 用于智能体的 tmux 窗格管理 | 并行智能体会话 |
| **Superset** | 支持 10+ 并行智能体的终端 IDE | 大规模编排 |
| **Claude Code Task 工具** | 进程内子智能体生成 | 会话内的程序化并行 |
| **Codex 多智能体** | 内置智能体角色 | Codex 特定的并行工作 |

## ECC 助手

ECC 现在包含一个助手，用于在独立的 git worktree 中进行外部 tmux 窗格编排：

```bash
node scripts/orchestrate-worktrees.js plan.json --execute
```

示例 `plan.json`：

```json
{
  "sessionName": "skill-audit",
  "baseRef": "HEAD",
  "launcherCommand": "codex exec --cwd {worktree_path} --task-file {task_file}",
  "workers": [
    { "name": "docs-a", "task": "Fix skills 1-4 and write handoff notes." },
    { "name": "docs-b", "task": "Fix skills 5-8 and write handoff notes." }
  ]
}
```

该助手：

* 为每个工作线程创建一个基于分支的 git worktree
* 可选择性地将主检出中的选定 `seedPaths` 覆盖到每个工作线程的 worktree 中
* 在 `.orchestration/<session>/` 下写入每个工作线程的 `task.md`、`handoff.md` 和 `status.md` 文件
* 启动一个 tmux 会话，每个工作线程对应一个窗格
* 在每个窗格中启动相应的工作线程命令
* 为主协调器保留主窗格空闲

当工作线程需要访问尚未纳入 `HEAD` 的脏文件或未跟踪的本地文件（例如本地编排脚本、草稿计划或文档）时，使用 `seedPaths`：

```json
{
  "sessionName": "workflow-e2e",
  "seedPaths": [
    "scripts/orchestrate-worktrees.js",
    "scripts/lib/tmux-worktree-orchestrator.js",
    ".claude/plan/workflow-e2e-test.json"
  ],
  "launcherCommand": "bash {repo_root}/scripts/orchestrate-codex-worker.sh {task_file} {handoff_file} {status_file}",
  "workers": [
    { "name": "seed-check", "task": "Verify seeded files are present before starting work." }
  ]
}
```

## 故障排除

* **窗格无响应：** 直接切换到该窗格或使用 `tmux capture-pane -pt <session>:0.<pane-index>` 检查它。
* **合并冲突：** 使用 git worktree 来隔离每个窗格的文件更改。
* **令牌使用量高：** 减少并行窗格的数量。每个窗格都是一个完整的智能体会话。
* **找不到 tmux：** 使用 `brew install tmux`（macOS）或 `apt install tmux`（Linux）安装。
