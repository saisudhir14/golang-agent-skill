---
name: go-performance
description: >-
  Use when writing, reviewing, or optimizing Go code for performance. Covers
  string operations, memory allocation, preallocating slices and maps,
  strings.Builder, strconv, container-aware GOMAXPROCS, and runtime
  considerations for Go 1.25.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go performance optimization memory allocation
  category: languages
---

# Go Performance

Performance optimization patterns for production Go.

## Prefer strconv Over fmt

`strconv` is faster for primitive conversions.

```go
// Slower
s := fmt.Sprintf("%d", n)

// Faster
s := strconv.Itoa(n)
```

## Avoid Repeated String to Byte Conversions

```go
// Wrong: converts on every iteration
for i := 0; i < n; i++ {
    w.Write([]byte("hello"))
}

// Correct
data := []byte("hello")
for i := 0; i < n; i++ {
    w.Write(data)
}
```

## Specify Map Capacity

```go
// Wrong
m := make(map[string]int)

// Correct when size is known
m := make(map[string]int, len(items))
```

## Preallocate Slice Capacity

```go
// Wrong
var result []Item
for _, v := range input {
    result = append(result, transform(v))
}

// Correct
result := make([]Item, 0, len(input))
for _, v := range input {
    result = append(result, transform(v))
}
```

## Use strings.Builder for Concatenation

```go
// Wrong: creates many allocations
var s string
for _, part := range parts {
    s += part
}

// Correct
var b strings.Builder
b.Grow(totalLen) // optional: preallocate
for _, part := range parts {
    b.WriteString(part)
}
s := b.String()
```

## Container-Aware GOMAXPROCS (Go 1.25+)

Go 1.25 automatically adjusts GOMAXPROCS based on container CPU limits.

```go
// On Linux with cgroups, GOMAXPROCS now considers:
// - CPU bandwidth limits (CPU limit in Kubernetes)
// - Changes dynamically if limits change

// Automatic behavior is disabled if you set GOMAXPROCS explicitly:
// - Via GOMAXPROCS environment variable
// - Via runtime.GOMAXPROCS() call
```

This means Go programs in containers perform better out-of-the-box without manual GOMAXPROCS tuning.

## Resource Management (Go 1.24+)

### runtime.AddCleanup

Prefer `runtime.AddCleanup` over `runtime.SetFinalizer` for cleanup operations.

```go
func NewResource() *Resource {
    r := &Resource{handle: allocHandle()}
    runtime.AddCleanup(r, func(handle uintptr) {
        freeHandle(handle)
    }, r.handle)
    return r
}
```

Advantages over `SetFinalizer`:
- Multiple cleanups per object
- Works with interior pointers
- No cycle-related leaks
- Object freed promptly (single GC cycle)

### Weak Pointers (Go 1.24+)

The `weak` package provides weak references that don't prevent garbage collection.

```go
import "weak"

type Cache struct {
    mu    sync.Mutex
    items map[string]weak.Pointer[ExpensiveResource]
}

func (c *Cache) Get(key string) *ExpensiveResource {
    c.mu.Lock()
    defer c.mu.Unlock()
    if wp, ok := c.items[key]; ok {
        if r := wp.Value(); r != nil {
            return r
        }
        delete(c.items, key)
    }
    return nil
}
```

Use cases: caches, canonicalization maps, observer patterns.
