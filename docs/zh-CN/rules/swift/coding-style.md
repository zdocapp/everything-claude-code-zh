---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 编码风格

> 本文档扩展了 [common/coding-style.md](../common/coding-style.md) 中关于 Swift 的特定内容。

## 格式化

* 使用 **SwiftFormat** 进行自动格式化，使用 **SwiftLint** 进行风格检查
* `swift-format` 已捆绑在 Xcode 16+ 中，可作为替代方案

## 不可变性

* 优先使用 `let` 而非 `var` — 将所有内容定义为 `let`，仅在编译器要求时才更改为 `var`
* 默认使用具有值语义的 `struct`；仅在需要标识或引用语义时使用 `class`

## 命名

遵循 [Apple API 设计指南](https://www.swift.org/documentation/api-design-guidelines/)：

* 在使用点保持清晰 — 省略不必要的词语
* 根据方法和属性的角色而非其类型来命名
* 使用 `static let` 而非全局常量

## 错误处理

使用类型化抛出（Swift 6+）和模式匹配：

```swift
func load(id: String) throws(LoadError) -> Item {
    guard let data = try? read(from: path) else {
        throw .fileNotFound(id)
    }
    return try decode(data)
}
```

## 并发

启用 Swift 6 严格并发检查。优先选择：

* 使用 `Sendable` 值类型来跨越隔离边界传递数据
* 使用 Actor 处理共享可变状态
* 结构化并发（`async let`, `TaskGroup`）优于非结构化的 `Task {}`
