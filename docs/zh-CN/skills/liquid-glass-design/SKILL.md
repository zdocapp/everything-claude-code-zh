---
name: liquid-glass-design
description: iOS 26 Liquid Glass 设计系统 — 适用于 SwiftUI、UIKit 和 WidgetKit 的动态玻璃材质，具有模糊、反射和交互式变形效果。
---

# Liquid Glass 设计系统 (iOS 26)

实现苹果 Liquid Glass 的模式 —— 这是一种动态材质，会模糊其后的内容，反射周围内容的颜色和光线，并对触摸和指针交互做出反应。涵盖 SwiftUI、UIKit 和 WidgetKit 集成。

## 何时启用

* 为 iOS 26+ 构建或更新应用，并采用新的设计语言时
* 实现玻璃风格的按钮、卡片、工具栏或容器时
* 在玻璃元素之间创建变形过渡时
* 将 Liquid Glass 效果应用于小组件时
* 将现有的模糊/材质效果迁移到新的 Liquid Glass API 时

## 核心模式 — SwiftUI

### 基本玻璃效果

为任何视图添加 Liquid Glass 的最简单方法：

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect()  // Default: regular variant, capsule shape
```

### 自定义形状和色调

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive(), in: .rect(cornerRadius: 16.0))
```

关键自定义选项：

* `.regular` — 标准玻璃效果
* `.tint(Color)` — 添加颜色色调以增强突出度
* `.interactive()` — 对触摸和指针交互做出反应
* 形状：`.capsule`（默认）、`.rect(cornerRadius:)`、`.circle`

### 玻璃按钮样式

```swift
Button("Click Me") { /* action */ }
    .buttonStyle(.glass)

Button("Important") { /* action */ }
    .buttonStyle(.glassProminent)
```

### 用于多个元素的 GlassEffectContainer

出于性能和变形考虑，始终将多个玻璃视图包装在容器中：

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80.0, height: 80.0)
            .font(.system(size: 36))
            .glassEffect()

        Image(systemName: "eraser.fill")
            .frame(width: 80.0, height: 80.0)
            .font(.system(size: 36))
            .glassEffect()
    }
}
```

`spacing` 参数控制合并距离 —— 距离更近的元素会将其玻璃形状融合在一起。

### 合并玻璃效果

使用 `glassEffectUnion` 将多个视图合并为单个玻璃形状：

```swift
@Namespace private var namespace

GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(symbolSet.indices, id: \.self) { item in
            Image(systemName: symbolSet[item])
                .frame(width: 80.0, height: 80.0)
                .glassEffect()
                .glassEffectUnion(id: item < 2 ? "group1" : "group2", namespace: namespace)
        }
    }
}
```

### 变形过渡

在玻璃元素出现/消失时创建平滑变形：

```swift
@State private var isExpanded = false
@Namespace private var namespace

GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80.0, height: 80.0)
            .glassEffect()
            .glassEffectID("pencil", in: namespace)

        if isExpanded {
            Image(systemName: "eraser.fill")
                .frame(width: 80.0, height: 80.0)
                .glassEffect()
                .glassEffectID("eraser", in: namespace)
        }
    }
}

Button("Toggle") {
    withAnimation { isExpanded.toggle() }
}
.buttonStyle(.glass)
```

### 将水平滚动内容延伸至侧边栏下方

若要允许水平滚动内容延伸至侧边栏或检查器下方，请确保 `ScrollView` 内容到达容器的前缘/后缘。当布局延伸至边缘时，系统会自动处理侧边栏下方的滚动行为 —— 无需额外的修饰符。

## 核心模式 — UIKit

### 基本 UIGlassEffect

```swift
let glassEffect = UIGlassEffect()
glassEffect.tintColor = UIColor.systemBlue.withAlphaComponent(0.3)
glassEffect.isInteractive = true

let visualEffectView = UIVisualEffectView(effect: glassEffect)
visualEffectView.translatesAutoresizingMaskIntoConstraints = false
visualEffectView.layer.cornerRadius = 20
visualEffectView.clipsToBounds = true

view.addSubview(visualEffectView)
NSLayoutConstraint.activate([
    visualEffectView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    visualEffectView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
    visualEffectView.widthAnchor.constraint(equalToConstant: 200),
    visualEffectView.heightAnchor.constraint(equalToConstant: 120)
])

// Add content to contentView
let label = UILabel()
label.text = "Liquid Glass"
label.translatesAutoresizingMaskIntoConstraints = false
visualEffectView.contentView.addSubview(label)
NSLayoutConstraint.activate([
    label.centerXAnchor.constraint(equalTo: visualEffectView.contentView.centerXAnchor),
    label.centerYAnchor.constraint(equalTo: visualEffectView.contentView.centerYAnchor)
])
```

### 用于多个元素的 UIGlassContainerEffect

```swift
let containerEffect = UIGlassContainerEffect()
containerEffect.spacing = 40.0

let containerView = UIVisualEffectView(effect: containerEffect)

let firstGlass = UIVisualEffectView(effect: UIGlassEffect())
let secondGlass = UIVisualEffectView(effect: UIGlassEffect())

containerView.contentView.addSubview(firstGlass)
containerView.contentView.addSubview(secondGlass)
```

### 滚动边缘效果

```swift
scrollView.topEdgeEffect.style = .automatic
scrollView.bottomEdgeEffect.style = .hard
scrollView.leftEdgeEffect.isHidden = true
```

### 工具栏玻璃集成

```swift
let favoriteButton = UIBarButtonItem(image: UIImage(systemName: "heart"), style: .plain, target: self, action: #selector(favoriteAction))
favoriteButton.hidesSharedBackground = true  // Opt out of shared glass background
```

## 核心模式 — WidgetKit

### 渲染模式检测

```swift
struct MyWidgetView: View {
    @Environment(\.widgetRenderingMode) var renderingMode

    var body: some View {
        if renderingMode == .accented {
            // Tinted mode: white-tinted, themed glass background
        } else {
            // Full color mode: standard appearance
        }
    }
}
```

### 用于视觉层次结构的强调色组

```swift
HStack {
    VStack(alignment: .leading) {
        Text("Title")
            .widgetAccentable()  // Accent group
        Text("Subtitle")
            // Primary group (default)
    }
    Image(systemName: "star.fill")
        .widgetAccentable()  // Accent group
}
```

### 强调色模式下的图像渲染

```swift
Image("myImage")
    .widgetAccentedRenderingMode(.monochrome)
```

### 容器背景

```swift
VStack { /* content */ }
    .containerBackground(for: .widget) {
        Color.blue.opacity(0.2)
    }
```

## 关键设计决策

| 决策 | 理由 |
|----------|-----------|
| 使用 GlassEffectContainer 包装 | 性能优化，支持玻璃元素之间的变形 |
| `spacing` 参数 | 控制合并距离 —— 微调元素需要多近才能融合 |
| `@Namespace` + `glassEffectID` | 在视图层次结构变化时实现平滑的变形过渡 |
| `interactive()` 修饰符 | 明确选择启用触摸/指针反应 —— 并非所有玻璃都应响应 |
| UIKit 中的 UIGlassContainerEffect | 与 SwiftUI 保持一致的容器模式 |
| 小组件中的强调色渲染模式 | 当用户选择带色调的主屏幕时，系统会应用带色调的玻璃效果 |

## 最佳实践

* **始终使用 GlassEffectContainer** 将玻璃效果应用于多个同级视图 —— 它支持变形并提高渲染性能
* **在其他外观修饰符**（frame、font、padding）**之后应用** `.glassEffect()`
* **仅对响应用户交互的元素**（按钮、可切换项目）**使用** `.interactive()`
* **在容器中仔细选择间距**以控制玻璃效果何时合并
* **在更改视图层次结构时使用** `withAnimation` 以启用平滑的变形过渡
* **在各种外观下测试** —— 浅色模式、深色模式和强调色/色调模式
* **确保无障碍对比度** —— 玻璃上的文本必须保持可读性

## 应避免的反模式

* 使用多个独立的 `.glassEffect()` 视图而没有 GlassEffectContainer
* 嵌套太多玻璃效果 —— 会降低性能和视觉清晰度
* 对每个视图都应用玻璃效果 —— 保留给交互元素、工具栏和卡片使用
* 在 UIKit 中使用圆角半径时忘记 `clipsToBounds = true`
* 忽略小组件中的强调色渲染模式 —— 会破坏带色调的主屏幕外观
* 在玻璃后面使用不透明背景 —— 破坏了半透明效果

## 何时使用

* 采用 iOS 26 新设计的导航栏、工具栏和标签栏
* 浮动操作按钮和卡片式容器
* 需要视觉深度和触摸反馈的交互式控件
* 应与系统的 Liquid Glass 外观集成的小组件
* 相关 UI 状态之间的变形过渡
