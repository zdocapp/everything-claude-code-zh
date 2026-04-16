# Git 工作流程

## 提交信息格式

```
<类型>: <描述>

<可选正文>
```

类型：feat, fix, refactor, docs, test, chore, perf, ci

注意：已通过 ~/.claude/settings.json 全局禁用归属。

## 拉取请求工作流程

创建 PR 时：

1. 分析完整的提交历史（不仅仅是最近的提交）
2. 使用 `git diff [base-branch]...HEAD` 查看所有更改
3. 起草全面的 PR 摘要
4. 包含带有待办事项的测试计划
5. 如果是新分支，使用 `-u` 标志推送

> 关于 git 操作之前的完整开发流程（规划、测试驱动开发、代码审查），
> 请参阅 [development-workflow.md](development-workflow.md)。
