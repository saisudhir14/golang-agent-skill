# Performance Reference

Extended performance optimization patterns for Go.

## String Operations Benchmark Comparison

```go
// BenchmarkFmtSprintf-8    10000000    120 ns/op    16 B/op    2 allocs/op
s := fmt.Sprintf("%d", n)

// BenchmarkStrconvItoa-8    50000000    28 ns/op     7 B/op    1 allocs/op
s := strconv.Itoa(n)
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

```go
// Benchmark: 1000 concatenations
// BenchmarkConcat-8        1000    5200000 ns/op
// BenchmarkBuilder-8    1000000       1200 ns/op

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
