---
name: continuous-learning-v2
description: 基于本能的学习系统，通过钩子观察会话，创建带有置信度评分的原子本能，并将其演变为技能/命令/代理。v2.1 增加了项目范围的本能，以防止跨项目污染。
origin: ECC
version: 2.1.0
---

# 持续学习 v2.1 - 本能

驱动架构

一个高级学习系统，通过原子化的“本能”（带有置信度评分的小型学习行为）将你的 Claude Code 会话转化为可复用的知识。

**v2.1** 新增了**项目级本能**——React 模式保留在 React 项目中，Python 约定保留在 Python 项目中，而通用模式（如“始终验证输入”）则全局共享。

## 何时激活

* 从 Claude Code 会话中设置自动学习
* 通过钩子配置基于本能的行为提取
* 调整学习行为的置信度阈值
* 审查、导出或导入本能库
* 将本能演化为完整的技能、命令或代理
* 管理项目级与全局本能
* 将本能从项目范围提升到全局范围

## v2.1 新特性

| 特性 | v2.0 | v2.1 |
|---------|------|------|
| 存储 | 全局 (~/.claude/homunculus/) | 项目级 (projects/<hash>/) |
| 范围 | 所有本能适用于任何地方 | 项目级 + 全局 |
| 检测 | 无 | git 远程 URL / 仓库路径 |
| 提升 | 不适用 | 在 2 个以上项目中出现时，项目 → 全局 |
| 命令 | 4 个 (status/evolve/export/import) | 6 个 (+promote/projects) |
| 跨项目 | 污染风险 | 默认隔离 |

## v2 新特性（对比 v1）

| 特性 | v1 | v2 |
|---------|----|----|
| 观察 | 停止钩子（会话结束） | PreToolUse/PostToolUse（100% 可靠） |
| 分析 | 主上下文 | 后台代理（Haiku） |
| 粒度 | 完整技能 | 原子化“本能” |
| 置信度 | 无 | 0.3-0.9 加权 |
| 演化 | 直接到技能 | 本能 -> 聚类 -> 技能/命令/代理 |
| 共享 | 无 | 导出/导入本能 |

## 本能模型

本能是一种小型学习行为：

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
scope: project
project_id: "a1b2c3d4e5f6"
project_name: "my-react-app"
---

# Prefer Functional Style

## Action
Use functional patterns over classes when appropriate.

## Evidence
- Observed 5 instances of functional pattern preference
- User corrected class-based approach to functional on 2025-01-15
```

**属性：**

* **原子化**——一个触发条件，一个动作
* **置信度加权**——0.3 = 试探性，0.9 = 几乎确定
* **领域标记**——代码风格、测试、git、调试、工作流等
* **有证据支持**——追踪创建它的观察记录
* **范围感知**——`project`（默认）或 `global`

## 工作原理

```
会话活动（在 git 仓库中）
      |
      | 钩子捕获提示 + 工具使用（100% 可靠）
      | + 检测项目上下文（git 远程 / 仓库路径）
      v
+---------------------------------------------+
|  projects/<project-hash>/observations.jsonl  |
|   （提示、工具调用、结果、项目）               |
+---------------------------------------------+
      |
      | 观察者代理读取（后台，Haiku）
      v
+---------------------------------------------+
|          模式检测                            |
|   * 用户修正 -> 本能                        |
|   * 错误解决 -> 本能                        |
|   * 重复工作流 -> 本能                      |
|   * 范围决策：项目还是全局？                 |
+---------------------------------------------+
      |
      | 创建/更新
      v
+---------------------------------------------+
|  projects/<project-hash>/instincts/personal/ |
|   * prefer-functional.yaml (0.7) [项目]      |
|   * use-react-hooks.yaml (0.9) [项目]        |
+---------------------------------------------+
|  instincts/personal/  （全局）               |
|   * always-validate-input.yaml (0.85) [全局] |
|   * grep-before-edit.yaml (0.6) [全局]       |
+---------------------------------------------+
      |
      | /evolve 聚类 + /promote
      v
+---------------------------------------------+
|  projects/<hash>/evolved/ （项目范围）       |
|  evolved/ （全局）                           |
|   * commands/new-feature.md                  |
|   * skills/testing-workflow.md               |
|   * agents/refactor-specialist.md            |
+---------------------------------------------+
```

## 项目检测

系统自动检测当前项目：

1. **`CLAUDE_PROJECT_DIR` 环境变量**（最高优先级）
2. **`git remote get-url origin`**——哈希后生成可移植的项目 ID（同一仓库在不同机器上获得相同 ID）
3. **`git rev-parse --show-toplevel`**——使用仓库路径作为后备方案（机器特定）
4. **全局后备**——如果未检测到项目，本能归入全局范围

每个项目获得一个 12 字符的哈希 ID（例如 `a1b2c3d4e5f6`）。注册文件 `~/.claude/homunculus/projects.json` 将 ID 映射为人类可读的名称。

## 快速开始

### 1. 启用观察钩子

**如果作为插件安装**（推荐）：

无需额外的 `settings.json` 钩子块。Claude Code v2.1+ 会自动加载插件 `hooks/hooks.json`，且 `observe.sh` 已在该处注册。

如果你之前将 `observe.sh` 复制到了 `~/.claude/settings.json`，请移除重复的 `PreToolUse` / `PostToolUse` 块。重复的插件钩子会导致双重执行以及 `${CLAUDE_PLUGIN_ROOT}` 解析错误，因为该变量仅在插件管理的 `hooks/hooks.json` 条目中可用。

**如果手动安装**到 `~/.claude/skills`，请将其添加到你的 `~/.claude/settings.json` 中：

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }]
  }
}
```

### 2. 初始化目录结构

系统在首次使用时自动创建目录，但你也可以手动创建：

```bash
# Global directories
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands},projects}

# Project directories are auto-created when the hook first runs in a git repo
```

### 3. 使用本能命令

```bash
/instinct-status     # Show learned instincts (project + global)
/evolve              # Cluster related instincts into skills/commands
/instinct-export     # Export instincts to file
/instinct-import     # Import instincts from others
/promote             # Promote project instincts to global scope
/projects            # List all known projects and their instinct counts
```

## 命令

| 命令 | 描述 |
|---------|-------------|
| `/instinct-status` | 显示所有本能（项目级 + 全局）及置信度 |
| `/evolve` | 将相关本能聚类为技能/命令，建议提升 |
| `/instinct-export` | 导出本能（可按范围/领域过滤） |
| `/instinct-import <file>` | 导入本能并控制范围 |
| `/promote [id]` | 将项目本能提升到全局范围 |
| `/projects` | 列出所有已知项目及其本能数量 |

## 配置

编辑 `config.json` 以控制后台观察者：

```json
{
  "version": "2.1",
  "observer": {
    "enabled": false,
    "run_interval_minutes": 5,
    "min_observations_to_analyze": 20
  }
}
```

| 键 | 默认值 | 描述 |
|-----|---------|-------------|
| `observer.enabled` | `false` | 启用后台观察者代理 |
| `observer.run_interval_minutes` | `5` | 观察者分析观察结果的频率 |
| `observer.min_observations_to_analyze` | `20` | 分析运行前的最小观察次数 |

其他行为（观察捕获、本能阈值、项目范围、提升标准）通过 `instinct-cli.py` 和 `observe.sh` 中的代码默认值配置。

## 文件结构

```
~/.claude/homunculus/
+-- identity.json           # 你的个人资料、技术水平
+-- projects.json           # 注册表：项目哈希 -> 名称/路径/远程仓库
+-- observations.jsonl      # 全局观察记录（备用）
+-- instincts/
|   +-- personal/           # 全局自动学习的本能
|   +-- inherited/          # 全局导入的本能
+-- evolved/
|   +-- agents/             # 全局生成的智能体
|   +-- skills/             # 全局生成的技能
|   +-- commands/           # 全局生成的命令
+-- projects/
    +-- a1b2c3d4e5f6/       # 项目哈希（来自 Git 远程仓库 URL）
    |   +-- project.json    # 每个项目的元数据镜像（ID/名称/根目录/远程仓库）
    |   +-- observations.jsonl
    |   +-- observations.archive/
    |   +-- instincts/
    |   |   +-- personal/   # 项目特定的自动学习
    |   |   +-- inherited/  # 项目特定的导入
    |   +-- evolved/
    |       +-- skills/
    |       +-- commands/
    |       +-- agents/
    +-- f6e5d4c3b2a1/       # 另一个项目
        +-- ...
```

## 范围决策指南

| 模式类型 | 范围 | 示例 |
|-------------|-------|---------|
| 语言/框架约定 | **项目** | “使用 React 钩子”、“遵循 Django REST 模式” |
| 文件结构偏好 | **项目** | “测试放在 `__tests__`/”、“组件放在 src/components/” |
| 代码风格 | **项目** | “使用函数式风格”、“偏好数据类” |
| 错误处理策略 | **项目** | “使用 Result 类型处理错误” |
| 安全实践 | **全局** | “验证用户输入”、“清理 SQL” |
| 通用最佳实践 | **全局** | “先写测试”、“始终处理错误” |
| 工具工作流偏好 | **全局** | “编辑前先搜索”、“写入前先读取” |
| Git 实践 | **全局** | “约定式提交”、“小而专注的提交” |

## 本能提升（项目 -> 全局）

当同一本能在多个项目中出现且置信度较高时，它就有资格提升到全局范围。

**自动提升标准：**

* 同一本能 ID 出现在 2 个以上项目中
* 平均置信度 >= 0.8

**如何提升：**

```bash
# Promote a specific instinct
python3 instinct-cli.py promote prefer-explicit-errors

# Auto-promote all qualifying instincts
python3 instinct-cli.py promote

# Preview without changes
python3 instinct-cli.py promote --dry-run
```

`/evolve` 命令也会建议提升候选。

## 置信度评分

置信度随时间演变：

| 分数 | 含义 | 行为 |
|-------|---------|----------|
| 0.3 | 试探性 | 建议但不强制 |
| 0.5 | 中等 | 相关时应用 |
| 0.7 | 强 | 自动批准应用 |
| 0.9 | 几乎确定 | 核心行为 |

**置信度增加**当：\*\*

* 模式被重复观察到
* 用户未纠正建议的行为
* 来自其他来源的类似本能一致

**置信度降低**当：\*\*

* 用户明确纠正行为
* 长时间未观察到模式
* 出现矛盾证据

## 为什么用钩子而非技能进行观察？

> “v1 依赖技能进行观察。技能是概率性的——根据 Claude 的判断，它们大约在 50-80% 的情况下触发。”

钩子**100%** 触发，确定性执行。这意味着：

* 每次工具调用都被观察
* 不会遗漏任何模式
* 学习是全面的

## 向后兼容性

v2.1 完全兼容 v2.0 和 v1：

* `~/.claude/homunculus/instincts/` 中现有的全局本能仍作为全局本能工作
* v1 中现有的 `~/.claude/skills/learned/` 技能仍然有效
* 停止钩子仍然运行（但现在也会输入到 v2）
* 逐步迁移：两者并行运行

## 隐私

* 观察结果**本地**存储在你的机器上
* 项目级本能按项目隔离
* 只有**本能**（模式）可以导出——而非原始观察结果
* 不共享任何实际代码或对话内容
* 你控制导出和提升的内容

## 相关

* [ECC-Tools GitHub 应用](https://github.com/apps/ecc-tools) - 从仓库历史生成本能
* Homunculus - 启发 v2 本能驱动架构的社区项目（原子化观察、置信度评分、本能演化管道）
* [长篇指南](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习部分

***

*基于本能的学习：一次一个项目，教会 Claude 你的模式。*
