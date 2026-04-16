---
name: swift-actor-persistence
description: 使用Actor在Swift中实现线程安全的数据持久化——基于文件存储的内存缓存，通过设计消除数据竞争。
origin: ECC
---

# 使用 Swift Actors 实现线程安全的数据持久化

利用 Swift actors 构建线程安全数据持久化层的模式。结合内存缓存与文件存储，借助 actor 模型在编译时消除数据竞争。

## 适用场景

* 在 Swift 5.5+ 中构建数据持久化层
* 需要对共享可变状态进行线程安全访问
* 希望消除手动同步（锁、DispatchQueues）
* 构建具备本地存储的离线优先应用

## 核心模式

### 基于 Actor 的存储库

Actor 模型保证串行化访问 —— 无数据竞争，由编译器强制执行。

```swift
public actor LocalRepository<T: Codable & Identifiable> where T.ID == String {
    private var cache: [String: T] = [:]
    private let fileURL: URL

    public init(directory: URL = .documentsDirectory, filename: String = "data.json") {
        self.fileURL = directory.appendingPathComponent(filename)
        // Synchronous load during init (actor isolation not yet active)
        self.cache = Self.loadSynchronously(from: fileURL)
    }

    // MARK: - Public API

    public func save(_ item: T) throws {
        cache[item.id] = item
        try persistToFile()
    }

    public func delete(_ id: String) throws {
        cache[id] = nil
        try persistToFile()
    }

    public func find(by id: String) -> T? {
        cache[id]
    }

    public func loadAll() -> [T] {
        Array(cache.values)
    }

    // MARK: - Private

    private func persistToFile() throws {
        let data = try JSONEncoder().encode(Array(cache.values))
        try data.write(to: fileURL, options: .atomic)
    }

    private static func loadSynchronously(from url: URL) -> [String: T] {
        guard let data = try? Data(contentsOf: url),
              let items = try? JSONDecoder().decode([T].self, from: data) else {
            return [:]
        }
        return Dictionary(uniqueKeysWithValues: items.map { ($0.id, $0) })
    }
}
```

### 使用方法

由于 actor 隔离，所有调用自动变为异步：

```swift
let repository = LocalRepository<Question>()

// Read — fast O(1) lookup from in-memory cache
let question = await repository.find(by: "q-001")
let allQuestions = await repository.loadAll()

// Write — updates cache and persists to file atomically
try await repository.save(newQuestion)
try await repository.delete("q-001")
```

### 与 @Observable ViewModel 结合使用

```swift
@Observable
final class QuestionListViewModel {
    private(set) var questions: [Question] = []
    private let repository: LocalRepository<Question>

    init(repository: LocalRepository<Question> = LocalRepository()) {
        self.repository = repository
    }

    func load() async {
        questions = await repository.loadAll()
    }

    func add(_ question: Question) async throws {
        try await repository.save(question)
        questions = await repository.loadAll()
    }
}
```

## 关键设计决策

| 决策 | 理由 |
|----------|-----------|
| 使用 Actor（而非类 + 锁） | 编译器强制保证的线程安全，无需手动同步 |
| 内存缓存 + 文件持久化 | 从缓存快速读取，持久化写入磁盘 |
| 同步初始化加载 | 避免异步初始化的复杂性 |
| 使用 ID 作为键的字典 | 通过标识符实现 O(1) 查找 |
| 对 `Codable & Identifiable` 泛型化 | 可在任何模型类型中复用 |
| 原子文件写入（`.atomic`） | 防止崩溃时部分写入 |

## 最佳实践

* **对所有跨越 actor 边界的数据使用 `Sendable` 类型**
* **保持 actor 的公共 API 最小化** —— 仅暴露领域操作，而非持久化细节
* **使用 `.atomic` 写入** 以防止应用在写入过程中崩溃导致数据损坏
* **在 `init` 中同步加载** —— 异步初始化器为本地文件带来的益处微乎其微，却增加了复杂性
* **与 `@Observable` ViewModels 结合使用** 以实现响应式 UI 更新

## 应避免的反模式

* 在新的 Swift 并发代码中使用 `DispatchQueue` 或 `NSLock` 替代 actors
* 将内部缓存字典暴露给外部调用者
* 使文件 URL 可配置但未经验证
* 忘记所有 actor 方法调用都是 `await` —— 调用者必须处理异步上下文
* 使用 `nonisolated` 绕过 actor 隔离（违背了其初衷）

## 使用时机

* iOS/macOS 应用中的本地数据存储（用户数据、设置、缓存内容）
* 稍后同步到服务器的离线优先架构
* 任何被应用的多个部分并发访问的共享可变状态
* 用现代 Swift 并发替换基于 `DispatchQueue` 的遗留线程安全方案
