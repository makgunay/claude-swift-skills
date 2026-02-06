# SwiftUI Styled Text Editing

## TextEditor with AttributedString

```swift
struct RichTextEditorView: View {
    @State private var text = AttributedString("Editable styled text...")
    var body: some View {
        TextEditor(text: $text)
            .frame(minHeight: 100)
    }
}
```

## TextEditor with Selection

```swift
struct EditorWithSelection: View {
    @State private var text = AttributedString("Select text to format")
    @State private var selection = AttributedTextSelection()

    var body: some View {
        TextEditor(text: $text, selection: $selection)
    }
}
```

## Selection-Based Suggestions

```swift
struct SuggestionEditor: View {
    @State var text: AttributedString = ""
    @State var selection = AttributedTextSelection()

    var body: some View {
        VStack {
            TextEditor(text: $text, selection: $selection)
            SuggestionsView(substrings: getSubstrings(
                text: text, indices: selection.indices(in: text)))
        }
    }

    private func getSubstrings(
        text: String, indices: AttributedTextSelection.Indices
    ) -> [Substring] {
        // Resolve substrings for current selection
    }
}
```

## Selection Affinity
```swift
VStack {
    TextEditor(text: $text, selection: $selection)
}
.textSelectionAffinity(.upstream)
```

## Formatting Toolbar with Selection

```swift
struct FormattingEditor: View {
    @State private var text = AttributedString("Format me")
    @State private var selection = AttributedTextSelection()
    @Environment(\.fontResolutionContext) private var fontResolutionContext

    var body: some View {
        VStack {
            TextEditor(text: $text, selection: $selection)
            HStack {
                Button { toggleBold() } label: { Image(systemName: "bold") }
                Button { toggleItalic() } label: { Image(systemName: "italic") }
                Button { toggleUnderline() } label: { Image(systemName: "underline") }
            }
        }
    }

    private func toggleBold() {
        text.transformAttributes(in: &selection) {
            let font = $0.font ?? .default
            let resolved = font.resolve(in: fontResolutionContext)
            $0.font = font.bold(!resolved.isBold)
        }
    }

    private func toggleItalic() {
        text.transformAttributes(in: &selection) {
            let font = $0.font ?? .default
            let resolved = font.resolve(in: fontResolutionContext)
            $0.font = font.italic(!resolved.isItalic)
        }
    }

    private func toggleUnderline() {
        text.transformAttributes(in: &selection) {
            $0.underlineStyle = ($0.underlineStyle != nil) ? nil : .single
        }
    }
}
```

## Replace Selection Content
```swift
var text = AttributedString("Here is my dog")
var selection = AttributedTextSelection(range: text.range(of: "dog")!)

// Replace with plain text
text.replaceSelection(&selection, withCharacters: "cat")

// Replace with attributed text
let replacement = AttributedString("horse")
text.replaceSelection(&selection, with: replacement)
```

## Custom Formatting Definition
```swift
struct MyTextFormatting: AttributedTextFormattingDefinition {
    typealias Scope = AttributeScopes.SwiftUIAttributes
    // Define allowed formatting constraints
}

TextEditor(text: $text, selection: $selection)
    .textFormattingDefinition(MyTextFormatting.self)
```

## Markdown in Text Views
```swift
// SwiftUI Text supports inline Markdown
Text("This is **bold** and *italic* text")
Text("Visit [Apple](https://www.apple.com)")

// Limitations: no lists, block quotes, code blocks, tables, or images
```

## Text Styling Reference

```swift
// Font styles
Text("Hello").font(.largeTitle)
Text("Hello").font(.system(size: 24, design: .rounded))
Text("Hello").fontWeight(.bold)

// Colors
Text("Hello").foregroundStyle(.red)
Text("Hello").foregroundStyle(.linearGradient(colors: [.yellow, .blue], startPoint: .top, endPoint: .bottom))

// Decoration
Text("Hello").underline(true, pattern: .dash, color: .blue)
Text("Hello").strikethrough(true, pattern: .dot, color: .green)
Text("Hello").bold(someCondition)  // Conditional
```
