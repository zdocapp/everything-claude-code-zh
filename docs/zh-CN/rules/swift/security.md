---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 安全

> 本文档扩展了 [common/security.md](../common/security.md) 中与 Swift 相关的内容。

## 密钥管理

* 使用 **Keychain Services** 存储敏感数据（令牌、密码、密钥）—— 切勿使用 `UserDefaults`
* 使用环境变量或 `.xcconfig` 文件存储构建时密钥
* 切勿在源代码中硬编码密钥 —— 反编译工具可以轻易提取它们

```swift
let apiKey = ProcessInfo.processInfo.environment["API_KEY"]
guard let apiKey, !apiKey.isEmpty else {
    fatalError("API_KEY not configured")
}
```

## 传输安全

* 应用传输安全 (ATS) 默认强制执行 —— 不要禁用它
* 对关键端点使用证书锁定
* 验证所有服务器证书

## 输入验证

* 在显示所有用户输入之前进行清理，以防止注入攻击
* 使用带有验证的 `URL(string:)`，而非强制解包
* 在处理来自外部源（API、深度链接、剪贴板）的数据之前进行验证
