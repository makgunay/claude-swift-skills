# WidgetKit Liquid Glass Patterns

## Rendering Modes

Widgets can appear in two modes:
- **Full Color**: Default — all colors, images, transparency as designed
- **Accented**: When user chooses tinted/clear Home Screen appearance — content tinted white, background replaced with glass

## Detecting Rendering Mode
```swift
struct MyWidgetView: View {
    @Environment(\.widgetRenderingMode) var renderingMode

    var body: some View {
        if renderingMode == .accented {
            // Accented mode layout
        } else {
            // Full color layout
        }
    }
}
```

## Marking Accent Content
```swift
HStack {
    VStack(alignment: .leading) {
        Text("Title")
            .font(.headline)
            .widgetAccentable()  // Accent group — tinted in accented mode
        Text("Subtitle")
        // Primary group — default
    }
    Image(systemName: "star.fill")
        .widgetAccentable()  // Accent group
}
```

## Image Rendering in Accented Mode
```swift
Image("myImage")
    .widgetAccentedRenderingMode(.monochrome)
```

## Container Background
```swift
var body: some View {
    VStack { /* content */ }
        .containerBackground(for: .widget) {
            Color.blue.opacity(0.2)
        }
}

// Prevent background removal (excludes from iPad Lock Screen, StandBy)
var body: some WidgetConfiguration {
    StaticConfiguration(kind: "MyWidget", provider: Provider()) { entry in
        MyWidgetView(entry: entry)
    }
    .containerBackgroundRemovable(false)
}
```

## Glass Effects in Widgets
```swift
// Same SwiftUI API works in widgets
Text("Custom Element").padding()
    .glassEffect()

GlassEffectContainer(spacing: 20) {
    HStack(spacing: 20) {
        Image(systemName: "cloud").frame(width: 60, height: 60).glassEffect()
        Image(systemName: "sun").frame(width: 60, height: 60).glassEffect()
    }
}
```

## visionOS Widget Texture
```swift
.widgetTexture(.glass)  // Default
.widgetTexture(.paper)  // Paper-like
```

## visionOS Proximity Awareness
```swift
@Environment(\.levelOfDetail) var levelOfDetail

var fontSize: Font {
    levelOfDetail == .simplified ? .largeTitle : .title
}
```

## Best Practices
1. Display full-color images only in `fullColor` mode
2. Use `widgetAccentable()` strategically for visual hierarchy
3. Test with different accent colors and backgrounds
4. Test in StandBy mode on compatible devices
