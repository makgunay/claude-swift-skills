# Common Stack Patterns for Apple Platform Apps

Reusable decision trees and framework comparisons. Use these during validation to quickly assess choices.

## App Archetype → Recommended Stack

### Menu Bar Utility (e.g., clipboard manager, prompt tool, quick launcher)
```
Language:     Swift 6.2 (default MainActor isolation)
UI:           SwiftUI + AppKit bridge (NSPanel, NSStatusItem)
Persistence:  SwiftData (simple models) or UserDefaults + JSON files (very simple)
Sync:         iCloud via CloudKit (if needed)
Hotkeys:      Carbon RegisterEventHotKey + NSEvent monitors
Text insert:  NSPasteboard + CGEvent simulation
Distribution: Direct (full capability) or App Store (limited by sandbox)
Design:       Liquid Glass (macOS 26+)
```

### Document-Based App (e.g., text editor, note-taking, design tool)
```
Language:     Swift 6.2
UI:           SwiftUI (NavigationSplitView + detail)
Persistence:  SwiftData with class inheritance (complex models)
              or Core Data (if macOS 13 support needed)
Sync:         CloudKit + SwiftData auto-sync
Rich text:    TextEditor + AttributedString (macOS 26+)
              or TextKit 2 via AppKit bridge (older targets)
Distribution: App Store preferred (document types registration)
Design:       Standard SwiftUI + Liquid Glass accents
```

### Media / Content App (e.g., photo organizer, podcast player)
```
Language:     Swift 6.2
UI:           SwiftUI (grid layouts, media playback)
Persistence:  SwiftData (metadata) + file system (media files)
Processing:   @concurrent functions for heavy media work
ML/AI:        Core ML (image classification) or FoundationModels (text)
Sync:         CloudKit for metadata, iCloud Drive for files
Distribution: App Store
```

### Developer Tool (e.g., API client, code previewer, log viewer)
```
Language:     Swift 6.2
UI:           SwiftUI + AppKit bridge (NSTextView for code, custom views)
Persistence:  SQLite via GRDB (large datasets, complex queries)
              or SwiftData (simpler data needs)
Web:          Native WebView (macOS 26+) or WKWebView bridge
Network:      URLSession + Codable
Syntax:       tree-sitter or custom TextKit 2 highlighter
Distribution: Direct preferred (fewer sandbox limits)
```

### AI-Powered App (e.g., writing assistant, smart summarizer)
```
Language:     Swift 6.2
UI:           SwiftUI
On-device AI: FoundationModels (macOS 26+, Apple Silicon)
Cloud AI:     URLSession + Anthropic/OpenAI API (if on-device insufficient)
Persistence:  SwiftData (conversations, preferences)
Streaming:    AsyncSequence for token streaming
Distribution: App Store (on-device) or Direct (if cloud API keys needed)
```

## Framework Comparison Tables

### Persistence: SwiftData vs Core Data vs SQLite

| Aspect | SwiftData | Core Data | SQLite (GRDB) |
|--------|-----------|-----------|---------------|
| Min OS | macOS 14 / iOS 17 | macOS 10.4 / iOS 3 | Any |
| Swift-native | ✅ @Model macro | ❌ NSManagedObject | ⚠️ Swift wrapper |
| iCloud sync | ✅ Built-in | ✅ Built-in | ❌ Manual |
| Complex queries | ⚠️ #Predicate (limited) | ✅ NSPredicate (flexible) | ✅ Full SQL |
| Large datasets (>100K) | ⚠️ Untested at scale | ✅ Proven | ✅ Proven |
| Class inheritance | ✅ (macOS 15+) | ✅ | N/A (schema) |
| Migration | ✅ Lightweight auto | ✅ Mapping models | ✅ SQL migrations |
| Encryption | ❌ Not built-in | ❌ Not built-in | ✅ SQLCipher |
| Thread safety | ✅ ModelActor | ⚠️ Manual contexts | ✅ DatabasePool |
| Learning curve | Low | High | Medium |

**Decision:** SwiftData for new projects targeting macOS 14+. Core Data for older targets or proven scale needs. SQLite/GRDB for complex queries, encryption, or >100K records.

### UI: Pure SwiftUI vs SwiftUI + AppKit Bridge

| Aspect | Pure SwiftUI | SwiftUI + AppKit |
|--------|-------------|-----------------|
| Development speed | ✅ Faster | ⚠️ Slower (bridging code) |
| Maintenance | ✅ Simpler | ⚠️ Two paradigms |
| Floating panels | ❌ Not possible | ✅ NSPanel |
| Custom window levels | ❌ Not possible | ✅ NSWindow.level |
| Menu bar (advanced) | ⚠️ MenuBarExtra (limited) | ✅ NSStatusItem (full control) |
| Global hotkeys | ❌ Not possible | ✅ Carbon/NSEvent |
| System-wide text insert | ❌ Not possible | ✅ CGEvent + Pasteboard |
| Rich text (advanced) | ⚠️ TextEditor (basic) | ✅ NSTextView (full) |
| Custom drawing | ⚠️ Canvas (limited) | ✅ NSView.draw() |
| App Store review | ✅ Simpler | ✅ Fine |

**Decision:** Pure SwiftUI for standard apps. Add AppKit bridge ONLY for capabilities SwiftUI can't provide (panels, hotkeys, text insertion, custom windows).

### Sync: CloudKit vs Custom Server vs Third-Party

| Aspect | CloudKit | Custom Server | Firebase/Supabase |
|--------|----------|---------------|-------------------|
| Cost | ✅ Free (generous limits) | ❌ Server hosting | ⚠️ Free tier + paid |
| Setup complexity | ✅ Low | ❌ High | ⚠️ Medium |
| Apple-only | ✅ Native | ✅ Cross-platform | ✅ Cross-platform |
| Real-time | ⚠️ Subscriptions (delayed) | ✅ WebSockets | ✅ Built-in |
| Offline-first | ✅ Built-in | ❌ Manual | ⚠️ Depends |
| Conflict resolution | ✅ Automatic (last-write) | ❌ Manual | ⚠️ Depends |
| Privacy | ✅ User's iCloud | ❌ Your server | ⚠️ Third-party |
| Access control | ⚠️ Basic (zones, sharing) | ✅ Full | ✅ Row-level |
| Migration | ⚠️ Locked to Apple | ✅ Portable | ⚠️ Some lock-in |

**Decision:** CloudKit for Apple-only apps with simple sync. Custom server for cross-platform or complex access control. Third-party BaaS for fast prototyping with future migration planned.

## Red Flag Patterns

These combinations almost always cause problems:

| Pattern | Problem | Fix |
|---------|---------|-----|
| SwiftData + macOS 13 target | SwiftData requires macOS 14+ | Use Core Data or raise minimum OS |
| Liquid Glass + macOS 14 target | .glassEffect() requires macOS 26+ | Use NSVisualEffectView or raise minimum OS |
| App Store + CGEvent text insertion | Sandbox blocks CGEvent | Direct distribution, or use Accessibility entitlement |
| FoundationModels + Intel Mac support | Requires Apple Silicon + Apple Intelligence | Fallback to cloud API or drop Intel |
| Solo dev + custom sync server + 8 week timeline | Server development consumes the entire timeline | Use CloudKit |
| SwiftData @Query + full-text search on 50K+ items | @Query may be slow for complex text search | Add SQLite FTS index alongside SwiftData |
| @Observable + macOS 13 deployment | @Observable requires macOS 14+ | Use ObservableObject or raise minimum OS |
| Swift 6 strict concurrency + large existing codebase | Migration effort can be enormous | Enable incrementally per target |

## Third-Party Dependency Evaluation Checklist

Before adding any non-Apple dependency:

- [ ] **Maintained?** Last commit < 6 months ago, responsive to issues
- [ ] **License?** MIT/Apache for App Store, check GPL restrictions
- [ ] **Size?** Binary size impact acceptable
- [ ] **Swift version?** Compatible with project's Swift version
- [ ] **Privacy?** No tracking, no unexpected network calls
- [ ] **Alternatives?** Is there an Apple framework that does 80% of this?
- [ ] **Exit plan?** How hard is it to replace if abandoned?
