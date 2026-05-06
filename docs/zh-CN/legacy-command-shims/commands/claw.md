---
description: 用于 nanoclaw-repl 技能的旧版斜杠入口填充程序。建议直接使用该技能。
---

# Claw 命令（旧版兼容层）

仅当您仍凭肌肉记忆使用 `/claw` 时使用此命令。维护版本位于 `skills/nanoclaw-repl/SKILL.md`。

## 规范接口

* 优先直接使用 `nanoclaw-repl` 技能。
* 在逐步淘汰命令优先用法的过渡期间，仅保留此文件作为兼容性入口点。

## 参数

`$ARGUMENTS`

## 委托

应用 `nanoclaw-repl` 技能，并将响应聚焦于操作或扩展 `scripts/claw.js`。

* 如果用户希望运行它，请使用 `node scripts/claw.js` 或 `npm run claw`。
* 如果用户希望扩展它，请保留零依赖和基于 Markdown 的会话模型。
* 如果请求实际上涉及长时间运行的工作流编排而非 NanoClaw 本身，请重定向至 `dmux-workflows` 或 `autonomous-agent-harness`。
