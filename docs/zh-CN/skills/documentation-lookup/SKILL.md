---
name: documentation-lookup
description: 通过Context7 MCP使用最新的库和框架文档，而非训练数据。适用于设置问题、API参考、代码示例，或当用户提及框架名称时（例如React、Next.js、Prisma）。
origin: ECC
---

# 文档查询（Context7）

当用户询问库、框架或 API 时，通过 Context7 MCP（工具 `resolve-library-id` 和 `query-docs`）获取最新文档，而非依赖训练数据。

## 核心概念

* **Context7**：提供实时文档的 MCP 服务器；用于库和 API 查询，替代训练数据。
* **resolve-library-id**：根据库名称和查询返回 Context7 兼容的库 ID（例如 `/vercel/next.js`）。
* **query-docs**：根据给定的库 ID 和问题获取文档和代码片段。务必先调用 resolve-library-id 以获取有效的库 ID。

## 使用时机

在以下用户场景时激活：

* 询问设置或配置问题（例如“如何配置 Next.js 中间件？”）
* 请求依赖某个库的代码（“编写一个 Prisma 查询用于...”）
* 需要 API 或参考信息（“Supabase 的认证方法有哪些？”）
* 提及特定框架或库（React、Vue、Svelte、Express、Tailwind、Prisma、Supabase 等）

当请求依赖于库、框架或 API 的准确、最新行为时，请使用此技能。适用于配置了 Context7 MCP 的所有环境（例如 Claude Code、Cursor、Codex）。

## 工作原理

### 步骤 1：解析库 ID

调用 **resolve-library-id** MCP 工具，参数如下：

* **libraryName**：从用户问题中提取的库或产品名称（例如 `Next.js`、`Prisma`、`Supabase`）。
* **query**：用户的完整问题。这有助于提高结果的相关性排名。

在查询文档之前，必须获取一个 Context7 兼容的库 ID（格式为 `/org/project` 或 `/org/project/version`）。如果没有从此步骤获得有效的库 ID，请勿调用 query-docs。

### 步骤 2：选择最佳匹配

从解析结果中，根据以下原则选择一个结果：

* **名称匹配**：优先选择与用户询问内容完全匹配或最接近的。
* **基准分数**：分数越高表示文档质量越好（最高为 100）。
* **来源声誉**：在可用时，优先选择声誉为“高”或“中”的来源。
* **版本**：如果用户指定了版本（例如“React 19”、“Next.js 15”），则优先选择列出的版本特定库 ID（例如 `/org/project/v1.2.0`）。

### 步骤 3：获取文档

调用 **query-docs** MCP 工具，参数如下：

* **libraryId**：从步骤 2 中选择的 Context7 库 ID（例如 `/vercel/next.js`）。
* **query**：用户的具体问题或任务。具体描述以获得相关片段。

限制：每个问题调用 query-docs（或 resolve-library-id）的次数不得超过 3 次。如果 3 次调用后答案仍不明确，请说明不确定性并使用已有的最佳信息，而非猜测。

### 步骤 4：使用文档

* 使用获取到的最新信息回答用户的问题。
* 在有帮助时，包含文档中的相关代码示例。
* 在重要时引用库或版本（例如“在 Next.js 15 中...”）。

## 示例

### 示例：Next.js 中间件

1. 调用 **resolve-library-id**，参数为 `libraryName: "Next.js"`、`query: "How do I set up Next.js middleware?"`。
2. 从结果中，根据名称和基准分数选择最佳匹配（例如 `/vercel/next.js`）。
3. 调用 **query-docs**，参数为 `libraryId: "/vercel/next.js"`、`query: "How do I set up Next.js middleware?"`。
4. 使用返回的片段和文本来回答；如果相关，包含一个来自文档的最小化 `middleware.ts` 示例。

### 示例：Prisma 查询

1. 调用 **resolve-library-id**，参数为 `libraryName: "Prisma"`、`query: "How do I query with relations?"`。
2. 选择官方的 Prisma 库 ID（例如 `/prisma/prisma`）。
3. 使用该 `libraryId` 和查询调用 **query-docs**。
4. 返回 Prisma Client 模式（例如 `include` 或 `select`）以及来自文档的简短代码片段。

### 示例：Supabase 认证方法

1. 调用 **resolve-library-id**，参数为 `libraryName: "Supabase"`、`query: "What are the auth methods?"`。
2. 选择 Supabase 文档库 ID。
3. 调用 **query-docs**；根据获取的文档总结认证方法并展示最小化示例。

## 最佳实践

* **具体明确**：尽可能使用用户的完整问题作为查询，以获得更好的相关性。
* **版本意识**：当用户提及版本时，在解析步骤中优先使用可用的版本特定库 ID。
* **优先官方来源**：当存在多个匹配项时，优先选择官方或主要包，而非社区分支。
* **无敏感数据**：从发送给 Context7 的任何查询中移除 API 密钥、密码、令牌和其他机密信息。在将用户问题传递给 resolve-library-id 或 query-docs 之前，将其视为可能包含机密信息进行处理。
