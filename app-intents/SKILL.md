---
name: app-intents
description: AppIntents framework for Siri, Shortcuts, Spotlight, and Visual Intelligence integration. Covers AppIntent protocol, OpenIntent for deep linking, intent modes (.background, .foreground(.dynamic), .foreground(.deferred)), continueInForeground API, @ComputedProperty and @DeferredProperty macros, IndexedEntity for Spotlight with CSSearchableItemAttributeSet, interactive snippets (SnippetIntent), AppShortcutsProvider, multiple choice API (requestChoice), Swift Package support (AppIntentsPackage), Visual Intelligence integration (@UnionValue, IntentValueQuery, SemanticContentDescriptor). Use when adding Siri support, Shortcuts actions, Spotlight indexing, or system-wide app entity exposure.
---

# AppIntents — Siri, Shortcuts & Spotlight

## Critical Constraints

- ❌ DO NOT use old SiriKit `INIntent` → ✅ Use `AppIntent` protocol
- ❌ DO NOT use `NSUserActivity` alone for Spotlight → ✅ Use `IndexedEntity` + `CSSearchableIndex.default().indexAppEntities()`
- ❌ DO NOT hardcode foreground-only intents → ✅ Use `supportedModes` for flexible execution

## Basic App Intent
```swift
import AppIntents

struct FindNearestLandmarkIntent: AppIntent {
    static var title: LocalizedStringResource = "Find Nearest Landmark"

    @Parameter(title: "Category")
    var category: String?

    func perform() async throws -> some IntentResult {
        let landmark = await findNearestLandmark(category: category)
        return .result(value: landmark)
    }
}
```

## Intent Modes (Background/Foreground Control)
```swift
struct GetCrowdStatusIntent: AppIntent {
    static let supportedModes: IntentModes = [.background, .foreground(.dynamic)]

    func perform() async throws -> some ReturnsValue<Int> & ProvidesDialog {
        guard await modelData.isOpen(landmark) else {
            return .result(value: 0, dialog: "Currently closed.")
        }
        if systemContext.currentMode.canContinueInForeground {
            do {
                try await continueInForeground(alwaysConfirm: false)
                await navigator.navigateToCrowdStatus(landmark)
            } catch { }
        }
        let status = await modelData.getCrowdStatus(landmark)
        return .result(value: status, dialog: "Crowd level: \(status)")
    }
}
```

Mode combinations:
- `[.background, .foreground]` — foreground default, background fallback
- `[.background, .foreground(.dynamic)]` — background default, can request foreground
- `[.background, .foreground(.deferred)]` — background first, guaranteed foreground later

## Property Macros

```swift
struct LandmarkEntity: IndexedEntity {
    // Computed — reads from source of truth
    @ComputedProperty
    var isFavorite: Bool { UserDefaults.standard.favorites.contains(id) }

    // Deferred — expensive, fetched only when requested
    @DeferredProperty
    var crowdStatus: Int {
        get async throws { await modelData.getCrowdStatus(self) }
    }
}
```

## Spotlight Integration
```swift
struct LandmarkEntity: AppEntity, IndexedEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(
        name: "Landmark", systemImage: "mountain.2"
    )
    var id: String
    var name: String
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(name)", image: .init(systemName: "mountain.2"))
    }

    var searchableAttributes: CSSearchableItemAttributeSet {
        let attrs = CSSearchableItemAttributeSet()
        attrs.title = name
        return attrs
    }
}

// Index entities
try await CSSearchableIndex.default().indexAppEntities(landmarks, priority: .normal)

// Remove from index
try await CSSearchableIndex.default().deleteAppEntities(identifiedBy: [id], ofType: LandmarkEntity.self)
```

## Interactive Snippets
```swift
struct LandmarkSnippetIntent: SnippetIntent {
    @Parameter var landmark: LandmarkEntity

    var snippet: some View {
        VStack {
            Text(landmark.name).font(.headline)
            HStack {
                Button("Add to Favorites") { }
                Button("Search Tickets") { }
            }
        }.padding()
    }
}
```

## App Shortcuts
```swift
struct AppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: FindNearestLandmarkIntent(),
            phrases: ["Find the closest landmark with \(.applicationName)"],
            systemImageName: "location"
        )
    }
}
```

## Swift Package Support
```swift
// In framework
public struct LandmarksKitPackage: AppIntentsPackage { }

// In app target
struct LandmarksPackage: AppIntentsPackage {
    static var includedPackages: [any AppIntentsPackage.Type] {
        [LandmarksKitPackage.self]
    }
}
```

## References

- [App Intents Documentation](https://developer.apple.com/documentation/AppIntents)
- [WWDC 2025: Explore new advances in App Intents](https://developer.apple.com/videos/play/wwdc2025/275)
