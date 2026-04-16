---
name: agent-eval
description: 编码代理（Claude Code、Aider、Codex等）在自定义任务上的头对头比较，包含通过率、成本、时间和一致性指标
origin: ECC
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Agent Eval 技能

一个轻量级 CLI 工具，用于在可复现的任务上对编码代理进行头对头比较。每次“哪个编码代理最好？”的比较都基于感觉——这个工具将其系统化。

## 何时激活

* 在你自己的代码库上比较编码代理（Claude Code、Aider、Codex 等）
* 在采用新工具或模型之前测量代理性能
* 当代理更新其模型或工具时运行回归检查
* 为团队做出有数据支持的代理选择决策

## 安装

> **注意：** 在审查源代码后从其仓库安装 agent-eval。

## 核心概念

### YAML 任务定义

以声明方式定义任务。每个任务指定要做什么、要操作哪些文件以及如何判断成功：

```yaml
name: add-retry-logic
description: Add exponential backoff retry to the HTTP client
repo: ./my-project
files:
  - src/http_client.py
prompt: |
  Add retry logic with exponential backoff to all HTTP requests.
  Max 3 retries. Initial delay 1s, max delay 30s.
judge:
  - type: pytest
    command: pytest tests/test_http_client.py -v
  - type: grep
    pattern: "exponential_backoff|retry"
    files: src/http_client.py
commit: "abc1234"  # pin to specific commit for reproducibility
```

### Git Worktree 隔离

每个代理运行都获得自己的 git worktree——无需 Docker。这提供了可复现性隔离，因此代理之间不会相互干扰或破坏基础仓库。

### 收集的指标

| 指标 | 测量内容 |
|--------|-----------------|
| 通过率 | 代理生成的代码是否通过了评判？ |
| 成本 | 每个任务的 API 花费（如果可用） |
| 时间 | 完成所需的挂钟秒数 |
| 一致性 | 跨重复运行的通过率（例如，3/3 = 100%） |

## 工作流程

### 1. 定义任务

创建一个 `tasks/` 目录，其中包含 YAML 文件，每个任务一个文件：

```bash
mkdir tasks
# Write task definitions (see template above)
```

### 2. 运行代理

针对你的任务执行代理：

```bash
agent-eval run --task tasks/add-retry-logic.yaml --agent claude-code --agent aider --runs 3
```

每次运行：

1. 从指定的提交创建一个新的 git worktree
2. 将提示交给代理
3. 运行评判标准
4. 记录通过/失败、成本和时间

### 3. 比较结果

生成比较报告：

```bash
agent-eval report --format table
```

```
任务：添加重试逻辑（每次运行3次）
┌──────────────┬───────────┬────────┬────────┬─────────────┐
│ Agent        │ Pass Rate │ Cost   │ Time   │ Consistency │
├──────────────┼───────────┼────────┼────────┼─────────────┤
│ claude-code  │ 3/3       │ $0.12  │ 45s    │ 100%        │
│ aider        │ 2/3       │ $0.08  │ 38s    │  67%        │
└──────────────┴───────────┴────────┴────────┴─────────────┘
```

## 评判类型

### 基于代码（确定性）

```yaml
judge:
  - type: pytest
    command: pytest tests/ -v
  - type: command
    command: npm run build
```

### 基于模式

```yaml
judge:
  - type: grep
    pattern: "class.*Retry"
    files: src/**/*.py
```

### 基于模型（LLM 作为评判者）

```yaml
judge:
  - type: llm
    prompt: |
      Does this implementation correctly handle exponential backoff?
      Check for: max retries, increasing delays, jitter.
```

## 最佳实践

* **从 3-5 个代表你真实工作负载的任务开始**，而不是玩具示例
* **每个代理至少运行 3 次试验**以捕捉方差——代理是非确定性的
* **在你的任务 YAML 中固定提交**，以便结果在几天/几周内可复现
* **每个任务至少包含一个确定性评判者**（测试、构建）——LLM 评判者会增加噪音
* **在跟踪通过率的同时跟踪成本**——一个通过率 95% 但成本高出 10 倍的代理可能不是正确的选择
* **对你的任务定义进行版本控制**——它们是测试夹具，应将其视为代码

## 链接

* 仓库：[github.com/joaquinhuigomez/agent-eval](https://github.com/joaquinhuigomez/agent-eval)
