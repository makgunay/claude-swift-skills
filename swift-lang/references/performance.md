# Swift 6.2 Performance Primitives: InlineArray & Span

## InlineArray

Fixed-size array with inline storage. No heap allocation, no reference counting, no copy-on-write.

### Declaration
```swift
@frozen struct InlineArray<let count: Int, Element> where Element: ~Copyable
```

### Initialization
```swift
let a: InlineArray<4, Int> = [1, 2, 4, 8]
let b: InlineArray<_, Int> = [1, 2, 4, 8]  // count inferred as 4
let c: InlineArray = [1, 2, 4, 8]          // Both inferred: InlineArray<4, Int>
```

### Usage
```swift
var array: InlineArray<3, Int> = [1, 2, 3]
array[0] = 4                // Modify in place
// array.append(4)          // ❌ Cannot resize
// let big: InlineArray<6, Int> = array  // ❌ Cannot assign different sizes

for element in array { print(element) }
for i in array.indices { array[i] += 1 }
```

### Memory Layout
```swift
MemoryLayout<InlineArray<3, UInt16>>.size       // 6 (2 bytes × 3)
MemoryLayout<InlineArray<3, UInt16>>.stride     // 6
MemoryLayout<InlineArray<3, UInt16>>.alignment  // 2 (same as UInt16)
MemoryLayout<InlineArray<0, UInt16>>.size       // 0
```

### Copy Behavior — NO copy-on-write
```swift
var original: InlineArray<3, Int> = [1, 2, 3]
var copy = original         // Eager copy on assignment
copy[0] = 99
// original[0] is still 1
// copy[0] is 99
```

### When to Use
- ✅ Performance-critical hot paths
- ✅ Fixed-size collections (e.g., RGBA channels, matrix rows)
- ✅ Embedded within other structs to avoid heap allocation
- ❌ Collections that grow or shrink
- ❌ Collections that are frequently copied (no CoW optimization)

## Span

Safe abstraction for accessing contiguous memory. Replaces unsafe pointers.

### Types
| Type | Access | Use Case |
|------|--------|----------|
| `Span<Element>` | Read-only elements | Processing array data |
| `MutableSpan<Element>` | Read-write elements | In-place modification |
| `RawSpan` | Read-only bytes | Binary parsing |
| `MutableRawSpan` | Read-write bytes | Binary writing |
| `UTF8Span` | UTF-8 text | String processing |

### Accessing Spans
```swift
let array = [1, 2, 3, 4]
let span = array.span           // Span<Int>

let data = Data([0, 1, 2, 3])
let dataSpan = data.span        // Span<UInt8>

var mutable = [1, 2, 3]
let mSpan = mutable.mutableSpan // MutableSpan<Int>
```

### Safety Constraints (compile-time enforced)
```swift
// ❌ Cannot escape scope
func getSpan() -> Span<UInt8> {
    let array: [UInt8] = Array(repeating: 0, count: 128)
    return array.span  // ERROR: depends on local variable
}

// ❌ Cannot capture in closures
let span = array.span
let closure = { span.count }  // ERROR: cannot capture

// ❌ Cannot use after mutation
var array = [1, 2, 3]
let span = array.span
array.append(4)
// span[0]  // ERROR: array was mutated
```

### Processing with Span
```swift
func sum(_ array: [Int]) -> Int {
    let span = array.span
    var result = 0
    for i in 0..<span.count {
        result += span[i]
    }
    return result
}
```

### RawSpan byte reading
```swift
extension RawSpan {
    mutating func readByte() -> UInt8? {
        guard !isEmpty else { return nil }
        let value = unsafeLoadUnaligned(as: UInt8.self)
        self = self._extracting(droppingFirst: 1)
        return value
    }
}
```

### When to Use
- ✅ High-performance algorithms on contiguous data
- ✅ Binary parsing without unsafe pointers
- ✅ Processing large arrays with minimal overhead
- ❌ Data that needs to outlive the source container
- ❌ Simple iteration (just use `for element in array`)
