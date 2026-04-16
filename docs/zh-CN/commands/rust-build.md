---
description: 逐步修复 Rust 构建错误、借用检查器问题和依赖问题。调用 rust-build-resolver 代理进行最小化、精准的修复。
---

# Rust 构建与修复

此命令调用 **rust-build-resolver** 代理，以最小的更改增量修复 Rust 构建错误。

## 此命令的作用

1. **运行诊断**：执行 `cargo check`、`cargo clippy`、`cargo fmt --check`
2. **解析错误**：识别错误代码和受影响的文件
3. **增量修复**：一次修复一个错误
4. **验证每个修复**：每次更改后重新运行 `cargo check`
5. **报告摘要**：显示已修复的内容和剩余的问题

## 何时使用

在以下情况使用 `/rust-build`：

* `cargo build` 或 `cargo check` 因错误而失败
* `cargo clippy` 报告警告
* 借用检查器或生命周期错误阻碍编译
* Cargo 依赖项解析失败
* 拉取更改后导致构建失败

## 运行的诊断命令

```bash
# Primary build check
cargo check 2>&1

# Lints and suggestions
cargo clippy -- -D warnings 2>&1

# Formatting check
cargo fmt --check 2>&1

# Dependency issues
cargo tree --duplicates

# Security audit (if available)
if command -v cargo-audit >/dev/null; then cargo audit; else echo "cargo-audit not installed"; fi
```

## 示例会话

````text
User: /rust-build

Agent:
# Rust 构建问题解决

## 初始诊断

```bash
$ cargo check
error[E0502]: cannot borrow `map` as mutable because it is also borrowed as immutable
  --> src/service/cache.rs:25:5
error[E0308]: mismatched types
  --> src/handler/api.rs:42:12
error[E0425]: cannot find value `db` in this scope
  --> src/handler/api.rs:58:9
```

发现错误：3

## 修复 1：借用检查器错误

文件：src/service/cache.rs:25
错误：E0502 — 无法可变借用 `map`，因为它同时被不可变借用
原因：在可变插入期间，不可变借用仍然有效

```rust
// 更改：重构以在可变访问前结束不可变借用
if !map.contains_key("key") {
    map.insert("key".into(), default);
}
```

```bash
$ cargo check
# 剩余 2 个错误
```

## 修复 2：类型不匹配

文件：src/handler/api.rs:42
错误：E0308 — 类型不匹配：期望 `usize`，找到 `&str`

```rust
// 更改
let count = params.get("count");
// 为
let count: usize = params.get("count")
    .and_then(|s| s.parse().ok())
    .unwrap_or(0);
```

```bash
$ cargo check
# 剩余 1 个错误
```

## 修复 3：未解析的名称

文件：src/handler/api.rs:58
错误：E0425 — 在此作用域中找不到值 `db`
原因：缺少导入

```rust
// 添加
use crate::db;
```

```bash
$ cargo check
# 构建成功！
```

## 最终验证

```bash
$ cargo clippy -- -D warnings
# 无警告

$ cargo test
running 6 tests
test service::cache::tests::test_insert ... ok
test service::cache::tests::test_get ... ok
test handler::api::tests::test_parse_count ... ok
test handler::api::tests::test_missing_count ... ok
test handler::api::tests::test_db_import ... ok
test handler::api::tests::test_response ... ok

test result: ok. 6 passed; 0 failed; 0 ignored
```

## 总结

| 指标 | 数量 |
|--------|-------|
| 已修复的构建错误 | 3 |
| 已修复的 Clippy 警告 | 0 |
| 修改的文件 | 2 |
| 剩余问题 | 0 |

构建状态：成功
````

## 常见错误修复

| 错误 | 典型修复方法 |
|-------|-------------|
| `cannot borrow as mutable` | 重构以先结束不可变借用；仅在合理时克隆 |
| `does not live long enough` | 使用拥有类型或添加生命周期注解 |
| `cannot move out of` | 重构以获取所有权；仅作为最后手段进行克隆 |
| `mismatched types` | 添加 `.into()`、`as` 或显式转换 |
| `trait X not implemented` | 添加 `#[derive(Trait)]` 或手动实现 |
| `unresolved import` | 添加到 Cargo.toml 或修复 `use` 路径 |
| `cannot find value` | 添加导入或修复路径 |

## 修复策略

1. **构建错误优先** - 代码必须能够编译
2. **Clippy 警告其次** - 修复可疑结构
3. **格式化第三** - 符合 `cargo fmt`
4. **一次修复一个** - 验证每次更改
5. **最小化更改** - 不进行重构，只进行修复

## 停止条件

代理将在以下情况停止并报告：

* 同一错误在 3 次尝试后仍然存在
* 修复引入了更多错误
* 需要架构更改
* 借用检查器错误需要重新设计数据所有权

## 相关命令

* `/rust-test` - 构建成功后运行测试
* `/rust-review` - 审查代码质量
* `/verify` - 完整验证循环

## 相关

* 代理：`agents/rust-build-resolver.md`
* 技能：`skills/rust-patterns/`
