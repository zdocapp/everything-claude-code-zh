---
description: dmux-workflows 和 autonomous-agent-harness 的旧版斜杠入口垫片。建议直接使用技能。
---

# 编排命令（旧版适配层）

仅在你仍调用 `/orchestrate` 时使用此命令。维护中的编排指南位于 `skills/dmux-workflows/SKILL.md` 和 `skills/autonomous-agent-harness/SKILL.md`。

## 规范使用场景

* 对于并行窗格、工作树和多智能体拆分，优先使用 `dmux-workflows`。
* 对于长时间运行的循环、治理、调度和控制平面式执行，优先使用 `autonomous-agent-harness`。
* 仅将此文件保留为兼容性入口点。

## 参数

`$ARGUMENTS`

## 委托

应用编排技能，而不是在此处维护第二个工作流规范。

* 对于拆分/并行执行，从 `dmux-workflows` 开始。
* 当用户真正需要持久循环、治理或操作层行为时，引入 `autonomous-agent-harness`。
* 保持交接结构清晰，但让技能定义维护中的排序规则。
  安全审查员：\[摘要]

### 已更改的文件

\[列出所有修改的文件]

### 测试结果

\[测试通过/失败摘要]

### 安全状态

\[安全发现]

### 建议

\[通过 / 需要改进 / 受阻]

````

## 并行执行

对于独立的检查，并行运行代理：

```markdown
### 并行阶段
同时运行：
- code-reviewer（质量）
- security-reviewer（安全）
- architect（设计）

### 合并结果
将输出合并为单一报告
````

对于具有独立 git 工作树的外部 tmux 窗格工作器，使用 `node scripts/orchestrate-worktrees.js plan.json --execute`。内置的编排模式保持在进程内；该助手用于长时间运行或跨工具链的会话。

当工作器需要查看主检出中的脏文件或未跟踪的本地文件时，将 `seedPaths` 添加到计划文件中。ECC 仅在 `git worktree add` 之后将那些选定的路径覆盖到每个工作器工作树中，这既保持了分支隔离，又仍能暴露正在进行的本地脚本、计划或文档。

```json
{
  "sessionName": "workflow-e2e",
  "seedPaths": [
    "scripts/orchestrate-worktrees.js",
    "scripts/lib/tmux-worktree-orchestrator.js",
    ".claude/plan/workflow-e2e-test.json"
  ],
  "workers": [
    { "name": "docs", "task": "Update orchestration docs." }
  ]
}
```

要为活动的 tmux/工作树会话导出控制平面快照，请运行：

```bash
node scripts/orchestration-status.js .claude/plan/workflow-visual-proof.json
```

该快照以 JSON 格式包含会话活动、tmux 窗格元数据、工作器状态、目标、种子覆盖和最近的交接摘要。

## 操作员指挥中心交接

当工作流跨越多个会话、工作树或 tmux 窗格时，在最终交接中附加一个控制平面块：

```markdown
控制平面
-------------
会话：
- 活动会话 ID 或别名
- 每个活动工作线程的分支 + 工作树路径
- 适用时的 tmux 窗格或分离会话名称

差异：
- git 状态摘要
- 已修改文件的 git diff --stat
- 合并/冲突风险说明

审批：
- 待处理的用户审批
- 等待确认的受阻步骤

遥测：
- 上次活动时间戳或空闲信号
- 预估的令牌或成本漂移
- 由钩子或审查器引发的策略事件
```

这使得规划者、实施者、审查员和循环工作器在操作员界面上保持清晰可读。

## 工作流参数

$ARGUMENTS:

* `feature <description>` - 完整功能工作流
* `bugfix <description>` - 错误修复工作流
* `refactor <description>` - 重构工作流
* `security <description>` - 安全审查工作流
* `custom <agents> <description>` - 自定义智能体序列

## 自定义工作流示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "重新设计缓存层"
```

## 提示

1. **从规划者开始**处理复杂功能
2. **合并前始终包含代码审查员**
3. **对于认证/支付/个人身份信息，使用安全审查员**
4. **保持交接简洁** - 专注于下一个智能体需要的内容
5. **如果需要，在智能体之间运行验证**
