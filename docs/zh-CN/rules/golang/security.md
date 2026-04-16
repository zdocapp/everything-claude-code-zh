---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 安全

> 本文档在 [common/security.md](../common/security.md) 的基础上扩展了 Go 语言特定的内容。

## 密钥管理

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY not configured")
}
```

## 安全扫描

* 使用 **gosec** 进行静态安全分析：
  ```bash
  gosec ./...
  ```

## 上下文与超时

始终使用 `context.Context` 进行超时控制：

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```
