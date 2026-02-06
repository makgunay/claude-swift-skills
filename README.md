# Swift & macOS Skills for Claude

22 skills that make Claude an expert at building native macOS and iOS apps with Swift 6.2, SwiftUI, Liquid Glass, and the latest Apple frameworks from WWDC 2025.

> **What are skills?** Structured knowledge files Claude reads before responding. They encode constraints, patterns, decision trees, and gotchas — turning Claude from "knows the syntax" into "knows how to ship."

## Installation

### Claude.ai Projects
1. Create or open a Project
2. Go to **Project Knowledge** → **Add content** → **Upload files**
3. Upload the skill folders you need (each folder is self-contained)

### Claude Code
```bash
# Clone into your skills directory
git clone https://github.com/yourname/claude-swift-skills.git /path/to/skills/user/claude-swift-skills

# Or symlink individual skills
ln -s /path/to/claude-swift-skills/liquid-glass ~/.claude/skills/liquid-glass
```

### API / System Prompt
Reference skill files in your system prompt or include their content directly. Each `SKILL.md` is designed to work standalone.

---

## Which Skills Do I Need?

**Building a macOS menu bar / utility app?**
→ `swift-lang` · `swiftui-core` · `macos-app-structure` · `appkit-bridge` · `global-hotkeys` · `liquid-glass` · `app-patterns`

**Building a macOS app with text insertion (snippets, expanders)?**
→ Above + `pasteboard-textinsertion` · `macos-permissions`

**Adding iOS support to a macOS app?**
→ `cross-platform` · `accessibility` · `app-intents`

**Starting from scratch with just an idea?**
→ `app-prd-architect` → `tech-stack-validator` → then implementation skills

**Adding on-device AI?**
→ `foundation-models` (requires macOS 26+ / iOS 26+)

**Shipping to the App Store?**
→ `macos-distribution` · `testing-swift`

---

## Skills

### 🏗️ Planning & Architecture

| Skill | What It Does |
|-------|-------------|
| [**app-prd-architect**](app-prd-architect/) | Interactive discovery → PRD → Architecture docs. Takes a rough idea through structured exploration, produces three deliverables. |
| [**tech-stack-validator**](tech-stack-validator/) | Validates tech choices against requirements. Checks OS availability, sandbox compatibility, deprecation risks. GO / STOP verdicts. |

### 🔤 Language & Framework Core

| Skill | What It Does |
|-------|-------------|
| [**swift-lang**](swift-lang/) | Swift 6.2 — default MainActor isolation, `@concurrent`, isolated conformances, `@Observable`, `InlineArray`, `Span`. |
| [**swiftui-core**](swiftui-core/) | NavigationSplitView/Stack, state management (`@State` + `@Observable` + `@Bindable`), toolbars, TextEditor, AttributedString. |
| [**swiftdata**](swiftdata/) | `@Model`, `@Query`, relationships, class inheritance, type predicates, ModelContainer/Context, CRUD. |
| [**testing-swift**](testing-swift/) | Swift Testing — `@Test`, `#expect`, `@Suite`, parameterized tests, confirmation API, XCTest migration. |

### 🖥️ macOS Architecture

| Skill | What It Does |
|-------|-------------|
| [**macos-app-structure**](macos-app-structure/) | App lifecycle, Scene types (WindowGroup, Settings, MenuBarExtra), NSApplicationDelegateAdaptor, Info.plist, entitlements. |
| [**appkit-bridge**](appkit-bridge/) | SwiftUI ↔ AppKit — NSViewRepresentable, NSHostingView, NSPanel floating windows, NSPopover, NSStatusItem menu bar. |
| [**global-hotkeys**](global-hotkeys/) | System-wide shortcuts — NSEvent monitoring, Carbon RegisterEventHotKey, modifier handling, key codes, `.onKeyPress`. |
| [**pasteboard-textinsertion**](pasteboard-textinsertion/) | Inserting text into other apps — clipboard + paste, CGEvent typing, Accessibility API, `{{variable}}` expansion. |
| [**macos-permissions**](macos-permissions/) | Accessibility (AXIsProcessTrusted), Screen Recording, TCC, graceful degradation, PrivacyInfo.xcprivacy. |

### 🎨 UI & Design

| Skill | What It Does |
|-------|-------------|
| [**liquid-glass**](liquid-glass/) | Liquid Glass — `.glassEffect()`, GlassEffectContainer, morphing, button styles, AppKit/UIKit/WidgetKit variants. |
| [**app-patterns**](app-patterns/) | Utility app patterns — Spotlight panel, sidebar layout, floating editor, grid palette, Settings form, toasts, `{{variable}}` system, rich copy. |
| [**swiftui-webkit**](swiftui-webkit/) | Native WebView in SwiftUI — WebPage navigation, async JavaScript, custom URL schemes, content capture. macOS 26+. |
| [**accessibility**](accessibility/) | Reduce Transparency/Motion fallbacks, VoiceOver labels/actions/rotors, Dynamic Type, keyboard navigation, color contrast. |

### 📱 Cross-Platform

| Skill | What It Does |
|-------|-------------|
| [**cross-platform**](cross-platform/) | macOS ↔ iOS — project structure, `#if os()` abstraction, iOS extensions (keyboard, widgets, share), App Groups, CloudKit sync, migration checklist. |

### 🤖 AI

| Skill | What It Does |
|-------|-------------|
| [**foundation-models**](foundation-models/) | On-device LLM — SystemLanguageModel, LanguageModelSession, `@Generable` structured output, `@Guide` constraints, streaming, Tools. |

### 📦 Distribution

| Skill | What It Does |
|-------|-------------|
| [**macos-distribution**](macos-distribution/) | Code signing, notarization, DMG, App Store, sandboxing, StoreKit subscriptions, launch at login. |
| [**app-intents**](app-intents/) | Siri, Shortcuts, Spotlight — AppIntent protocol, IndexedEntity, SnippetIntent, AppShortcutsProvider. |

### 🗺️ Domain-Specific (Optional)

| Skill | What It Does |
|-------|-------------|
| [**mapkit-geo**](mapkit-geo/) | MapKit + GeoToolbox — PlaceDescriptor, third-party place IDs, MKMapItem, SwiftUI Map annotations. |
| [**charts-3d**](charts-3d/) | Swift Charts 3D — Chart3D, SurfacePlot, Chart3DPose, surface styling, visionOS volumetric. |

### 🔧 Meta

| Skill | What It Does |
|-------|-------------|
| [**skill-maintainer**](skill-maintainer/) | Ingests new Apple docs (WWDC sessions, API diffs, migration guides) and updates existing skills. |

---

## How Skills Work Together

```
 ┌─────────────────────────────────────────────────────────┐
 │                    PLANNING LAYER                        │
 │                                                          │
 │   "I want to build a prompt manager"                     │
 │           │                                              │
 │           ▼                                              │
 │   app-prd-architect ──▶ tech-stack-validator             │
 │   (PRD + Architecture)    (GO / STOP verdict)            │
 └──────────────────────────────┬──────────────────────────┘
                                │
 ┌──────────────────────────────▼──────────────────────────┐
 │                  FOUNDATION LAYER                        │
 │                                                          │
 │   swift-lang          swiftui-core          swiftdata    │
 │   (Language rules)    (UI framework)      (Persistence)  │
 └──────┬──────────────────┬──────────────────┬────────────┘
        │                  │                  │
 ┌──────▼──────────────────▼──────────────────▼────────────┐
 │                SPECIALIZATION LAYER                      │
 │                                                          │
 │  ┌──────────────┐ ┌────────────┐ ┌────────────────────┐ │
 │  │ macOS System │ │  UI/Design │ │   Capabilities     │ │
 │  │              │ │            │ │                    │ │
 │  │ app-structure│ │ liquid-    │ │ foundation-models  │ │
 │  │ appkit-bridge│ │   glass    │ │ app-intents        │ │
 │  │ global-      │ │ app-       │ │ pasteboard-text    │ │
 │  │   hotkeys    │ │   patterns │ │   insertion        │ │
 │  │ macos-perms  │ │ swiftui-   │ │ cross-platform     │ │
 │  │              │ │   webkit   │ │ accessibility      │ │
 │  └──────────────┘ └────────────┘ └────────────────────┘ │
 └──────────────────────────┬──────────────────────────────┘
                            │
 ┌──────────────────────────▼──────────────────────────────┐
 │                   SHIPPING LAYER                         │
 │                                                          │
 │   testing-swift       macos-distribution                 │
 │   (Tests)             (Sign, notarize, submit)           │
 └─────────────────────────────────────────────────────────┘
```

**Example flow:** You say *"Build me a macOS prompt manager with a global hotkey."* Claude reads `app-prd-architect` to explore the idea, `tech-stack-validator` to verify the stack, then `swift-lang` + `swiftui-core` + `macos-app-structure` for the foundation, `global-hotkeys` + `pasteboard-textinsertion` + `app-patterns` for the core features, and `liquid-glass` for the UI.

---

## Skill Anatomy

```
skill-name/
├── SKILL.md              # Main file — Claude reads this first
│   ├── YAML frontmatter  # name + description (trigger matching)
│   ├── Critical Constraints  # ❌ / ✅ patterns (read first)
│   ├── Quick Reference   # Most-used code patterns
│   └── Decision Trees    # When to use what
└── references/           # Optional deep-dive documents
    ├── topic-a.md
    └── topic-b.md
```

The `description` in YAML frontmatter is how Claude decides whether to read a skill. It's written like a search index — packed with trigger phrases, API names, and use cases.

## Design Principles

**Constraints over examples.** Every skill leads with "Critical Constraints" — things that will break your code. Examples follow. Claude generates decent code by default; it needs help remembering the gotchas.

**Decision trees over opinions.** Need a floating panel? → NSPanel. Menu bar popover? → NSPopover. Standard window? → SwiftUI only. Skills match the right tool to the requirement rather than prescribing one approach.

**Swift 6.2 baseline.** Default MainActor isolation is a paradigm shift. Async functions stay on the caller's actor unless explicitly marked `@concurrent`. The `swift-lang` skill covers this in depth.

**macOS 26+ primary target.** Liquid Glass, native WebView, FoundationModels all require macOS 26 (Tahoe). Skills note OS availability where relevant; `tech-stack-validator` catches version mismatches automatically.

## Keeping Skills Current

Use `skill-maintainer` to ingest new Apple documentation:

1. Upload a WWDC session transcript, API diff, or migration guide
2. Tell Claude: *"Update the [skill-name] skill with this new documentation"*
3. It handles extraction, conflict resolution, and versioning

---

## Stats

22 skills · 35 files · 6,653 lines · 13 reference documents

## Source Material

Synthesized from WWDC 2025 session documentation covering Swift 6.2 concurrency, Liquid Glass, SwiftUI updates, SwiftData inheritance, FoundationModels, native WebKit, AppIntents, StoreKit, MapKit GeoToolbox, and Swift Charts 3D — plus macOS system programming patterns for hotkeys, panels, text insertion, clipboard management, accessibility, permissions, and app distribution.

## License

MIT
