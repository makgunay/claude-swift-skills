# AttributedString API Reference

## Core Operations
```swift
var text = AttributedString("Styled text")
text.foregroundColor = .red
text.backgroundColor = .yellow
text.font = .systemFont(ofSize: 14)

// Range-based styling
let range = text.range(of: "Styled")!
text[range].underlineStyle = .single
text[range].underlineColor = .blue
```

## Text Alignment
```swift
var paragraph = AttributedString("Centered text")
paragraph.alignment = .center  // .left, .right, .center
```

## Writing Direction
```swift
var text = AttributedString("Hello عربي")
text.writingDirection = .rightToLeft  // .leftToRight, .rightToLeft
```

## Line Height
```swift
var text = AttributedString("Multi-line text")
text.lineHeight = .exact(points: 32)
text.lineHeight = .multiple(factor: 2.5)
text.lineHeight = .loose
```

## DiscontiguousAttributedSubstring
```swift
let text = AttributedString("Select multiple parts of this text")
let range1 = text.range(of: "Select")!
let range2 = text.range(of: "text")!
let rangeSet = RangeSet([range1, range2])
var substring = text[rangeSet]
substring.backgroundColor = .yellow
let combined = AttributedString(substring)
```

## UTF-8 View
```swift
let text = AttributedString("Hello")
for codeUnit in text.utf8 { print(codeUnit) }
```
