# AppKit Liquid Glass Patterns

## NSGlassEffectView — Basic Usage
```swift
import AppKit

let glassView = NSGlassEffectView(frame: NSRect(x: 20, y: 20, width: 200, height: 100))
glassView.cornerRadius = 16.0
glassView.tintColor = NSColor.systemBlue.withAlphaComponent(0.3)

// Set content via contentView property
let label = NSTextField(labelWithString: "Liquid Glass")
label.translatesAutoresizingMaskIntoConstraints = false
glassView.contentView = label

// Content is ONLY guaranteed inside contentView
// Arbitrary subviews may not have consistent z-order
```

## NSGlassEffectContainerView — Multiple Glass Elements
```swift
let container = NSGlassEffectContainerView(frame: NSRect(x: 20, y: 20, width: 400, height: 200))
container.spacing = 40.0  // Distance at which effects begin to merge

let contentView = NSView(frame: container.bounds)
contentView.autoresizingMask = [.width, .height]
container.contentView = contentView

let glass1 = NSGlassEffectView(frame: NSRect(x: 20, y: 50, width: 150, height: 100))
glass1.cornerRadius = 12.0

let glass2 = NSGlassEffectView(frame: NSRect(x: 190, y: 50, width: 150, height: 100))
glass2.cornerRadius = 12.0

contentView.addSubview(glass1)
contentView.addSubview(glass2)
```

## Hover Effects with Tracking Areas
```swift
class InteractiveGlassView: NSGlassEffectView {
    override init(frame frameRect: NSRect) {
        super.init(frame: frameRect)
        let options: NSTrackingArea.Options = [.mouseEnteredAndExited, .mouseMoved, .activeInActiveApp]
        let trackingArea = NSTrackingArea(rect: bounds, options: options, owner: self, userInfo: nil)
        addTrackingArea(trackingArea)
    }
    required init?(coder: NSCoder) { super.init(coder: coder); /* same setup */ }

    override func mouseEntered(with event: NSEvent) {
        NSAnimationContext.runAnimationGroup { context in
            context.duration = 0.2
            animator().tintColor = NSColor.systemBlue.withAlphaComponent(0.2)
        }
    }

    override func mouseExited(with event: NSEvent) {
        NSAnimationContext.runAnimationGroup { context in
            context.duration = 0.2
            animator().tintColor = nil
        }
    }
}
```

## Animated Merging
```swift
NSAnimationContext.runAnimationGroup { context in
    context.duration = 0.5
    context.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
    // Move glass2 closer to glass1 — triggers merge when within container spacing
    glass2.animator().frame = NSRect(x: 100, y: 50, width: 150, height: 100)
}
```

## Performance Notes
- Use NSGlassEffectContainerView for multiple glass views (reduces render passes)
- Limit glass effects — significant GPU cost
- Only contentView content is reliably inside the glass effect
