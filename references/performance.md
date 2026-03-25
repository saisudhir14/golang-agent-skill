# Performance Reference

Extended performance optimization patterns for Go.

## String Operations

`strconv` avoids the reflection and formatting overhead of `fmt`. Run your own benchmarks to confirm, but `strconv.Itoa` is typically several times faster than `fmt.Sprintf` for integer-to-string conversion and allocates less.

```go
s := fmt.Sprintf("%d", n)  // slower: format parsing, interface boxing
s := strconv.Itoa(n)       // faster: direct conversion, fewer allocations
```

## Efficient Map Operations

```go
// Preallocate with known size
m := make(map[string]*User, len(users))
for _, u := range users {
    m[u.ID] = u
}

// Iterate with sorted keys when order matters
keys := slices.Sorted(maps.Keys(m))
for _, k := range keys {
    process(m[k])
}
```

## Memory-Efficient Slice Operations

```go
// Reuse slice memory
result = result[:0]
for _, v := range input {
    if predicate(v) {
        result = append(result, v)
    }
}

// Prevent memory leaks with large backing arrays
func trimSlice(s []Item) []Item {
    result := make([]Item, len(s))
    copy(result, s)
    return result
}
```

## strings.Builder vs Concatenation

String concatenation with `+=` allocates a new string on each append. `strings.Builder` writes to a growable buffer and allocates once.

```go
var b strings.Builder
b.Grow(estimatedSize)
for _, s := range parts {
    b.WriteString(s)
}
result := b.String()
```

## Struct Field Ordering for Memory Alignment

```go
// Wrong: 32 bytes due to padding
type Bad struct {
    a bool    // 1 byte + 7 padding
    b int64   // 8 bytes
    c bool    // 1 byte + 7 padding
    d int64   // 8 bytes
}

// Correct: 24 bytes, fields ordered by size
type Good struct {
    b int64   // 8 bytes
    d int64   // 8 bytes
    a bool    // 1 byte
    c bool    // 1 byte + 6 padding
}
```

## Sources

- [Go Blog - Profiling Go Programs](https://go.dev/blog/pprof)
- [Go Performance Wiki](https://go.dev/wiki/Performance)
- [Go 1.25 Release Notes](https://go.dev/doc/go1.25)
