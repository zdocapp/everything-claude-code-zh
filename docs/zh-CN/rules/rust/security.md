---
paths:
  - "**/*.rs"
---

# Rust 安全

> 本文档在 [common/security.md](../common/security.md) 的基础上，补充了 Rust 相关的内容。

## 密钥管理

* 切勿在源代码中硬编码 API 密钥、令牌或凭证
* 使用环境变量：`std::env::var("API_KEY")`
* 启动时若缺少必需的密钥，应快速失败
* 将 `.env` 文件保存在 `.gitignore`

```rust
// BAD
const API_KEY: &str = "sk-abc123...";

// GOOD — environment variable with early validation
fn load_api_key() -> anyhow::Result<String> {
    std::env::var("PAYMENT_API_KEY")
        .context("PAYMENT_API_KEY must be set")
}
```

## SQL 注入防范

* 始终使用参数化查询——切勿将用户输入格式化为 SQL 字符串
* 使用支持绑定参数的查询构建器或 ORM（如 sqlx、diesel、sea-orm）

```rust
// BAD — SQL injection via format string
let query = format!("SELECT * FROM users WHERE name = '{name}'");
sqlx::query(&query).fetch_one(&pool).await?;

// GOOD — parameterized query with sqlx
// Placeholder syntax varies by backend: Postgres: $1  |  MySQL: ?  |  SQLite: $1
sqlx::query("SELECT * FROM users WHERE name = $1")
    .bind(&name)
    .fetch_one(&pool)
    .await?;
```

## 输入验证

* 在处理前，于系统边界处验证所有用户输入
* 利用类型系统强制不变量（使用 newtype 模式）
* 解析而非验证——在边界处将非结构化数据转换为有类型的结构体
* 拒绝无效输入，并提供清晰的错误信息

```rust
// Parse, don't validate — invalid states are unrepresentable
pub struct Email(String);

impl Email {
    pub fn parse(input: &str) -> Result<Self, ValidationError> {
        let trimmed = input.trim();
        let at_pos = trimmed.find('@')
            .filter(|&p| p > 0 && p < trimmed.len() - 1)
            .ok_or_else(|| ValidationError::InvalidEmail(input.to_string()))?;
        let domain = &trimmed[at_pos + 1..];
        if trimmed.len() > 254 || !domain.contains('.') {
            return Err(ValidationError::InvalidEmail(input.to_string()));
        }
        // For production use, prefer a validated email crate (e.g., `email_address`)
        Ok(Self(trimmed.to_string()))
    }

    pub fn as_str(&self) -> &str {
        &self.0
    }
}
```

## 不安全代码

* 尽量减少 `unsafe` 块——优先使用安全的抽象
* 每个 `unsafe` 块都必须附带 `// SAFETY:` 注释，解释其不变式
* 切勿为了方便而使用 `unsafe` 来绕过借用检查器
* 在代码审查时审核所有 `unsafe` 代码——若无正当理由，它就是一个危险信号
* 优先使用 `safe` 作为 C 库的 FFI 包装器

```rust
// GOOD — safety comment documents ALL required invariants
let widget: &Widget = {
    // SAFETY: `ptr` is non-null, aligned, points to an initialized Widget,
    // and no mutable references or mutations exist for its lifetime.
    unsafe { &*ptr }
};

// BAD — no safety justification
unsafe { &*ptr }
```

## 依赖项安全

* 运行 `cargo audit` 以扫描依赖项中已知的 CVE
* 运行 `cargo deny check` 以确保许可证和公告合规
* 使用 `cargo tree` 来审计传递依赖项
* 保持依赖项更新——设置 Dependabot 或 Renovate
* 尽量减少依赖项数量——在添加新 crate 前进行评估

```bash
# Security audit
cargo audit

# Deny advisories, duplicate versions, and restricted licenses
cargo deny check

# Inspect dependency tree
cargo tree
cargo tree -d  # Show duplicates only
```

## 错误信息

* 切勿在 API 响应中暴露内部路径、堆栈跟踪或数据库错误
* 在服务器端记录详细错误；向客户端返回通用信息
* 使用 `tracing` 或 `log` 进行结构化的服务器端日志记录

```rust
// Map errors to appropriate status codes and generic messages
// (Example uses axum; adapt the response type to your framework)
match order_service.find_by_id(id) {
    Ok(order) => Ok((StatusCode::OK, Json(order))),
    Err(ServiceError::NotFound(_)) => {
        tracing::info!(order_id = id, "order not found");
        Err((StatusCode::NOT_FOUND, "Resource not found"))
    }
    Err(e) => {
        tracing::error!(order_id = id, error = %e, "unexpected error");
        Err((StatusCode::INTERNAL_SERVER_ERROR, "Internal server error"))
    }
}
```

## 参考资料

有关不安全代码指南和所有权模式，请参阅技能：`rust-patterns`。
有关通用安全检查清单，请参阅技能：`security-review`。
