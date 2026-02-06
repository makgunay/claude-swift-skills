# SwiftUI Toolbar System (Updated)

## Customizable Toolbars

Users can personalize by adding, removing, rearranging items.

```swift
ContentView()
    .toolbar(id: "main-toolbar") {
        ToolbarItem(id: "tag") { TagButton() }
        ToolbarItem(id: "share") { ShareButton() }
        ToolbarSpacer(.fixed)
        ToolbarItem(id: "more") { MoreButton() }
    }
```

Every item in a customizable toolbar needs its own `id`.

## Toolbar Spacers

```swift
ToolbarSpacer(.fixed)      // Fixed-width space
ToolbarSpacer(.flexible)   // Flexible space (pushes items apart)
```

Users can add multiple spacers from customization panel.

## Search Integration

### Minimize Behavior
```swift
NavigationStack {
    RecipeList()
        .searchable($searchText)
        .searchToolbarBehavior(.minimize)  // Renders as button, expands on tap
}
```

### Repositioning Search
```swift
NavigationSplitView {
    Sidebar()
} detail: {
    DetailView()
        .searchable($query)
        .toolbar {
            ToolbarItem(placement: .bottomBar) { CalendarPicker() }
            DefaultToolbarItem(kind: .search, placement: .bottomBar)
            ToolbarSpacer(placement: .bottomBar)
            ToolbarItem(placement: .bottomBar) { NewEventButton() }
        }
}
```

## Large Subtitle Placement
```swift
NavigationStack {
    DetailView()
        .navigationTitle("Title")
        .navigationSubtitle("Subtitle")
        .toolbar {
            ToolbarItem(placement: .largeSubtitle) {
                CustomLargeNavigationSubtitle()
            }
        }
}
```

## Matched Transition Source

Smooth transitions between toolbar items and sheets/views.

```swift
struct ContentView: View {
    @State private var isPresented = false
    @Namespace private var namespace

    var body: some View {
        NavigationStack {
            DetailView()
                .toolbar {
                    ToolbarItem(placement: .topBarTrailing) {
                        Button("Show Sheet", systemImage: "globe") {
                            isPresented = true
                        }
                    }
                    .matchedTransitionSource(id: "world", in: namespace)
                }
                .sheet(isPresented: $isPresented) {
                    SheetView()
                        .navigationTransition(.zoom(sourceID: "world", in: namespace))
                }
        }
    }
}
```

## Shared Background Visibility

Control glass background on toolbar items.

```swift
.toolbar(id: "main") {
    ToolbarItem(id: "build-status", placement: .principal) {
        BuildStatus()
    }
    .sharedBackgroundVisibility(.hidden)  // Item stands out from shared glass
}
```

## System-Defined Toolbar Items
```swift
.toolbar {
    DefaultToolbarItem(kind: .search, placement: .bottomBar)
    DefaultToolbarItem(kind: .sidebar, placement: .navigationBarLeading)
}
```
