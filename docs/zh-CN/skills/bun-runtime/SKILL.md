---
name: bun-runtime
description: Bun 作为运行时、包管理器、打包器和测试运行器。何时选择 Bun 与 Node、迁移注意事项以及 Vercel 支持。
origin: ECC
---

# Bun 运行时

Bun 是一个快速的全能 JavaScript 运行时和工具包：包含运行时、包管理器、打包器和测试运行器。

## 何时使用

* **首选 Bun** 用于：新的 JS/TS 项目、安装/运行速度至关重要的脚本、使用 Bun 运行时的 Vercel 部署，以及当你想要一个单一工具链（运行 + 安装 + 测试 + 构建）时。
* **首选 Node** 用于：追求最大的生态系统兼容性、依赖假定为 Node 的遗留工具，或者当某个依赖项存在已知的 Bun 问题时。

适用于：采用 Bun、从 Node 迁移、编写或调试 Bun 脚本/测试，或在 Vercel 或其他平台上配置 Bun。

## 工作原理

* **运行时**：即插即用的 Node 兼容运行时（基于 JavaScriptCore，使用 Zig 实现）。
* **包管理器**：`bun install` 比 npm/yarn 快得多。在当前 Bun 版本中，默认锁文件是 `bun.lock`（文本格式）；旧版本使用 `bun.lockb`（二进制格式）。
* **打包器**：内置用于应用程序和库的打包器和转译器。
* **测试运行器**：内置 `bun test`，提供类似 Jest 的 API。

**从 Node 迁移**：将 `node script.js` 替换为 `bun run script.js` 或 `bun script.js`。运行 `bun install` 代替 `npm install`；大多数包都能正常工作。使用 `bun run` 执行 npm 脚本；使用 `bun x` 进行类似 npx 的一次性运行。支持 Node 内置模块；在存在的情况下优先使用 Bun API 以获得更好的性能。

**Vercel**：在项目设置中将运行时设置为 Bun。构建命令：`bun run build` 或 `bun build ./src/index.ts --outdir=dist`。安装命令：`bun install --frozen-lockfile` 以实现可重复的部署。

## 示例

### 运行和安装

```bash
# Install dependencies (creates/updates bun.lock or bun.lockb)
bun install

# Run a script or file
bun run dev
bun run src/index.ts
bun src/index.ts
```

### 脚本和环境变量

```bash
bun run --env-file=.env dev
FOO=bar bun run script.ts
```

### 测试

```bash
bun test
bun test --watch
```

```typescript
// test/example.test.ts
import { expect, test } from "bun:test";

test("add", () => {
  expect(1 + 2).toBe(3);
});
```

### 运行时 API

```typescript
const file = Bun.file("package.json");
const json = await file.json();

Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Hello");
  },
});
```

## 最佳实践

* 提交锁文件（`bun.lock` 或 `bun.lockb`）以实现可重复的安装。
* 脚本中优先使用 `bun run`。对于 TypeScript，Bun 原生运行 `.ts`。
* 保持依赖项最新；Bun 及其生态系统发展迅速。
