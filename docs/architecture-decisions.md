# Swift/macOS Agentic Coding Skills — Architecture v2

## Purpose

Bridge the gap between LLM training data (thin on Swift/macOS, absent for WWDC 2025 APIs) and the verified, working patterns needed for reliable code generation. Each skill injects constraints, patterns, and decision logic at the moment of code generation.

## Design Principles

1. **Constraints > Patterns** — Telling the agent what NOT to do prevents more errors than showing more examples
2. **Self-contained code** — Every code block includes imports and compiles independently
3. **Version-tagged** — Every pattern tagged with minimum OS version
4. **Progressive disclosure** — SKILL.md ≤500 lines; overflow goes to `references/`
5. **Living system** — The `skill-maintainer` meta-skill keeps everything current

---

## Skill Inventory (15 core + 3 optional)

### Tier 1 — Language & Framework Core (always relevant)

| # | Skill | Coverage | Primary Source Docs |
|---|---|---|---|
| 1 | `swift-lang` | Swift 6.2 concurrency (default MainActor, `@concurrent`, isolated conformances), `@Observable`, `InlineArray`, `Span`, modern error handling, macros | Swift-Concurrency-Updates.md, Swift-InlineArray-Span.md |
| 2 | `swiftui-core` | Views, navigation, state management, new toolbar system (customizable IDs, `ToolbarSpacer`, `DefaultToolbarItem`), styled text editing (`TextEditor` + `AttributedString`), modifiers, layout | SwiftUI-New-Toolbar-Features.md, SwiftUI-Styled-Text-Editing.md, Foundation-AttributedString-Updates.md |
| 3 | `swiftdata` | `@Model`, `@Query`, relationships, class inheritance, type-based predicates, `ModelContainer`/`ModelContext`, migrations | SwiftData-Class-Inheritance.md |
| 4 | `macos-app-structure` | App lifecycle, `WindowGroup`, `Window`, `Settings`, `MenuBarExtra`, multi-window, entitlements, sandboxing, Info.plist | Synthesized from project patterns + existing macos-tahoe-dev skill |

### Tier 2 — System Integration (loaded per-task)

| # | Skill | Coverage | Primary Source Docs |
|---|---|---|---|
| 5 | `appkit-bridge` | `NSViewRepresentable`, `NSPanel`, `NSWindow`, `NSEvent` monitoring, responder chain, `NSGlassEffectView`, `NSAnimationContext` | AppKit-Implementing-Liquid-Glass-Design.md |
| 6 | `global-hotkeys` | `CGEvent` taps, `NSEvent.addGlobalMonitorForEvents`, `AXIsProcessTrusted()`, key code mapping | Existing Linea Prompt code |
| 7 | `pasteboard-textinsertion` | `NSPasteboard` read/write, CGEvent keystroke simulation, Accessibility API insertion, clipboard preserve/restore | Existing Linea Prompt code |
| 8 | `macos-permissions` | Accessibility, screen recording, full disk access, TCC database, graceful degradation | Synthesized from project patterns |
| 9 | `swiftui-webkit` | `WebView` struct, `WebPage` observable, navigation management, `callJavaScript`, custom URL schemes, snapshots, PDF generation | SwiftUI-WebKit-Integration.md |
| 10 | `foundation-models` | `SystemLanguageModel`, `LanguageModelSession`, instructions/prompts, `@Generable` + `@Guide`, snapshot streaming, tool calling, `response.content` (not output) | FoundationModels-Using-on-device-LLM-in-your-app.md |

### Tier 3 — UI & Distribution

| # | Skill | Coverage | Primary Source Docs |
|---|---|---|---|
| 11 | `liquid-glass` | SwiftUI `.glassEffect()`, `GlassEffectContainer`, morphing, AppKit `NSGlassEffectView`, UIKit `UIGlassEffect`, WidgetKit accented mode, button styles | SwiftUI-Implementing-Liquid-Glass-Design.md, AppKit-Implementing-Liquid-Glass-Design.md, UIKit-Implementing-Liquid-Glass-Design.md, WidgetKit-Implementing-Liquid-Glass-Design.md |
| 12 | `macos-distribution` | Code signing, notarization, DMG/pkg, PrivacyInfo.xcprivacy, StoreKit (`SubscriptionOfferView`, `appTransactionID`), App Store workflow | StoreKit-Updates.md |
| 13 | `testing-swift` | Swift Testing (`@Test`, `#expect`, `@Suite`), parameterized tests, testing `@Observable`, testing async code | Needs research/synthesis |

### Tier 4 — Specialized (on-demand)

| # | Skill | Coverage | Primary Source Docs |
|---|---|---|---|
| 14 | `app-intents` | `AppIntent`, `OpenIntent`, intent modes, `@ComputedProperty`, `@DeferredProperty`, `IndexedEntity`, Spotlight, interactive snippets | AppIntents-Updates.md |

### Meta

| # | Skill | Coverage |
|---|---|---|
| 15 | `skill-maintainer` | Ingests new Apple docs, classifies by framework, extracts patterns/constraints, diffs against existing skills, updates or creates skills, reports changes |

### Optional (build if needed)

| Skill | Source Doc | When Needed |
|---|---|---|
| `mapkit-geo` | MapKit-GeoToolbox-PlaceDescriptors.md | Location-based apps |
| `charts-3d` | Swift-Charts-3D-Visualization.md | Data visualization apps |
| `assistive-access` | Implementing-Assistive-Access-in-iOS.md | iOS accessibility apps |

---

## Build Priority

Ordered by impact on agent code quality × readiness of source material:

| Priority | Skill | Why First | Source Status |
|---|---|---|---|
| **1** | `swift-lang` | Fixes ~40% of agent errors; Swift 6.2 concurrency is a paradigm shift unknown to LLMs | ✅ Have 2 docs |
| **2** | `liquid-glass` | Apple's entire new design language; 4 comprehensive source docs | ✅ Have 4 docs |
| **3** | `swiftui-core` | Toolbar + text editing are daily-use APIs with breaking changes | ✅ Have 3 docs |
| **4** | `swiftdata` | Inheritance is new; LLMs constantly hallucinate SwiftData APIs | ✅ Have 1 doc |
| **5** | `foundation-models` | Brand new framework with zero LLM coverage | ✅ Have 1 doc |
| **6** | `swiftui-webkit` | Brand new API; agents will generate deprecated WKWebView patterns | ✅ Have 1 doc |
| **7** | `skill-maintainer` | Enables sustainable maintenance of all other skills | ✅ Created |
| **8** | `app-intents` | Siri/Shortcuts integration with new intent modes | ✅ Have 1 doc |
| **9** | `macos-app-structure` | Needs project patterns; less doc-dependent | 🟡 Synthesize |
| **10** | `appkit-bridge` | Partially covered by Liquid Glass AppKit doc | 🟡 Partial |
| **11** | `macos-distribution` | StoreKit updates available; rest needs synthesis | 🟡 Partial |
| **12+** | Rest | Build as encountered | Mixed |

---

## Internal Skill Template

Every skill follows this structure:

```markdown
---
name: [skill-name]
description: [Comprehensive trigger description — this is what determines when the skill loads]
---

# [Skill Title]

[One-line purpose statement]

## Reference Index (if using reference files)
| File | When to Use |
|------|-------------|

## Critical Constraints
- ❌ DO NOT [common LLM mistake] → ✅ INSTEAD [correct pattern]
- ❌ DO NOT [deprecated API] → ✅ INSTEAD [replacement] (since [OS version])

## Decision Tree
- If [condition A] → use [approach X]
- If [condition B] → use [approach Y]
- If [condition C] → see [reference file]

## Verified Patterns

### Pattern: [Name]
<!-- Verified: macOS XX, Swift X.X, Xcode XX -->
<!-- Source: [document name] -->
\```swift
import Framework
// Complete, self-contained, compilable code
\```

## Common Mistakes & Fixes
| Mistake | Fix |
|---------|-----|
| [What agents do wrong] | [Correct approach] |

## References
- [Apple Doc links]
- [WWDC session links with year]
```

---

## Sizing Guidelines

| Component | Target |
|---|---|
| SKILL.md body | 200–500 lines |
| Each reference file | ≤800 lines |
| Each code block | Self-contained, includes imports |
| Constraints section | 5–15 rules |
| Patterns section | 3–10 patterns |
| Total skill footprint | SKILL.md + 0–10 reference files |

---

## Maintenance Workflow

```
New Apple Doc Uploaded
        │
        ▼
  skill-maintainer
        │
        ├─ Classify → which skill(s)?
        ├─ Extract → patterns, constraints, decisions
        ├─ Diff → what's new vs existing?
        ├─ Update → patch target skill(s)
        ├─ Create → new skill if unmapped framework
        └─ Report → summary of all changes
```

Trigger the `skill-maintainer` skill whenever:
- User uploads new Apple documentation (markdown, transcripts, release notes)
- User says "update skills", "ingest this", "add this to skills"
- A new Xcode beta ships (audit existing skills for staleness)
- User notices agent generating deprecated patterns (indicates skill gap)

---

## Relationship to Existing Skills

The `macos-tahoe-dev` skill already exists at `/mnt/skills/user/macos-tahoe-dev/`. The new skill set **extends and decomposes** it:

| macos-tahoe-dev coverage | New skill(s) |
|---|---|
| Liquid Glass quick reference | `liquid-glass` (comprehensive, 4 source docs) |
| Hotkey + panel + insertion | `global-hotkeys` + `pasteboard-textinsertion` |
| SwiftData persistence | `swiftdata` (with inheritance) |
| App Store sandboxing | `macos-distribution` |
| Accessibility | `macos-permissions` |
| Project structure patterns | `macos-app-structure` |

The existing `macos-tahoe-dev` remains useful as a **quick-start composite** for prompt manager / utility app projects. The new skills provide deeper, framework-specific guidance that applies to any macOS app.

---

## Source Document Inventory

Documents available for skill creation (uploaded by user):

| Document | Status | Target Skill(s) |
|---|---|---|
| Swift-Concurrency-Updates.md | 🔵 Ready | swift-lang |
| Swift-InlineArray-Span.md | 🔵 Ready | swift-lang |
| SwiftUI-Implementing-Liquid-Glass-Design.md | 🔵 Ready | liquid-glass |
| AppKit-Implementing-Liquid-Glass-Design.md | 🔵 Ready | liquid-glass, appkit-bridge |
| UIKit-Implementing-Liquid-Glass-Design.md | 🔵 Ready | liquid-glass (reference) |
| WidgetKit-Implementing-Liquid-Glass-Design.md | 🔵 Ready | liquid-glass |
| SwiftUI-New-Toolbar-Features.md | 🔵 Ready | swiftui-core |
| SwiftUI-Styled-Text-Editing.md | 🔵 Ready | swiftui-core |
| Foundation-AttributedString-Updates.md | 🔵 Ready | swiftui-core |
| SwiftData-Class-Inheritance.md | 🔵 Ready | swiftdata |
| SwiftUI-WebKit-Integration.md | 🔵 Ready | swiftui-webkit |
| FoundationModels-Using-on-device-LLM-in-your-app.md | 🔵 Ready | foundation-models |
| AppIntents-Updates.md | 🔵 Ready | app-intents |
| StoreKit-Updates.md | 🔵 Ready | macos-distribution |
| SwiftUI-AlarmKit-Integration.md | 🟡 Not reviewed | TBD |
| Implementing-Assistive-Access-in-iOS.md | ⚪ Optional | assistive-access |
| Implementing-Visual-Intelligence-in-iOS.md | ⚪ Optional | Skip (iOS-only) |
| MapKit-GeoToolbox-PlaceDescriptors.md | ⚪ Optional | mapkit-geo |
| Swift-Charts-3D-Visualization.md | ⚪ Optional | charts-3d |
