---
description: 在运行验证检查后创建、验证或列出工作流检查点。
---

# 检查点命令

在您的工作流程中创建或验证一个检查点。

## 用法

`/checkpoint [create|verify|list] [name]`

## 创建检查点

创建检查点时：

1. 运行 `/verify quick` 以确保当前状态是干净的
2. 使用检查点名称创建一个 git stash 或提交
3. 将检查点记录到 `.claude/checkpoints.log`：

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 报告检查点已创建

## 验证检查点

针对某个检查点进行验证时：

1. 从日志中读取检查点

2. 将当前状态与检查点进行比较：
   * 自检查点以来新增的文件
   * 自检查点以来修改的文件
   * 当前与当时的测试通过率
   * 当前与当时的覆盖率

3. 报告：

```
CHECKPOINT COMPARISON: $NAME
============================
文件更改数: X
测试: +Y 通过 / -Z 失败
覆盖率: +X% / -Y%
构建: [通过/失败]
```

## 列出检查点

显示所有检查点，包含：

* 名称
* 时间戳
* Git SHA
* 状态（当前、落后、超前）

## 工作流程

典型的检查点流程：

```
[开始] --> /checkpoint create "feature-start"
   |
[实现] --> /checkpoint create "core-done"
   |
[测试] --> /checkpoint verify "core-done"
   |
[重构] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## 参数

$ARGUMENTS:

* `create <name>` - 创建指定名称的检查点
* `verify <name>` - 针对指定名称的检查点进行验证
* `list` - 显示所有检查点
* `clear` - 移除旧的检查点（保留最近5个）
