---
name: prompt-optimizer
description: 分析原始提示，识别意图和差距，匹配ECC组件（技能/命令/代理/钩子），并输出一个可直接粘贴的优化提示。仅提供咨询角色——从不执行任务本身。触发条件：当用户说“优化prompt”、“改进我的prompt”、“怎么写prompt”、“帮我优化这个指令”或明确要求提升提示质量时。中文等效词也触发：“优化prompt”、“改进prompt”、“怎么写prompt”、“帮我优化这个指令”。不触发条件：当用户希望直接执行任务，或说“直接做”时。不触发条件：当用户说“优化代码”、“优化性能”、“optimize performance”、“optimize this code”时——这些是重构/性能任务，而非提示优化。origin: community
metadata:
  author: YannJY02
  version: "1.0.0"
---

# 提示词优化器

分析一个草稿提示词，对其进行评价，将其与 ECC 生态系统组件匹配，并输出一个完整的优化后提示词，供用户复制粘贴并运行。

## 使用时机

* 用户说“优化这个提示词”、“改进我的提示词”、“重写这个提示词”
* 用户说“帮我写一个更好的提示词用于...”
* 用户说“让 Claude Code 做...的最佳方式是什么”
* 用户说“优化prompt”、“改进prompt”、“怎么写prompt”、“帮我优化这个指令”
* 用户粘贴了一个草稿提示词并要求反馈或改进
* 用户说“我不知道怎么为这个写提示词”
* 用户说“我应该如何使用 ECC 来...”
* 用户明确调用 `/prompt-optimize`

### 不要使用的情况

* 用户希望直接执行任务（直接做）
* 用户说“优化代码”、“优化性能”、“optimize this code”、“optimize performance”——这些是代码重构任务，不是提示词优化
* 用户询问 ECC 配置（改用 `configure-ecc`）
* 用户想要技能清单（改用 `skill-stocktake`）
* 用户说“just do it”或“直接做”

## 工作原理

**仅提供建议——不执行用户的任务。**

不要编写代码、创建文件、运行命令或采取任何实施行动。你的**唯一**输出是分析加上优化后的提示词。

如果用户说“just do it”、“直接做”或“不要优化，直接执行”，不要在此技能内切换到实施模式。告诉用户此技能仅生成优化后的提示词，并指导他们如果想要执行，请提出正常的任务请求。

按顺序运行这个 6 阶段流程。使用下面的输出格式呈现结果。

### 分析流程

### 阶段 0：项目检测

在分析提示词之前，检测当前项目上下文：

1. 检查工作目录中是否存在 `CLAUDE.md`——读取它以了解项目约定
2. 从项目文件中检测技术栈：
   * `package.json` → Node.js / TypeScript / React / Next.js
   * `go.mod` → Go
   * `pyproject.toml` / `requirements.txt` → Python
   * `Cargo.toml` → Rust
   * `build.gradle` / `pom.xml` → Java / Kotlin / Spring Boot
   * `Package.swift` → Swift
   * `Gemfile` → Ruby
   * `composer.json` → PHP
   * `*.csproj` / `*.sln` → .NET
   * `Makefile` / `CMakeLists.txt` → C / C++
   * `cpanfile` / `Makefile.PL` → Perl
3. 记下检测到的技术栈，供阶段 3 和阶段 4 使用

如果未找到项目文件（例如，提示词是抽象的或用于新项目），则跳过检测并在阶段 4 中标记“技术栈未知”。

### 阶段 1：意图检测

将用户的任务分类到一个或多个类别中：

| 类别 | 信号词 | 示例 |
|----------|-------------|---------|
| 新功能 | build, create, add, implement, 创建, 实现, 添加 | “构建一个登录页面” |
| 错误修复 | fix, broken, not working, error, 修复, 报错 | “修复认证流程” |
| 重构 | refactor, clean up, restructure, 重构, 整理 | “重构 API 层” |
| 研究 | how to, what is, explore, investigate, 怎么, 如何 | “如何添加 SSO” |
| 测试 | test, coverage, verify, 测试, 覆盖率 | “为购物车添加测试” |
| 审查 | review, audit, check, 审查, 检查 | “审查我的 PR” |
| 文档 | document, update docs, 文档 | “更新 API 文档” |
| 基础设施 | deploy, CI, docker, database, 部署, 数据库 | “设置 CI/CD 流水线” |
| 设计 | design, architecture, plan, 设计, 架构 | “设计数据模型” |

### 阶段 2：范围评估

如果阶段 0 检测到项目，则使用代码库大小作为信号。否则，仅根据提示词描述进行估计，并将估计标记为不确定。

| 范围 | 启发式判断 | 编排 |
|-------|-----------|---------------|
| 微小 | 单个文件，< 50 行 | 直接执行 |
| 低 | 单个组件或模块 | 单个命令或技能 |
| 中 | 多个组件，同一领域 | 命令链 + /verify |
| 高 | 跨领域，5+ 个文件 | 先 /plan，然后分阶段执行 |
| 史诗级 | 多会话，多 PR，架构变更 | 使用蓝图技能进行多会话规划 |

### 阶段 3：ECC 组件匹配

将意图 + 范围 + 技术栈（来自阶段 0）映射到特定的 ECC 组件。

#### 按意图类型

| 意图 | 命令 | 技能 | 代理 |
|--------|----------|--------|--------|
| 新功能 | /plan, /tdd, /code-review, /verify | tdd-workflow, verification-loop | planner, tdd-guide, code-reviewer |
| 错误修复 | /tdd, /build-fix, /verify | tdd-workflow | tdd-guide, build-error-resolver |
| 重构 | /refactor-clean, /code-review, /verify | verification-loop | refactor-cleaner, code-reviewer |
| 研究 | /plan | search-first, iterative-retrieval | — |
| 测试 | /tdd, /e2e, /test-coverage | tdd-workflow, e2e-testing | tdd-guide, e2e-runner |
| 审查 | /code-review | security-review | code-reviewer, security-reviewer |
| 文档 | /update-docs, /update-codemaps | — | doc-updater |
| 基础设施 | /plan, /verify | docker-patterns, deployment-patterns, database-migrations | architect |
| 设计（中-高） | /plan | — | planner, architect |
| 设计（史诗级） | — | blueprint（作为技能调用） | planner, architect |

#### 按技术栈

| 技术栈 | 要添加的技能 | 代理 |
|------------|--------------|-------|
| Python / Django | django-patterns, django-tdd, django-security, django-verification, python-patterns, python-testing | python-reviewer |
| Go | golang-patterns, golang-testing | go-reviewer, go-build-resolver |
| Spring Boot / Java | springboot-patterns, springboot-tdd, springboot-security, springboot-verification, java-coding-standards, jpa-patterns | code-reviewer |
| Kotlin / Android | kotlin-coroutines-flows, compose-multiplatform-patterns, android-clean-architecture | kotlin-reviewer |
| TypeScript / React | frontend-patterns, backend-patterns, coding-standards | code-reviewer |
| Swift / iOS | swiftui-patterns, swift-concurrency-6-2, swift-actor-persistence, swift-protocol-di-testing | code-reviewer |
| PostgreSQL | postgres-patterns, database-migrations | database-reviewer |
| Perl | perl-patterns, perl-testing, perl-security | code-reviewer |
| C++ | cpp-coding-standards, cpp-testing | code-reviewer |
| 其他 / 未列出 | coding-standards（通用） | code-reviewer |

### 阶段 4：缺失上下文检测

扫描提示词以查找缺失的关键信息。检查每一项，并标记是阶段 0 自动检测到的还是用户必须提供的：

* \[ ] **技术栈** —— 阶段 0 检测到的，还是用户必须指定？
* \[ ] **目标范围** —— 提到了文件、目录或模块吗？
* \[ ] **验收标准** —— 如何知道任务已完成？
* \[ ] **错误处理** —— 是否考虑了边界情况和故障模式？
* \[ ] **安全要求** —— 认证、输入验证、密钥？
* \[ ] **测试期望** —— 单元测试、集成测试、E2E？
* \[ ] **性能约束** —— 负载、延迟、资源限制？
* \[ ] **UI/UX 要求** —— 设计规范、响应式、无障碍？（如果是前端）
* \[ ] **数据库变更** —— 模式、迁移、索引？（如果是数据层）
* \[ ] **现有模式** —— 要遵循的参考文件或约定？
* \[ ] **范围边界** —— 哪些事情**不要**做？

**如果缺少 3 个或更多关键项**，在生成优化后的提示词之前，向用户提出最多 3 个澄清问题。然后将答案纳入优化后的提示词中。

### 阶段 5：工作流和模型推荐

确定此提示词在开发生命周期中的位置：

```
研究 → 规划 → 实现（测试驱动开发） → 审查 → 验证 → 提交
```

对于中等级别及以上的任务，始终从 /plan 开始。对于史诗级任务，使用蓝图技能。

**模型推荐**（包含在输出中）：

| 范围 | 推荐模型 | 理由 |
|-------|------------------|-----------|
| 微小-低 | Sonnet 4.6 | 快速，对于简单任务成本效益高 |
| 中 | Sonnet 4.6 | 标准工作的最佳编码模型 |
| 高 | Sonnet 4.6（主要） + Opus 4.6（规划） | Opus 用于架构，Sonnet 用于实施 |
| 史诗级 | Opus 4.6（蓝图） + Sonnet 4.6（执行） | 深度推理用于多会话规划 |

**多提示词拆分**（针对高/史诗级范围）：

对于超出单个会话的任务，拆分为顺序提示词：

* 提示词 1：研究 + 规划（使用 search-first 技能，然后 /plan）
* 提示词 2-N：每个提示词实施一个阶段（每个阶段以 /verify 结束）
* 最终提示词：集成测试 + 跨所有阶段的 /code-review
* 使用 /save-session 和 /resume-session 在会话之间保存上下文

***

## 输出格式

按照这个确切的结构呈现你的分析。使用与用户输入相同的语言进行回应。

### 第 1 部分：提示词诊断

**优点：** 列出原始提示词做得好的地方。

**问题：**

| 问题 | 影响 | 建议修复 |
|-------|--------|---------------|
| （问题） | （后果） | （如何修复） |

**需要澄清：** 用户应回答的问题编号列表。如果阶段 0 自动检测到了答案，则说明该答案而不是提问。

### 第 2 部分：推荐的 ECC 组件

| 类型 | 组件 | 目的 |
|------|-----------|---------|
| 命令 | /plan | 编码前规划架构 |
| 技能 | tdd-workflow | TDD 方法指导 |
| 代理 | code-reviewer | 实施后审查 |
| 模型 | Sonnet 4.6 | 针对此范围的推荐 |

### 第 3 部分：优化后的提示词——完整版

在单个围栏代码块内呈现完整的优化后提示词。提示词必须是自包含的，可以复制粘贴。包括：

* 清晰的任务描述和上下文
* 技术栈（检测到的或指定的）
* 在正确工作流阶段调用的 /command
* 验收标准
* 验证步骤
* 范围边界（哪些事情**不要**做）

对于引用蓝图的项目，请写：“使用蓝图技能来...”（而不是 `/blueprint`，因为蓝图是技能，不是命令）。

### 第 4 部分：优化后的提示词——快速版

为有经验的 ECC 用户提供的紧凑版本。根据意图类型变化：

| 意图 | 快速模式 |
|--------|--------------|
| 新功能 | `/plan [feature]. /tdd to implement. /code-review. /verify.` |
| 错误修复 | `/tdd — write failing test for [bug]. Fix to green. /verify.` |
| 重构 | `/refactor-clean [scope]. /code-review. /verify.` |
| 研究 | `Use search-first skill for [topic]. /plan based on findings.` |
| 测试 | `/tdd [module]. /e2e for critical flows. /test-coverage.` |
| 审查 | `/code-review. Then use security-reviewer agent.` |
| 文档 | `/update-docs. /update-codemaps.` |
| 史诗级 | `Use blueprint skill for "[objective]". Execute phases with /verify gates.` |

### 第 5 部分：改进理由

| 改进 | 原因 |
|-------------|--------|
| （添加了什么） | （为什么重要） |

### 页脚

> 这不是你需要的吗？告诉我需要调整什么，或者如果你想要执行而不是提示词优化，请提出正常的任务请求。

***

## 示例

### 触发示例

* “为 ECC 优化这个提示词”
* “重写这个提示词，让 Claude Code 使用正确的命令”
* “帮我优化这个指令”
* “我应该如何为这个任务提示 ECC？”

### 示例 1：模糊的中文提示词（检测到项目）

**用户输入：**

```
帮我写一个用户登录页面
```

**阶段 0 检测到：** `package.json`，使用 Next.js 15, TypeScript, Tailwind CSS

**优化后的提示词（完整版）：**

```
使用项目现有技术栈（Next.js 15 + TypeScript + Tailwind CSS）实现用户登录页面。

技术要求：
- 沿用项目现有的组件结构和路由约定
- 表单验证使用项目中已有的验证方案（检查是否已用 Zod/Yup/其他）
- 认证方式：沿用项目现有认证方案（如无，默认 JWT）
- 包含：邮箱/密码登录表单、表单验证、错误提示、加载状态、响应式布局

工作流：
1. /plan 先规划组件结构和认证流程，参考现有页面的模式
2. /tdd 测试先行：编写登录表单的单元测试和认证流程的集成测试
3. 实现登录页面和认证逻辑
4. /code-review 审查实现
5. /verify 验证所有测试通过且页面正常渲染

安全要求：
- 密码不明文传输
- 防止暴力破解（rate limiting）
- XSS 防护
- CSRF token

验收标准：
- 所有测试通过，覆盖率 80%+
- 页面在移动端和桌面端正常渲染
- 登录成功跳转到 dashboard，失败显示错误信息

不要做：
- 不要实现注册页面
- 不要实现忘记密码功能
- 不要修改现有的路由结构
```

### 示例 2：中等英文提示词

**用户输入：**

```
添加一个用于用户资料更新的 REST API 端点，并包含验证功能
```

**阶段 0 检测到：** `go.mod`，使用 Go 1.22, Chi 路由器

**优化后的提示词（完整版）：**

```
添加一个用于用户资料更新的 REST API 端点（PATCH /api/users/:id）。

技术栈：Go 1.22 + Chi 路由器（根据项目检测得出）

要求：
- PATCH /api/users/:id — 用户资料的部分更新
- 对字段进行输入验证：name、email、avatar_url、bio
- 认证：需要有效令牌，用户只能更新自己的资料
- 成功时返回 200 及更新后的用户信息
- 输入无效时返回 400 及验证错误
- 认证失败时返回 401/403
- 遵循代码库中现有的 API 模式

工作流程：
1. /plan 端点结构、中间件链和验证逻辑
2. /tdd — 为成功、验证失败、认证失败、未找到等情况编写表格驱动测试
3. 遵循现有的处理器模式进行实现
4. /go-review
5. /verify — 运行完整的测试套件，确认没有回归问题

禁止：
- 修改现有端点
- 更改数据库模式（使用现有的用户表）
- 未先检查现有依赖项就添加新依赖（使用 search-first 技能）
```

### 示例 3：史诗级项目

**用户输入：**

```
将我们的单体应用迁移到微服务
```

**优化后的提示词（完整版）：**

```
使用蓝图技能来规划："将单体应用迁移到微服务架构"

在执行之前，在蓝图中回答以下问题：
1. 当前单体应用中存在哪些领域边界？
2. 应该首先提取哪个服务（耦合度最低）？
3. 通信模式：REST API、gRPC 还是事件驱动（Kafka/RabbitMQ）？
4. 数据库策略：初期共享数据库，还是一开始就采用每个服务独立的数据库？
5. 部署目标：Kubernetes、Docker Compose 还是无服务器架构？

蓝图应产生如下阶段：
- 阶段 1：识别服务边界并创建领域映射
- 阶段 2：设置基础设施（API 网关、服务网格、每个服务的 CI/CD）
- 阶段 3：提取第一个服务（绞杀者模式）
- 阶段 4：通过集成测试验证，然后提取下一个服务
- 阶段 N：停用单体应用

每个阶段 = 1 个 PR，阶段之间设置 /verify 检查点。
在阶段之间使用 /save-session。使用 /resume-session 继续。
在依赖关系允许的情况下，使用 git worktrees 进行并行服务提取。

推荐：Opus 4.6 用于蓝图规划，Sonnet 4.6 用于阶段执行。
```

***

## 相关组件

| 组件 | 何时引用 |
|-----------|------------------|
| `configure-ecc` | 用户尚未设置 ECC |
| `skill-stocktake` | 审计安装了哪些组件（使用它而不是硬编码的目录） |
| `search-first` | 优化提示词中的研究阶段 |
| `blueprint` | 史诗级范围的优化提示词（作为技能调用，而不是命令） |
| `strategic-compact` | 长会话上下文管理 |
| `cost-aware-llm-pipeline` | 令牌优化推荐 |
