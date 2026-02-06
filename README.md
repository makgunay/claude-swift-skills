# Swift & macOS Skills for Claude

22 skills that make Claude an expert at building native macOS and iOS apps with Swift 6.2, SwiftUI, Liquid Glass, and the latest Apple frameworks from WWDC 2025.

> **What are skills?** Structured knowledge files Claude reads before responding. They encode constraints, patterns, decision trees, and gotchas — turning Claude from "knows the syntax" into "knows how to ship."

## How It Works

Claude's training data includes Swift and Apple frameworks, but it's a snapshot — it doesn't know about WWDC 2025 APIs, has stale patterns, and misses platform-specific gotchas. Skills fix this by injecting current, structured knowledge at the moment Claude needs it.

**The mechanics:**

1. **You ask Claude something** — *"Build me a macOS menu bar app with a global hotkey"*

2. **Claude scans available skills** — it reads the `description` field in each skill's YAML frontmatter (a one-paragraph summary packed with trigger phrases and API names) and decides which skills are relevant

3. **Claude reads the matching `SKILL.md` files** — typically 2-4 skills per request, not all 22. For the example above, it would read `macos-app-structure`, `appkit-bridge`, `global-hotkeys`, and `liquid-glass`

4. **Claude responds with skill-informed knowledge** — it now knows that `NSEvent.addGlobalMonitorForEvents` doesn't fire when your own app is focused (you need a local monitor too), that `NSPanel` needs `.nonactivating` style mask for floating windows, and that Liquid Glass requires `GlassEffectContainer` when you have multiple glass views

**What skills contain:**

Each `SKILL.md` follows a consistent structure designed for how LLMs process information:

- **Critical Constraints** come first — the ❌ DO NOT / ✅ DO rules that prevent the most common mistakes. Claude is good at generating plausible code; it's bad at remembering edge cases. Constraints fix this.
- **Decision Trees** help Claude choose the right approach — "Need system-wide hotkey that works even when app isn't focused? → Carbon `RegisterEventHotKey`. Need in-app shortcuts? → SwiftUI `.onKeyPress`." This prevents Claude from defaulting to whatever pattern it saw most during training.
- **Code Patterns** are complete, tested implementations — not snippets. They include error handling, edge cases, and the surrounding context that Claude usually gets wrong.
- **Reference files** (in `references/` subdirectories) contain deeper material that Claude reads when it needs more detail on a specific subtopic.

**Token overhead:**

Skills cost context window space. Here's what typical usage looks like:

| Scenario | Skills loaded | Tokens used | % of 200K context |
|---|---|---|---|
| Quick question | 1-2 | ~3-5K | 1-2% |
| Build a feature | 3-5 | ~10-15K | 5-7% |
| Full app from scratch | 7-10 | ~20-30K | 10-15% |

The modular design is intentional — each skill is self-contained so Claude only loads what's relevant, leaving the vast majority of context for your actual conversation.

**Why not just put this in a system prompt?**

You could paste skill content into a system prompt, but skills-as-files have advantages: they're versioned in git, modular (use only what you need), updatable (swap in new WWDC content without touching your prompt), and shareable. The `skill-maintainer` meta-skill can even ingest new Apple documentation and update existing skills automatically.

## Installation

### Claude.ai Projects
1. Create or open a Project
2. Go to **Project Knowledge** → **Add content** → **Upload files**
3. Upload the skill folders you need (each folder is self-contained)

### Claude Code
```bash
# Clone into your skills directory
git clone https://github.com/makgunay/claude-swift-skills.git /path/to/skills/user/claude-swift-skills

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

## Platform Assumptions

**Swift 6.2 baseline.** Default MainActor isolation is a paradigm shift — async functions stay on the caller's actor unless explicitly marked `@concurrent`. The `swift-lang` skill covers this in depth.

**macOS 26+ primary target.** Liquid Glass, native WebView, and FoundationModels all require macOS 26 (Tahoe). Skills note OS availability where relevant; `tech-stack-validator` catches version mismatches automatically.

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
