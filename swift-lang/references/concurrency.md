# Swift 6.2 Concurrency Reference

## The Paradigm Shift

Swift 6.2 changes the default from "async functions hop to the background" to "async functions stay on the caller's actor." This eliminates most data-race safety errors without requiring code changes.

### Before Swift 6.2
```swift
class PhotoProcessor {
    func extractSticker(data: Data, with id: String?) async -> Sticker? { /* ... */ }
}

@MainActor
final class StickerModel {
    let photoProcessor = PhotoProcessor()

    func extractSticker(_ item: PhotosPickerItem) async throws -> Sticker? {
        guard let data = try await item.loadTransferable(type: Data.self) else { return nil }
        // ERROR in Swift 6.0/6.1: Sending 'self.photoProcessor' risks causing data races
        return await photoProcessor.extractSticker(data: data, with: item.itemIdentifier)
    }
}
```

### After Swift 6.2
```swift
// Same code — NO ERROR. The async function runs on the caller's actor (MainActor).
class PhotoProcessor {
    func extractSticker(data: Data, with id: String?) async -> Sticker? { /* ... */ }
}

@MainActor
final class StickerModel {
    let photoProcessor = PhotoProcessor()

    func extractSticker(_ item: PhotosPickerItem) async throws -> Sticker? {
        guard let data = try await item.loadTransferable(type: Data.self) else { return nil }
        return await photoProcessor.extractSticker(data: data, with: item.itemIdentifier)
    }
}
```

## @concurrent Attribute

Ensures a function always runs on the concurrent thread pool, freeing the actor.

```swift
class PhotoProcessor {
    var cachedStickers: [String: Sticker]

    func extractSticker(data: Data, with id: String) async -> Sticker {
        // Runs on caller's actor (e.g., MainActor) — safe to access cachedStickers
        if let sticker = cachedStickers[id] {
            return sticker
        }

        // Offload expensive work to background
        let sticker = await Self.extractSubject(from: data)
        cachedStickers[id] = sticker
        return sticker
    }

    // Runs on concurrent thread pool — never blocks the actor
    @concurrent
    static func extractSubject(from data: Data) async -> Sticker {
        // Heavy image processing
    }
}
```

### Rules for @concurrent
- Must be on a `nonisolated` function (or static function, or function on nonisolated type)
- Function must be `async`
- Parameters must be `Sendable`
- Callers must `await` the result

### Typical pattern: nonisolated struct + @concurrent
```swift
nonisolated struct ImageProcessor {
    @concurrent
    func resize(image: Data, to size: CGSize) async -> Data? {
        // Runs in background automatically
    }

    @concurrent
    func compress(data: Data, quality: Double) async -> Data {
        // Also runs in background
    }
}

// Usage from @MainActor code
let processor = ImageProcessor()
let resized = await processor.resize(image: rawData, to: CGSize(width: 200, height: 200))
```

## Isolated Conformances

Allow @MainActor types to conform to non-isolated protocols safely.

```swift
protocol Exportable {
    func export()
}

// The @MainActor annotation on the conformance means:
// - export() can access MainActor state
// - Compiler ensures this conformance is only used on the MainActor
extension StickerModel: @MainActor Exportable {
    func export() {
        photoProcessor.exportAsPNG()  // MainActor state — OK
    }
}

// ✅ Works — ImageExporter is @MainActor
@MainActor
struct ImageExporter {
    var items: [any Exportable]
    mutating func add(_ item: StickerModel) {
        items.append(item)
    }
}

// ❌ Compile error — can't use MainActor conformance off the MainActor
nonisolated struct GenericExporter {
    var items: [any Exportable]
    mutating func add(_ item: StickerModel) {
        items.append(item)  // ERROR: Main actor-isolated conformance cannot be used in nonisolated context
    }
}
```

## Default MainActor Isolation Mode

Opt-in mode that infers `@MainActor` on everything in the target.

### What it eliminates
- Data-race errors about unsafe global/static variables
- Errors calling SDK functions that are MainActor-isolated
- Need to annotate every class/struct with `@MainActor`

### What stays the same
- You still use `nonisolated` to opt specific types/functions OUT
- You still use `@concurrent` for background work
- Actor isolation rules still apply at boundaries

### Enable in Xcode
Build Settings → Swift Compiler - Concurrency → Default Actor Isolation → `MainActor`

### Enable in Package.swift
```swift
.target(
    name: "MyApp",
    swiftSettings: [
        .defaultIsolation(MainActor.self)
    ]
)
```

### Recommended for
- Apps, scripts, executable targets
- Projects that are mostly single-threaded
- Projects migrating to Swift 6 concurrency

### Not recommended for
- Libraries consumed by other packages
- Frameworks that need explicit isolation control

## Global and Static Variables

```swift
// PROBLEM in Swift 6.0: Static property not concurrency-safe
final class StickerLibrary {
    static let shared: StickerLibrary = .init()  // Warning/error
}

// FIX 1: Annotate with @MainActor
@MainActor
final class StickerLibrary {
    static let shared: StickerLibrary = .init()  // Safe
}

// FIX 2: Use default MainActor isolation mode (entire target)
// Then no annotation needed — everything is @MainActor by default
final class StickerLibrary {
    static let shared: StickerLibrary = .init()  // Implicitly @MainActor
}

// FIX 3: For truly constant global state
final class Constants: Sendable {
    static let maxRetries = 3  // Immutable — Sendable is fine
}
```

## Structured Concurrency Patterns

### Task Groups
```swift
func processAllImages(_ images: [Data]) async -> [ProcessedImage] {
    await withTaskGroup(of: ProcessedImage?.self) { group in
        for image in images {
            group.addTask {
                await ImageProcessor.process(image)  // Runs concurrently
            }
        }
        var results: [ProcessedImage] = []
        for await result in group {
            if let result { results.append(result) }
        }
        return results
    }
}
```

### AsyncSequence
```swift
// Observe changes
for await status in AlarmManager.shared.alarmUpdates {
    self.alarms = status
}

// With cancellation
let task = Task {
    for await value in someAsyncSequence {
        process(value)
    }
}
// Later:
task.cancel()
```

### Error Handling with async
```swift
func loadData() async throws {
    do {
        let data = try await networkService.fetch()
        try await database.save(data)
    } catch is CancellationError {
        // Task was cancelled — clean up
    } catch {
        // Handle other errors
        throw error
    }
}
```

## Migration Checklist: Swift 5/6.0 → 6.2

1. Enable default MainActor isolation in build settings
2. Remove explicit `@MainActor` annotations (now inferred)
3. Replace `Task.detached { }` with `@concurrent` functions
4. Replace `DispatchQueue.global().async { }` with `@concurrent async` functions
5. Add `nonisolated` to types/functions that must run off MainActor
6. Use Xcode migration tooling: swift.org/migration
