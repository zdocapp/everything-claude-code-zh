---
name: foundation-models-on-device
description: Apple FoundationModels 框架用于设备端 LLM — 文本生成、使用 @Generable 进行引导生成、工具调用，以及在 iOS 26+ 中的快照流式传输。
---

# FoundationModels：设备端大语言模型 (iOS 26)

使用 FoundationModels 框架将苹果的设备端语言模型集成到应用中的模式。涵盖文本生成、使用 `@Generable` 的结构化输出、自定义工具调用以及快照流式传输——全部在设备端运行，以支持隐私保护和离线使用。

## 何时启用

* 使用 Apple Intelligence 在设备端构建 AI 功能时
* 无需依赖云端即可生成或总结文本时
* 从自然语言输入中提取结构化数据时
* 为特定领域的 AI 操作实现自定义工具调用时
* 为实时 UI 更新流式传输结构化响应时
* 需要保护隐私的 AI（数据不会离开设备）时

## 核心模式 —— 可用性检查

在创建会话前，务必检查模型可用性：

```swift
struct GenerativeView: View {
    private var model = SystemLanguageModel.default

    var body: some View {
        switch model.availability {
        case .available:
            ContentView()
        case .unavailable(.deviceNotEligible):
            Text("Device not eligible for Apple Intelligence")
        case .unavailable(.appleIntelligenceNotEnabled):
            Text("Please enable Apple Intelligence in Settings")
        case .unavailable(.modelNotReady):
            Text("Model is downloading or not ready")
        case .unavailable(let other):
            Text("Model unavailable: \(other)")
        }
    }
}
```

## 核心模式 —— 基础会话

```swift
// Single-turn: create a new session each time
let session = LanguageModelSession()
let response = try await session.respond(to: "What's a good month to visit Paris?")
print(response.content)

// Multi-turn: reuse session for conversation context
let session = LanguageModelSession(instructions: """
    You are a cooking assistant.
    Provide recipe suggestions based on ingredients.
    Keep suggestions brief and practical.
    """)

let first = try await session.respond(to: "I have chicken and rice")
let followUp = try await session.respond(to: "What about a vegetarian option?")
```

指令的关键点：

* 定义模型的角色（"你是一位导师"）
* 指定要执行的操作（"帮助提取日历事件"）
* 设置风格偏好（"尽可能简短地回应"）
* 添加安全措施（"对于危险请求，请回应'我无法帮助处理此事'"）

## 核心模式 —— 使用 @Generable 进行引导生成

生成结构化的 Swift 类型，而非原始字符串：

### 1. 定义可生成类型

```swift
@Generable(description: "Basic profile information about a cat")
struct CatProfile {
    var name: String

    @Guide(description: "The age of the cat", .range(0...20))
    var age: Int

    @Guide(description: "A one sentence profile about the cat's personality")
    var profile: String
}
```

### 2. 请求结构化输出

```swift
let response = try await session.respond(
    to: "Generate a cute rescue cat",
    generating: CatProfile.self
)

// Access structured fields directly
print("Name: \(response.content.name)")
print("Age: \(response.content.age)")
print("Profile: \(response.content.profile)")
```

### 支持的 @Guide 约束

* `.range(0...20)` —— 数值范围
* `.count(3)` —— 数组元素数量
* `description:` —— 用于生成的语义引导

## 核心模式 —— 工具调用

让模型调用自定义代码以执行特定领域的任务：

### 1. 定义工具

```swift
struct RecipeSearchTool: Tool {
    let name = "recipe_search"
    let description = "Search for recipes matching a given term and return a list of results."

    @Generable
    struct Arguments {
        var searchTerm: String
        var numberOfResults: Int
    }

    func call(arguments: Arguments) async throws -> ToolOutput {
        let recipes = await searchRecipes(
            term: arguments.searchTerm,
            limit: arguments.numberOfResults
        )
        return .string(recipes.map { "- \($0.name): \($0.description)" }.joined(separator: "\n"))
    }
}
```

### 2. 创建带工具的会话

```swift
let session = LanguageModelSession(tools: [RecipeSearchTool()])
let response = try await session.respond(to: "Find me some pasta recipes")
```

### 3. 处理工具错误

```swift
do {
    let answer = try await session.respond(to: "Find a recipe for tomato soup.")
} catch let error as LanguageModelSession.ToolCallError {
    print(error.tool.name)
    if case .databaseIsEmpty = error.underlyingError as? RecipeSearchToolError {
        // Handle specific tool error
    }
}
```

## 核心模式 —— 快照流式传输

使用 `PartiallyGenerated` 类型为实时 UI 流式传输结构化响应：

```swift
@Generable
struct TripIdeas {
    @Guide(description: "Ideas for upcoming trips")
    var ideas: [String]
}

let stream = session.streamResponse(
    to: "What are some exciting trip ideas?",
    generating: TripIdeas.self
)

for try await partial in stream {
    // partial: TripIdeas.PartiallyGenerated (all properties Optional)
    print(partial)
}
```

### SwiftUI 集成

```swift
@State private var partialResult: TripIdeas.PartiallyGenerated?
@State private var errorMessage: String?

var body: some View {
    List {
        ForEach(partialResult?.ideas ?? [], id: \.self) { idea in
            Text(idea)
        }
    }
    .overlay {
        if let errorMessage { Text(errorMessage).foregroundStyle(.red) }
    }
    .task {
        do {
            let stream = session.streamResponse(to: prompt, generating: TripIdeas.self)
            for try await partial in stream {
                partialResult = partial
            }
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

## 关键设计决策

| 决策 | 理由 |
|----------|-----------|
| 设备端执行 | 隐私保护 —— 数据不会离开设备；支持离线工作 |
| 4,096 个令牌限制 | 设备端模型的限制；大型数据需跨会话分块处理 |
| 快照流式传输（非增量） | 对结构化输出友好；每个快照都是一个完整的部分状态 |
| `@Generable` 宏 | 为结构化生成提供编译时安全性；自动生成 `PartiallyGenerated` 类型 |
| 每个会话单次请求 | `isResponding` 防止并发请求；如需并发，请创建多个会话 |
| `response.content`（而非 `.output`） | 正确的 API —— 始终通过 `.content` 属性访问结果 |

## 最佳实践

* **始终在创建会话前检查 `model.availability`** —— 处理所有不可用的情况
* **使用 `instructions`** 来引导模型行为 —— 其优先级高于提示词
* **发送新请求前检查 `isResponding`** —— 会话一次处理一个请求
* **通过 `response.content` 访问结果** —— 而非 `.output`
* **将大型输入分块处理** —— 4,096 个令牌的限制适用于指令 + 提示词 + 输出的总和
* **使用 `@Generable` 进行结构化输出** —— 比解析原始字符串提供更强的保证
* **使用 `GenerationOptions(temperature:)` 调整创造性**（值越高越具创造性）
* **使用 Instruments 进行监控** —— 使用 Xcode Instruments 分析请求性能

## 应避免的反模式

* 未先检查 `model.availability` 就创建会话
* 发送超出 4,096 个令牌上下文窗口的输入
* 尝试在单个会话上进行并发请求
* 使用 `.output` 而非 `.content` 来访问响应数据
* 在 `@Generable` 结构化输出可行时，仍解析原始字符串响应
* 在单个提示词中构建复杂的多步骤逻辑 —— 应分解为多个聚焦的提示词
* 假设模型始终可用 —— 设备资格和设置各不相同

## 使用场景

* 为注重隐私的应用进行设备端文本生成
* 从用户输入（表单、自然语言命令）中提取结构化数据
* 必须离线工作的 AI 辅助功能
* 逐步显示生成内容的流式 UI
* 通过工具调用（搜索、计算、查找）执行特定领域的 AI 操作
