---
description: "用于审核Claude技能和命令质量时使用。支持快速扫描（仅更改的技能）和全面盘点模式，采用顺序子代理批量评估。"
origin: ECC
---

# skill-stocktake

斜杠命令 (`/skill-stocktake`)，使用质量检查清单 + AI 整体判断来审计所有 Claude 技能和命令。支持两种模式：快速扫描（针对最近更改的技能）和全面盘点（进行完整审查）。

## 范围

该命令针对以下**相对于调用命令所在目录**的路径：

| 路径 | 描述 |
|------|-------------|
| `~/.claude/skills/` | 全局技能（所有项目） |
| `{cwd}/.claude/skills/` | 项目级技能（如果目录存在） |

**在阶段 1 开始时，该命令会明确列出找到并扫描了哪些路径。**

### 针对特定项目

要包含项目级技能，请从该项目根目录运行：

```bash
cd ~/path/to/my-project
/skill-stocktake
```

如果项目没有 `.claude/skills/` 目录，则仅评估全局技能和命令。

## 模式

| 模式 | 触发条件 | 持续时间 |
|------|---------|---------|
| 快速扫描 | `results.json` 存在（默认） | 5–10 分钟 |
| 全面盘点 | `results.json` 不存在，或 `/skill-stocktake full` | 20–30 分钟 |

**结果缓存：** `~/.claude/skills/skill-stocktake/results.json`

## 快速扫描流程

仅重新评估自上次运行以来发生更改的技能（5–10 分钟）。

1. 读取 `~/.claude/skills/skill-stocktake/results.json`
2. 运行：`bash ~/.claude/skills/skill-stocktake/scripts/quick-diff.sh \   ~/.claude/skills/skill-stocktake/results.json`
   （项目目录从 `$PWD/.claude/skills` 自动检测；仅在需要时显式传递）
3. 如果输出是 `[]`：报告“自上次运行以来无更改。”并停止
4. 使用相同的阶段 2 标准仅重新评估那些已更改的文件
5. 沿用先前结果中未更改的技能
6. 仅输出差异
7. 运行：`bash ~/.claude/skills/skill-stocktake/scripts/save-results.sh \   ~/.claude/skills/skill-stocktake/results.json <<< "$EVAL_RESULTS"`

## 全面盘点流程

### 阶段 1 — 清单

运行：`bash ~/.claude/skills/skill-stocktake/scripts/scan.sh`

脚本枚举技能文件，提取 frontmatter，并收集 UTC 修改时间。
项目目录从 `$PWD/.claude/skills` 自动检测；仅在需要时显式传递。
呈现脚本输出中的扫描摘要和清单表：

```
扫描中:
  ✓ ~/.claude/skills/         (17 个文件)
  ✗ {cwd}/.claude/skills/    (未找到 — 仅全局技能)
```

| 技能 | 7 天使用 | 30 天使用 | 描述 |
|-------|--------|---------|-------------|

### 阶段 2 — 质量评估

启动一个 Agent 工具子代理（**通用代理**），并提供完整的清单和检查清单：

```text
Agent(
  subagent_type="general-purpose",
  prompt="
根据以下清单评估技能清单。

[INVENTORY]

[CHECKLIST]

为每项技能返回JSON：
{ \"verdict\": \"保留\"|\"改进\"|\"更新\"|\"弃用\"|\"合并至[X]\", \"reason\": \"...\" }
"
)
```

子代理读取每个技能，应用检查清单，并返回每个技能的 JSON：

`{ "verdict": "Keep"|"Improve"|"Update"|"Retire"|"Merge into [X]", "reason": "..." }`

**分块指导：** 每次子代理调用处理约 20 个技能，以保持上下文可控。每个块处理后，将中间结果保存到 `results.json` (`status: "in_progress"`)。

所有技能评估完成后：设置 `status: "completed"`，进入阶段 3。

**恢复检测：** 如果在启动时找到 `status: "in_progress"`，则从第一个未评估的技能恢复。

每个技能根据此检查清单进行评估：

```
- [ ] 已检查与其他技能的内容重叠
- [ ] 已检查与 MEMORY.md / CLAUDE.md 的重叠
- [ ] 已验证技术参考的新鲜度（如果存在工具名称 / CLI 标志 / API，请使用 WebSearch）
- [ ] 已考虑使用频率
```

判定标准：

| 判定 | 含义 |
|---------|---------|
| 保留 | 有用且最新 |
| 改进 | 值得保留，但需要具体改进 |
| 更新 | 引用的技术已过时（通过 WebSearch 验证） |
| 停用 | 质量低、陈旧或成本不对称 |
| 合并到 \[X] | 与另一技能有大量重叠；命名合并目标 |

评估是**整体 AI 判断**——而非数字评分标准。指导维度：

* **可操作性**：代码示例、命令或步骤，可让你立即行动
* **范围契合度**：名称、触发器和内容一致；不过于宽泛或狭窄
* **独特性**：价值无法被 MEMORY.md / CLAUDE.md / 另一技能替代
* **时效性**：技术引用在当前环境中有效

**理由质量要求** — `reason` 字段必须是自包含且能支持决策的：

* 不要只写“未更改”——始终重述核心证据
* 对于**停用**：说明 (1) 发现了什么具体缺陷，(2) 什么替代方案覆盖了相同需求
  * 差：`"Superseded"`
  * 好：`"disable-model-invocation: true already set; superseded by continuous-learning-v2 which covers all the same patterns plus confidence scoring. No unique content remains."`
* 对于**合并**：命名目标并描述要集成什么内容
  * 差：`"Overlaps with X"`
  * 好：`"42-line thin content; Step 4 of chatlog-to-article already covers the same workflow. Integrate the 'article angle' tip as a note in that skill."`
* 对于**改进**：描述所需的具体更改（哪个部分，什么操作，如果相关则说明目标大小）
  * 差：`"Too long"`
  * 好：`"276 lines; Section 'Framework Comparison' (L80–140) duplicates ai-era-architecture-principles; delete it to reach ~150 lines."`
* 对于**保留**（快速扫描中仅修改时间更改）：重述原始判定理由，不要写“未更改”
  * 差：`"Unchanged"`
  * 好：`"mtime updated but content unchanged. Unique Python reference explicitly imported by rules/python/; no overlap found."`

### 阶段 3 — 摘要表

| 技能 | 7 天使用 | 判定 | 理由 |
|-------|--------|---------|--------|

### 阶段 4 — 整合

1. **停用 / 合并**：在用户确认前，按文件呈现详细理由：
   * 发现了什么具体问题（重叠、陈旧、引用损坏等）
   * 什么替代方案覆盖了相同功能（对于停用：哪个现有技能/规则；对于合并：目标文件以及要集成什么内容）
   * 移除的影响（任何依赖技能、MEMORY.md 引用或受影响的工作流）
2. **改进**：呈现具体的改进建议及理由：
   * 更改什么以及为什么（例如，“将 430 行缩减至 200 行，因为 X/Y 部分与 python-patterns 重复”）
   * 用户决定是否执行
3. **更新**：呈现已检查来源的更新内容
4. 检查 MEMORY.md 行数；如果 >100 行，则建议压缩

## 结果文件模式

`~/.claude/skills/skill-stocktake/results.json`：

**`evaluated_at`**：必须设置为评估完成时的实际 UTC 时间。
通过 Bash 获取：`date -u +%Y-%m-%dT%H:%M:%SZ`。切勿使用仅日期的近似值，如 `T00:00:00Z`。

```json
{
  "evaluated_at": "2026-02-21T10:00:00Z",
  "mode": "full",
  "batch_progress": {
    "total": 80,
    "evaluated": 80,
    "status": "completed"
  },
  "skills": {
    "skill-name": {
      "path": "~/.claude/skills/skill-name/SKILL.md",
      "verdict": "Keep",
      "reason": "Concrete, actionable, unique value for X workflow",
      "mtime": "2026-01-15T08:30:00Z"
    }
  }
}
```

## 备注

* 评估是盲目的：无论来源如何（ECC、自创、自动提取），所有技能都应用相同的检查清单
* 归档/删除操作始终需要用户明确确认
* 不按技能来源进行判定分支
