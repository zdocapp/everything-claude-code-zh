# 更新日志

## 1.10.0 - 2026-04-05

### 亮点

* 公开版本界面已与活跃仓库同步，经过数周的开源增长和积压合并。
* 操作员工作流通道扩展，新增语音、图排序、计费、工作区和外发技能。
* 媒体生成通道扩展，新增 Manim 和 Remotion-first 启动工具。
* ECC 2.0 alpha 控制平面二进制文件现已可从 `ecc2/` 本地构建，并公开了首个可用的 CLI/TUI 界面。

### 发布界面

* 将插件、市场、Codex、OpenCode 和代理元数据更新至 `1.10.0`。
* 同步了已发布数量至活跃的开源界面：38 个代理，156 项技能，72 条命令。
* 更新了面向安装的顶层文档和市场描述，以匹配当前仓库状态。

### 新工作流通道

* `brand-voice` — 源自规范源的写作风格系统。
* `social-graph-ranker` — 加权暖介绍图排序原语。
* `connections-optimizer` — 基于图排序的网络修剪/添加工作流。
* `customer-billing-ops`, `google-workspace-ops`, `project-flow-ops`, `workspace-surface-audit`。
* `manim-video`, `remotion-video-creation`, `nestjs-patterns`。

### ECC 2.0 Alpha

* `cargo build --manifest-path ecc2/Cargo.toml` 通过了仓库基线测试。
* `ecc-tui` 当前公开了 `dashboard`, `start`, `sessions`, `status`, `stop`, `resume`, 和 `daemon`。
* 该 alpha 版本真实可用，可用于本地实验，但更广泛的控制平面路线图仍未完成，不应视为正式发布版本。

### 备注

* Claude 插件仍受平台级规则分发限制；选择性安装 / 开源路径仍是最可靠的完整安装方式。
* 此版本是仓库界面修正和生态系统同步，并非声称完整的 ECC 2.0 路线图已完成。

## 1.9.0 - 2026-03-20

### 亮点

* 选择性安装架构，包含清单驱动的流水线和 SQLite 状态存储。
* 语言覆盖范围扩展至 10+ 个生态系统，新增 6 个代理和特定语言规则。
* 观察者可靠性增强，包含内存限制、沙箱修复和 5 层循环防护。
* 自改进技能基础，包含技能演进和会话适配器。

### 新代理

* `typescript-reviewer` — TypeScript/JavaScript 代码审查专家 (#647)
* `pytorch-build-resolver` — PyTorch 运行时、CUDA 和训练错误解决 (#549)
* `java-build-resolver` — Maven/Gradle 构建错误解决 (#538)
* `java-reviewer` — Java 和 Spring Boot 代码审查 (#528)
* `kotlin-reviewer` — Kotlin/Android/KMP 代码审查 (#309)
* `kotlin-build-resolver` — Kotlin/Gradle 构建错误 (#309)
* `rust-reviewer` — Rust 代码审查 (#523)
* `rust-build-resolver` — Rust 构建错误解决 (#523)
* `docs-lookup` — 文档和 API 参考研究 (#529)

### 新技能

* `pytorch-patterns` — PyTorch 深度学习工作流 (#550)
* `documentation-lookup` — API 参考和库文档研究 (#529)
* `bun-runtime` — Bun 运行时模式 (#529)
* `nextjs-turbopack` — Next.js Turbopack 工作流 (#529)
* `mcp-server-patterns` — MCP 服务器设计模式 (#531)
* `data-scraper-agent` — AI 驱动的公共数据收集 (#503)
* `team-builder` — 团队组成技能 (#501)
* `ai-regression-testing` — AI 回归测试工作流 (#433)
* `claude-devfleet` — 多代理编排 (#505)
* `blueprint` — 多会话构建规划
* `everything-claude-code` — 自引用 ECC 技能 (#335)
* `prompt-optimizer` — 提示优化技能 (#418)
* 8 项 Evos 运营领域技能 (#290)
* 3 项 Laravel 技能 (#420)
* VideoDB 技能 (#301)

### 新命令

* `/docs` — 文档查找 (#530)
* `/aside` — 侧边对话 (#407)
* `/prompt-optimize` — 提示优化 (#418)
* `/resume-session`, `/save-session` — 会话管理
* `learn-eval` 改进，包含基于检查表的整体判定

### 新规则

* Java 语言规则 (#645)
* PHP 规则包 (#389)
* Perl 语言规则和技能（模式、安全、测试）
* Kotlin/Android/KMP 规则 (#309)
* C++ 语言支持 (#539)
* Rust 语言支持 (#523)

### 基础设施

* 选择性安装架构，包含清单解析 (`install-plan.js`, `install-apply.js`) (#509, #512)
* SQLite 状态存储，包含用于跟踪已安装组件的查询 CLI (#510)
* 用于结构化会话记录的会话适配器 (#511)
* 用于自改进技能的技能演进基础 (#514)
* 具有确定性评分的编排工具 (#524)
* CI 中的目录计数强制执行 (#525)
* 所有 109 项技能的安装清单验证 (#537)
* PowerShell 安装程序包装器 (#532)
* 通过 `--target antigravity` 标志支持 Antigravity IDE (#332)
* Codex CLI 自定义脚本 (#336)

### 错误修复

* 解决了 6 个文件中的 19 个 CI 测试失败 (#519)
* 修复了安装流水线、编排器和修复中的 8 个测试失败 (#564)
* 观察者内存爆炸问题，通过限制、重入防护和尾部采样解决 (#536)
* 观察者沙箱访问修复，用于 Haiku 调用 (#661)
* 工作树项目 ID 不匹配修复 (#665)
* 观察者延迟启动逻辑 (#508)
* 观察者 5 层循环预防防护 (#399)
* 钩子可移植性和 Windows .cmd 支持
* Biome 钩子优化 — 消除了 npx 开销 (#359)
* InsAIts 安全钩子设为可选 (#370)
* Windows spawnSync 导出修复 (#431)
* instinct CLI 的 UTF-8 编码修复 (#353)
* 钩子中的密钥清理 (#348)

### 翻译

* 韩语 (ko-KR) 翻译 — README、代理、命令、技能、规则 (#392)
* 中文 (zh-CN) 文档同步 (#428)

### 致谢

* @ymdvsymd — 观察者沙箱和工作树修复
* @pythonstrup — biome 钩子优化
* @Nomadu27 — InsAIts 安全钩子
* @hahmee — 韩语翻译
* @zdocapp — 中文翻译同步
* @cookiee339 — Kotlin 生态系统
* @pangerlkr — CI 工作流修复
* @0xrohitgarg — VideoDB 技能
* @nocodemf — Evos 运营技能
* @swarnika-cmd — 社区贡献

## 1.8.0 - 2026-03-04

### 亮点

* 以工具链为首要的发布，专注于可靠性、评估纪律和自主循环操作。
* 钩子运行时现在支持基于配置文件的控制和定向钩子禁用。
* NanoClaw v2 增加了模型路由、技能热加载、分支、搜索、压缩、导出和指标。

### 核心

* 新增命令：`/harness-audit`, `/loop-start`, `/loop-status`, `/quality-gate`, `/model-route`。
* 新增技能：
* `agent-harness-construction`
* `agentic-engineering`
* `ralphinho-rfc-pipeline`
* `ai-first-engineering`
* `enterprise-agent-ops`
* `nanoclaw-repl`
* `continuous-agent-loop`
* 新增代理：
* `harness-optimizer`
* `loop-operator`

### 钩子可靠性

* 通过健壮的回退搜索修复了 SessionStart 根目录解析。
* 将会话摘要持久化移至 `Stop`，此处可获得转录负载。
* 增加了质量门和成本跟踪器钩子。
* 将脆弱的单行内联钩子替换为专用脚本文件。
* 增加了 `ECC_HOOK_PROFILE` 和 `ECC_DISABLED_HOOKS` 控制。

### 跨平台

* 改进了文档警告逻辑中 Windows 安全的路径处理。
* 强化了观察者循环行为，以避免非交互式挂起。

### 备注

* `autonomous-loops` 作为一个兼容性别名保留一个版本；`continuous-agent-loop` 是规范名称。

### 致谢

* 灵感来自 [zarazhangrui](https://github.com/zarazhangrui)
* 灵感来自 [humanplane](https://github.com/humanplane) 的 homunculus
