# SwiftUI Liquid Glass Patterns

## Basic Glass Effect
```swift
import SwiftUI

// Default capsule shape
Text("Hello").padding().glassEffect()

// Rectangle with corner radius
Text("Hello").padding().glassEffect(in: .rect(cornerRadius: 16))

// Circle
Image(systemName: "star").frame(width: 60, height: 60).glassEffect(in: .circle)
```

## Glass Variants
```swift
// Regular (default)
view.glassEffect(.regular)

// With tint color (suggests prominence or category)
view.glassEffect(.regular.tint(.orange))

// Interactive (responds to touch and pointer)
view.glassEffect(.regular.interactive())

// Combined
view.glassEffect(.regular.tint(.blue).interactive())

// Clear variant (subtle)
view.glassEffect(.clear)
```

## GlassEffectContainer

Required when using multiple glass effects. Enables merging, improves rendering performance.

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80, height: 80)
            .font(.system(size: 36))
            .glassEffect()

        Image(systemName: "eraser.fill")
            .frame(width: 80, height: 80)
            .font(.system(size: 36))
            .glassEffect()
    }
}
```

The `spacing` parameter controls merge distance — glass effects merge when closer than this value.

## Morphing Transitions

Use `@Namespace` and `glassEffectID` for animated glass transitions when views appear/disappear.

```swift
struct MorphingExample: View {
    @State private var isExpanded = false
    @Namespace private var namespace

    var body: some View {
        GlassEffectContainer(spacing: 40) {
            HStack(spacing: 40) {
                Image(systemName: "scribble.variable")
                    .frame(width: 80, height: 80)
                    .font(.system(size: 36))
                    .glassEffect()
                    .glassEffectID("pencil", in: namespace)

                if isExpanded {
                    Image(systemName: "eraser.fill")
                        .frame(width: 80, height: 80)
                        .font(.system(size: 36))
                        .glassEffect()
                        .glassEffectID("eraser", in: namespace)
                }
            }
        }

        Button("Toggle") {
            withAnimation { isExpanded.toggle() }
        }
        .buttonStyle(.glass)
    }
}
```

## Glass Effect Union

Combine multiple views into a single glass shape (useful for dynamic layouts or views outside stacks).

```swift
struct UnionExample: View {
    @Namespace private var namespace
    let symbols = ["star", "heart", "leaf", "bolt"]

    var body: some View {
        GlassEffectContainer(spacing: 20) {
            HStack(spacing: 20) {
                ForEach(symbols.indices, id: \.self) { index in
                    Image(systemName: symbols[index])
                        .frame(width: 80, height: 80)
                        .font(.system(size: 36))
                        .glassEffect()
                        .glassEffectUnion(
                            id: index < 2 ? "group1" : "group2",
                            namespace: namespace
                        )
                }
            }
        }
    }
}
```

## Button Styles

```swift
// Standard glass button
Button("Secondary Action") { }
    .buttonStyle(.glass)

// Prominent glass button (more visual weight)
Button("Primary Action") { }
    .buttonStyle(.glassProminent)
```

## Custom Badge with Glass
```swift
struct GlassBadge: View {
    let symbol: String
    let color: Color

    var body: some View {
        ZStack {
            Image(systemName: "hexagon.fill")
                .foregroundColor(color)
                .font(.system(size: 50))
            Image(systemName: symbol)
                .foregroundColor(.white)
                .font(.system(size: 30))
        }
        .glassEffect(.regular, in: .rect(cornerRadius: 16))
    }
}

// Usage
GlassEffectContainer(spacing: 20) {
    HStack(spacing: 20) {
        GlassBadge(symbol: "star.fill", color: .blue)
        GlassBadge(symbol: "heart.fill", color: .red)
    }
}
```

## Scroll Extension Under Sidebar
```swift
ScrollView(.horizontal) {
    // Content
}
.scrollExtensionMode(.underSidebar)
```

## Best Practices
1. Always use `GlassEffectContainer` for multiple glass views
2. Apply `.glassEffect()` after layout modifiers (`.frame()`, `.padding()`)
3. Use `withAnimation { }` when changing view hierarchy for morphing
4. Add `.interactive()` to controls that should respond to input
5. Keep tint colors subtle (low alpha)
6. Use consistent shapes across your app
