---
name: accessibility
description: "Concrete accessibility implementations for SwiftUI apps on macOS and iOS. Covers: adaptive glass effects that fall back to solid backgrounds when Reduce Transparency is enabled, motion-safe animations that respect Reduce Motion, VoiceOver support (accessible labels, custom actions, rotor entries, state change announcements for both macOS and iOS), Dynamic Type with @ScaledMetric and adaptive grid layouts, keyboard navigation with FocusState and onKeyPress, Commands menus with keyboard shortcuts, high contrast mode detection, color-blind safe status indicators using accessibilityDifferentiateWithoutColor, and a complete testing checklist. Use when building any user-facing SwiftUI app to ensure it meets accessibility standards. Especially important when using Liquid Glass (which needs Reduce Transparency fallbacks) or custom animations (which need Reduce Motion handling)."
---

# Accessibility Patterns

Concrete implementations for VoiceOver, Dynamic Type, Reduce Motion, keyboard navigation, and color contrast. These patterns apply to both macOS and iOS SwiftUI apps.

## Critical Rules

- ❌ Never rely on color alone to convey information — always pair with icons or text
- ❌ Never use glass effects without a solid fallback for Reduce Transparency
- ❌ Never ship animations without Reduce Motion handling
- ❌ Never skip VoiceOver labels on interactive elements
- ✅ Every interactive element must have an accessibility label
- ✅ Every state change must be announced to VoiceOver
- ✅ Every animation must respect `accessibilityReduceMotion`
- ✅ Every glass effect must have a `accessibilityReduceTransparency` fallback

## Adaptive Glass Effect

Glass effects become unreadable when Reduce Transparency is enabled. Always provide a solid fallback.

```swift
struct AdaptiveGlassModifier: ViewModifier {
    @Environment(\.accessibilityReduceTransparency) var reduceTransparency

    let style: GlassEffectStyle
    let shape: AnyShape

    func body(content: Content) -> some View {
        if reduceTransparency {
            content
                .background(Color(.systemBackground))
                .clipShape(shape)
        } else {
            content
                .glassEffect(style, in: shape)
        }
    }
}

extension View {
    func adaptiveGlass(
        _ style: GlassEffectStyle = .regular,
        in shape: some Shape = .rect
    ) -> some View {
        modifier(AdaptiveGlassModifier(style: style, shape: AnyShape(shape)))
    }
}

// Usage
Text("Accessible Glass")
    .padding()
    .adaptiveGlass(.regular, in: .rect(cornerRadius: 12))
```

## Motion-Safe Animations

### ViewModifier Approach
```swift
struct MotionSafeAnimation<V: Equatable>: ViewModifier {
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    let animation: Animation
    let value: V

    func body(content: Content) -> some View {
        content.animation(reduceMotion ? .none : animation, value: value)
    }
}

extension View {
    func motionSafeAnimation<V: Equatable>(
        _ animation: Animation = .default,
        value: V
    ) -> some View {
        modifier(MotionSafeAnimation(animation: animation, value: value))
    }
}
```

### Conditional withAnimation
```swift
struct MorphingCard: View {
    @State private var isExpanded = false
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    var body: some View {
        VStack { /* content */ }
        .onChange(of: isExpanded) { _, newValue in
            if reduceMotion {
                // Instant change, no animation
            } else {
                withAnimation(.spring(response: 0.3)) {
                    // Animated change
                }
            }
        }
    }
}
```

## VoiceOver Support

### Accessible Labels
```swift
struct PromptRow: View {
    let prompt: Prompt

    var body: some View {
        HStack {
            Image(systemName: prompt.icon)
            VStack(alignment: .leading) {
                Text(prompt.title)
                Text(prompt.preview)
                    .font(.caption)
            }
        }
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(prompt.title), \(prompt.category?.name ?? "uncategorized")")
        .accessibilityHint("Double tap to copy prompt")
        .accessibilityAddTraits(.isButton)
    }
}
```

### Custom Actions
Expose multiple actions without requiring complex gesture discovery.

```swift
struct PromptCard: View {
    let prompt: Prompt
    var onCopy: () -> Void
    var onEdit: () -> Void
    var onDelete: () -> Void

    var body: some View {
        VStack { /* card content */ }
            .accessibilityElement(children: .combine)
            .accessibilityLabel(prompt.title)
            .accessibilityActions {
                Button("Copy to clipboard") { onCopy() }
                Button("Edit") { onEdit() }
                Button("Delete", role: .destructive) { onDelete() }
            }
    }
}
```

### Rotor Support
Let VoiceOver users navigate by category or filter.

```swift
struct PromptListView: View {
    let prompts: [Prompt]
    @State private var favorites: Set<UUID> = []

    var body: some View {
        List(prompts) { prompt in
            PromptRow(prompt: prompt)
        }
        .accessibilityRotor("Favorites") {
            ForEach(prompts.filter { favorites.contains($0.id) }) { prompt in
                AccessibilityRotorEntry(prompt.title, id: prompt.id)
            }
        }
        .accessibilityRotor("Categories") {
            ForEach(Category.all) { category in
                AccessibilityRotorEntry(category.name, id: category.id)
            }
        }
    }
}
```

### Announce State Changes
Always announce clipboard operations and other invisible state changes.

```swift
func copyPrompt(_ prompt: Prompt) {
    ClipboardService.shared.copy(prompt.content)

    #if os(iOS)
    UIAccessibility.post(
        notification: .announcement,
        argument: "Copied \(prompt.title) to clipboard"
    )
    #else
    NSAccessibility.post(
        element: NSApp.mainWindow as Any,
        notification: .announcementRequested,
        userInfo: [.announcement: "Copied \(prompt.title) to clipboard"]
    )
    #endif
}
```

## Dynamic Type

### Scaled Metrics
```swift
struct PromptDetailView: View {
    let prompt: Prompt
    @ScaledMetric(relativeTo: .body) var iconSize = 24
    @ScaledMetric(relativeTo: .title) var headerPadding = 16

    var body: some View {
        VStack(alignment: .leading, spacing: headerPadding) {
            HStack {
                Image(systemName: prompt.icon)
                    .font(.system(size: iconSize))
                Text(prompt.title)
                    .font(.title)
            }
            Text(prompt.content)
                .font(.body)
        }
        .padding()
    }
}
```

### Adaptive Grid Layouts
Reduce columns as text size increases to prevent truncation.

```swift
struct AdaptivePromptGrid: View {
    let prompts: [Prompt]
    @Environment(\.dynamicTypeSize) var dynamicTypeSize

    var columns: [GridItem] {
        if dynamicTypeSize >= .accessibility1 {
            return [GridItem(.flexible())]                                    // 1 column
        } else if dynamicTypeSize >= .xxxLarge {
            return [GridItem(.flexible()), GridItem(.flexible())]             // 2 columns
        } else {
            return Array(repeating: GridItem(.flexible()), count: 3)          // 3 columns
        }
    }

    var body: some View {
        LazyVGrid(columns: columns, spacing: 16) {
            ForEach(prompts) { prompt in
                PromptCard(prompt: prompt)
            }
        }
    }
}
```

## Keyboard Navigation (macOS)

### Focus Management with FocusState
```swift
struct QuickAccessView: View {
    @FocusState private var focusedPrompt: UUID?
    let prompts: [Prompt]

    var body: some View {
        VStack {
            ForEach(prompts) { prompt in
                PromptRow(prompt: prompt)
                    .focused($focusedPrompt, equals: prompt.id)
            }
        }
        .onKeyPress(.upArrow) {
            moveFocus(direction: -1)
            return .handled
        }
        .onKeyPress(.downArrow) {
            moveFocus(direction: 1)
            return .handled
        }
        .onKeyPress(.return) {
            if let id = focusedPrompt { selectPrompt(id: id) }
            return .handled
        }
    }

    func moveFocus(direction: Int) {
        guard let current = focusedPrompt,
              let index = prompts.firstIndex(where: { $0.id == current }) else {
            focusedPrompt = prompts.first?.id
            return
        }
        let newIndex = index + direction
        if prompts.indices.contains(newIndex) {
            focusedPrompt = prompts[newIndex].id
        }
    }
}
```

### Commands Menu with Keyboard Shortcuts
```swift
struct PromptManagerCommands: Commands {
    var body: some Commands {
        CommandGroup(after: .newItem) {
            Button("New Prompt") { }
                .keyboardShortcut("n", modifiers: .command)
            Button("New Category") { }
                .keyboardShortcut("n", modifiers: [.command, .shift])
        }

        CommandMenu("Prompts") {
            Button("Copy Selected") { }
                .keyboardShortcut("c", modifiers: [.command, .shift])
            Button("Quick Access") { }
                .keyboardShortcut(" ", modifiers: [.command, .shift])
        }
    }
}
```

## Color Contrast & Color Blindness

### High Contrast Mode
```swift
struct ContrastAwareText: View {
    let text: String
    @Environment(\.colorSchemeContrast) var contrast

    var body: some View {
        Text(text)
            .foregroundStyle(contrast == .increased ? .primary : .secondary)
    }
}
```

### Color-Blind Safe Status Indicators
Always pair color with icon and optional text label.

```swift
struct StatusIndicator: View {
    let status: Status
    @Environment(\.accessibilityDifferentiateWithoutColor) var differentiateWithoutColor

    enum Status { case success, warning, error }

    var body: some View {
        HStack {
            Image(systemName: iconName)
                .foregroundStyle(color)

            if differentiateWithoutColor {
                Text(statusText)
                    .font(.caption)
            }
        }
    }

    var iconName: String {
        switch status {
        case .success: "checkmark.circle.fill"
        case .warning: "exclamationmark.triangle.fill"
        case .error: "xmark.circle.fill"
        }
    }

    var color: Color {
        switch status {
        case .success: .green
        case .warning: .yellow
        case .error: .red
        }
    }

    var statusText: String {
        switch status {
        case .success: "Success"
        case .warning: "Warning"
        case .error: "Error"
        }
    }
}
```

## System Preferences Detection

Observable object that monitors all accessibility settings at runtime.

```swift
class AccessibilitySettings: ObservableObject {
    @Published var reduceTransparency: Bool
    @Published var reduceMotion: Bool
    @Published var differentiateWithoutColor: Bool
    @Published var isVoiceOverRunning: Bool

    init() {
        #if os(macOS)
        reduceTransparency = NSWorkspace.shared.accessibilityDisplayShouldReduceTransparency
        reduceMotion = NSWorkspace.shared.accessibilityDisplayShouldReduceMotion
        differentiateWithoutColor = NSWorkspace.shared.accessibilityDisplayShouldDifferentiateWithoutColor
        isVoiceOverRunning = NSWorkspace.shared.isVoiceOverEnabled
        #else
        reduceTransparency = UIAccessibility.isReduceTransparencyEnabled
        reduceMotion = UIAccessibility.isReduceMotionEnabled
        differentiateWithoutColor = UIAccessibility.shouldDifferentiateWithoutColor
        isVoiceOverRunning = UIAccessibility.isVoiceOverRunning
        #endif
        observeChanges()
    }

    private func observeChanges() {
        #if os(iOS)
        NotificationCenter.default.addObserver(
            forName: UIAccessibility.reduceTransparencyStatusDidChangeNotification,
            object: nil, queue: .main
        ) { [weak self] _ in
            self?.reduceTransparency = UIAccessibility.isReduceTransparencyEnabled
        }
        NotificationCenter.default.addObserver(
            forName: UIAccessibility.reduceMotionStatusDidChangeNotification,
            object: nil, queue: .main
        ) { [weak self] _ in
            self?.reduceMotion = UIAccessibility.isReduceMotionEnabled
        }
        #endif
    }
}
```

## Testing Checklist

### VoiceOver
- [ ] All interactive elements have `.accessibilityLabel`
- [ ] Custom actions available where context menus exist
- [ ] Logical reading order (test by swiping through elements)
- [ ] Announcements for state changes (copy, delete, save)
- [ ] Rotors configured for key navigation paths

### Dynamic Type
- [ ] All text scales via system fonts (no hardcoded sizes)
- [ ] Layout adapts at accessibility sizes (grid → single column)
- [ ] No truncation at `.accessibility3` size
- [ ] Touch targets remain ≥ 44pt at all sizes

### Reduce Motion
- [ ] All `withAnimation` calls gated on `accessibilityReduceMotion`
- [ ] Glass morphing effects simplified or removed
- [ ] No essential information conveyed only through animation
- [ ] Transitions fall back to opacity-only

### Reduce Transparency
- [ ] Glass effects have solid background fallback
- [ ] Sufficient contrast maintained without blur
- [ ] Readability preserved on all backgrounds

### Keyboard (macOS)
- [ ] All features accessible via keyboard alone
- [ ] Logical tab/focus order
- [ ] Visible focus indicators
- [ ] Standard shortcuts work (⌘N, ⌘F, ⌘,)
- [ ] Custom shortcuts registered in Commands menu
