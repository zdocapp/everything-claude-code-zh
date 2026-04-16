---
name: swift-protocol-di-testing
description: 基于协议的依赖注入，用于可测试的Swift代码——使用聚焦协议和Swift Testing来模拟文件系统、网络和外部API。
origin: ECC
---

# 基于协议的 Swift 依赖注入用于测试

通过将外部依赖（文件系统、网络、iCloud）抽象为小型、专注的协议，使 Swift 代码可测试的模式。支持无需 I/O 的确定性测试。

## 何时启用

* 编写访问文件系统、网络或外部 API 的 Swift 代码时
* 需要在不触发真实故障的情况下测试错误处理路径时
* 构建跨环境（应用、测试、SwiftUI 预览）工作的模块时
* 设计支持 Swift 并发（actor、Sendable）的可测试架构时

## 核心模式

### 1. 定义小型、专注的协议

每个协议仅处理一个外部关注点。

```swift
// File system access
public protocol FileSystemProviding: Sendable {
    func containerURL(for purpose: Purpose) -> URL?
}

// File read/write operations
public protocol FileAccessorProviding: Sendable {
    func read(from url: URL) throws -> Data
    func write(_ data: Data, to url: URL) throws
    func fileExists(at url: URL) -> Bool
}

// Bookmark storage (e.g., for sandboxed apps)
public protocol BookmarkStorageProviding: Sendable {
    func saveBookmark(_ data: Data, for key: String) throws
    func loadBookmark(for key: String) throws -> Data?
}
```

### 2. 创建默认（生产）实现

```swift
public struct DefaultFileSystemProvider: FileSystemProviding {
    public init() {}

    public func containerURL(for purpose: Purpose) -> URL? {
        FileManager.default.url(forUbiquityContainerIdentifier: nil)
    }
}

public struct DefaultFileAccessor: FileAccessorProviding {
    public init() {}

    public func read(from url: URL) throws -> Data {
        try Data(contentsOf: url)
    }

    public func write(_ data: Data, to url: URL) throws {
        try data.write(to: url, options: .atomic)
    }

    public func fileExists(at url: URL) -> Bool {
        FileManager.default.fileExists(atPath: url.path)
    }
}
```

### 3. 创建用于测试的模拟实现

```swift
public final class MockFileAccessor: FileAccessorProviding, @unchecked Sendable {
    public var files: [URL: Data] = [:]
    public var readError: Error?
    public var writeError: Error?

    public init() {}

    public func read(from url: URL) throws -> Data {
        if let error = readError { throw error }
        guard let data = files[url] else {
            throw CocoaError(.fileReadNoSuchFile)
        }
        return data
    }

    public func write(_ data: Data, to url: URL) throws {
        if let error = writeError { throw error }
        files[url] = data
    }

    public func fileExists(at url: URL) -> Bool {
        files[url] != nil
    }
}
```

### 4. 使用默认参数注入依赖

生产代码使用默认值；测试注入模拟对象。

```swift
public actor SyncManager {
    private let fileSystem: FileSystemProviding
    private let fileAccessor: FileAccessorProviding

    public init(
        fileSystem: FileSystemProviding = DefaultFileSystemProvider(),
        fileAccessor: FileAccessorProviding = DefaultFileAccessor()
    ) {
        self.fileSystem = fileSystem
        self.fileAccessor = fileAccessor
    }

    public func sync() async throws {
        guard let containerURL = fileSystem.containerURL(for: .sync) else {
            throw SyncError.containerNotAvailable
        }
        let data = try fileAccessor.read(
            from: containerURL.appendingPathComponent("data.json")
        )
        // Process data...
    }
}
```

### 5. 使用 Swift Testing 编写测试

```swift
import Testing

@Test("Sync manager handles missing container")
func testMissingContainer() async {
    let mockFileSystem = MockFileSystemProvider(containerURL: nil)
    let manager = SyncManager(fileSystem: mockFileSystem)

    await #expect(throws: SyncError.containerNotAvailable) {
        try await manager.sync()
    }
}

@Test("Sync manager reads data correctly")
func testReadData() async throws {
    let mockFileAccessor = MockFileAccessor()
    mockFileAccessor.files[testURL] = testData

    let manager = SyncManager(fileAccessor: mockFileAccessor)
    let result = try await manager.loadData()

    #expect(result == expectedData)
}

@Test("Sync manager handles read errors gracefully")
func testReadError() async {
    let mockFileAccessor = MockFileAccessor()
    mockFileAccessor.readError = CocoaError(.fileReadCorruptFile)

    let manager = SyncManager(fileAccessor: mockFileAccessor)

    await #expect(throws: SyncError.self) {
        try await manager.sync()
    }
}
```

## 最佳实践

* **单一职责**：每个协议应处理一个关注点——不要创建包含许多方法的“上帝协议”
* **Sendable 一致性**：当协议跨 actor 边界使用时需要
* **默认参数**：让生产代码默认使用真实实现；只有测试需要指定模拟对象
* **错误模拟**：设计具有可配置错误属性的模拟对象以测试失败路径
* **仅模拟边界**：模拟外部依赖（文件系统、网络、API），而非内部类型

## 应避免的反模式

* 创建覆盖所有外部访问的单一大型协议
* 模拟没有外部依赖的内部类型
* 使用 `#if DEBUG` 条件语句而非适当的依赖注入
* 与 actor 一起使用时忘记 `Sendable` 一致性
* 过度设计：如果类型没有外部依赖，则不需要协议

## 使用场景

* 任何触及文件系统、网络或外部 API 的 Swift 代码
* 测试在真实环境中难以触发的错误处理路径时
* 构建需要在应用、测试和 SwiftUI 预览上下文中工作的模块时
* 使用 Swift 并发（actor、结构化并发）且需要可测试架构的应用
