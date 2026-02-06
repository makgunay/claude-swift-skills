# Architecture Document Template

Use this template for the technical architecture deliverable. Adapt to project complexity. For a SwiftUI macOS app, sections on microservices or API gateway are irrelevant — focus on what matters for native app development.

## Template

```markdown
# Technical Architecture: [App Name]

**Version:** [x.y]
**Last Updated:** [Date]
**PRD Reference:** [Link or version of corresponding PRD]

---

## 1. Architecture Overview

### Design Philosophy
[2-3 sentences on the guiding principles: e.g., "Offline-first, single-threaded by default, thin views with observable models"]

### High-Level Component Map
[Describe the major components and their relationships. Use a text diagram:]

```
┌─────────────────────────────────────────────┐
│                   App Layer                  │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │  Views    │  │ViewModels│  │  Services  │ │
│  │(SwiftUI) │──│(@Observable)│──│(Business) │ │
│  └──────────┘  └──────────┘  └───────────┘ │
├─────────────────────────────────────────────┤
│                 Data Layer                   │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │SwiftData │  │ Keychain │  │  iCloud    │ │
│  │ (Local)  │  │(Secrets) │  │  (Sync)   │ │
│  └──────────┘  └──────────┘  └───────────┘ │
├─────────────────────────────────────────────┤
│               System Integration             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │Hotkeys   │  │Pasteboard│  │Permissions │ │
│  │(Carbon)  │  │(AppKit)  │  │   (TCC)   │ │
│  └──────────┘  └──────────┘  └───────────┘ │
└─────────────────────────────────────────────┘
```

## 2. Technology Stack

| Layer | Choice | Justification |
|-------|--------|---------------|
| Language | Swift 6.2 | Default MainActor isolation, modern concurrency |
| UI | SwiftUI | Native, declarative, Liquid Glass support |
| Persistence | SwiftData | Native Swift ORM, iCloud sync built-in |
| Design | Liquid Glass | Apple's current design language (macOS 26+) |
| Distribution | [App Store / Direct / Both] | [Reason] |
| Minimum OS | [macOS XX / iOS XX] | [Reason — feature dependency] |

### Framework Dependencies
| Framework | Purpose | Availability |
|-----------|---------|-------------|
| [Framework] | [Why] | [OS version] |

## 3. System Components

### Component: [Name]
- **Responsibility:** [Single responsibility description]
- **Owns:** [What data/state does it own?]
- **Dependencies:** [What does it depend on?]
- **Interface:** [Key public methods/properties]

[Repeat for each major component]

## 4. Data Layer

### Data Models

#### [Entity Name]
```swift
@Model
class [Entity] {
    var id: UUID
    // properties...

    @Relationship(deleteRule: .cascade)
    var children: [ChildEntity]
}
```

### Persistence Strategy
- **Local storage:** SwiftData with [SQLite/in-memory] backing
- **Sync:** [iCloud via CloudKit / None / Custom]
- **Conflict resolution:** [Last-write-wins / Merge / Manual]
- **Migration strategy:** [Lightweight / Staged / None for v1]

### Data Flow
```
User Action → View → ViewModel → Service → SwiftData → iCloud
                                    ↓
                              Local Cache
```

## 5. Concurrency Model

### Actor Isolation Strategy
- **Default:** MainActor (Swift 6.2 default isolation)
- **Background work:** `@concurrent` functions for heavy processing
- **Data access:** Through SwiftData ModelContext (MainActor)

### Threading Rules
1. All UI code runs on MainActor (default)
2. File I/O and parsing use `@concurrent` functions
3. Network requests use structured concurrency (`async let`, `TaskGroup`)
4. Never block MainActor with synchronous I/O

## 6. UI Architecture

### Navigation Structure
[Describe the navigation model]

### State Management
| State Type | Mechanism | Scope |
|-----------|-----------|-------|
| View-local | `@State` | Single view |
| Shared model | `@Observable` + `@State` | View tree |
| App-wide | `@Environment` | Entire app |
| Persistent | SwiftData `@Query` | Disk-backed |

### View Hierarchy
```
App
├── WindowGroup: Main Window
│   └── NavigationSplitView
│       ├── Sidebar: [List/Categories]
│       └── Detail: [Content Area]
├── Window: [Secondary Window] (if any)
├── MenuBarExtra: [Menu Bar] (if any)
└── Settings: Preferences
```

## 7. Integration Points

### System Services
| Service | Purpose | Permission Required |
|---------|---------|-------------------|
| [Service] | [Why] | [Permission type] |

### Permission Flow
```
First Launch → Check permissions → Request if needed → Degrade gracefully if denied
```

## 8. Security & Privacy

### Data at Rest
- [Encryption approach for sensitive data]
- [Keychain usage for secrets/tokens]
- [SwiftData encryption options]

### Data in Transit
- [Network security: HTTPS, certificate pinning]
- [iCloud encryption]

### Privacy Manifest
- Required API declarations for PrivacyInfo.xcprivacy
- No tracking, no third-party analytics (or specify what's used)

## 9. Performance Budget

| Metric | Budget | Measurement Method |
|--------|--------|-------------------|
| Cold launch to interactive | < [X]ms | Instruments: App Launch |
| Search latency ([N] items) | < [X]ms | Instruments: Time Profiler |
| Idle memory | < [X]MB | Instruments: Allocations |
| Storage per [unit of content] | ~[X]KB | Disk measurement |
| Animation frame rate | 60fps (120fps on ProMotion) | Instruments: Core Animation |

## 10. Testing Strategy

### Test Pyramid
| Level | Framework | Coverage Target | Focus |
|-------|-----------|----------------|-------|
| Unit | Swift Testing | [X]% | Models, ViewModels, Services |
| Integration | Swift Testing | Key flows | Data layer, sync |
| UI | XCUITest | Critical paths | Navigation, core workflows |

### Key Test Scenarios
1. [Scenario 1: What's tested and why it matters]
2. [Scenario 2]

## 11. Build & Distribution

### Build Pipeline
- Xcode [version] + Swift [version]
- Code signing: [Developer ID / Apple Distribution]
- Notarization: [If direct distribution]

### Release Strategy
- [App Store review process]
- [Beta testing via TestFlight]
- [Versioning scheme: SemVer]

## 12. Migration & Upgrade Path

### v1.0 → v1.1
- [Data model changes, if any]
- [SwiftData migration approach]

### Future Considerations
- [Design decisions made now that enable future expansion]
- [What would need to change for iOS port]
- [What would need to change for team/collaboration features]
```

## Architecture Decision Records (ADRs)

For significant technical decisions, include brief ADRs:

```markdown
### ADR-001: [Decision Title]
- **Status:** Accepted
- **Context:** [Why this decision needed to be made]
- **Decision:** [What was decided]
- **Alternatives Considered:** [What else was evaluated]
- **Consequences:** [Trade-offs accepted]
```
