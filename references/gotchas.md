# Common Gotchas Reference

Detailed explanations and fixes for common Go pitfalls.

## Loop Variable Capture (Fixed in Go 1.22+)

```go
// Pre-Go 1.22: All goroutines see last value
for _, v := range values {
    go func() {
        process(v) // Bug: captures loop variable
    }()
}

// Fix for pre-Go 1.22
for _, v := range values {
    v := v // shadow the loop variable
    go func() {
        process(v)
    }()
}

// Go 1.22+: Loop variables are per-iteration (no fix needed)
for _, v := range values {
    go func() {
        process(v) // Safe: v is unique per iteration
    }()
}
```

## Defer Argument Evaluation

Defer evaluates arguments when the defer statement executes, not when the deferred function runs.

```go
// Prints: 4 3 2 1 0 (i is evaluated at each defer statement, LIFO execution)
for i := 0; i < 5; i++ {
    defer fmt.Println(i)
}
```

Note on defer in loops: prior to Go 1.22, `for _, f := range files { defer f.Close() }` was a bug because `f` was shared across iterations. In Go 1.22+, loop variables are per-iteration, so this is safe. However, deferring inside loops still delays cleanup until the enclosing function returns. Prefer closing resources within each iteration when possible:

```go
for _, name := range files {
    func() {
        f, err := os.Open(name)
        if err != nil {
            return
        }
        defer f.Close()
        process(f)
    }()
}
```

## Nil Interface vs Nil Pointer

An interface containing a nil pointer is not nil.

```go
type MyError struct{}
func (e *MyError) Error() string { return "error" }

func returnsError() error {
    var e *MyError = nil
    return e // Returns non-nil interface containing nil pointer!
}

if err := returnsError(); err != nil {
    fmt.Println("error is not nil!") // This prints!
}

// Fix: return nil explicitly
func returnsError() error {
    var e *MyError = nil
    if e == nil {
        return nil
    }
    return e
}
```

## Use Result Before Checking Error (Go 1.25 Fix)

Go 1.25 fixed a compiler bug where using a result before checking for error sometimes didn't panic.

```go
// Wrong: uses f before checking err
f, err := os.Open("file.txt")
fmt.Println(f.Name()) // May panic if f is nil
if err != nil {
    return err
}

// Correct: always check error first
f, err := os.Open("file.txt")
if err != nil {
    return err
}
fmt.Println(f.Name()) // Safe: err was nil, so f is valid
```

## Map Iteration Order

Map iteration order is randomized. Do not depend on it.

```go
// Wrong: results vary between runs
for k, v := range m {
    results = append(results, v)
}

// Correct: sort keys first if order matters
keys := slices.Sorted(maps.Keys(m))
for _, k := range keys {
    results = append(results, m[k])
}
```

## Slice Append Gotcha

Append may or may not allocate new backing array.

```go
a := []int{1, 2, 3}
b := a[:2]
b = append(b, 4)
// a is now [1, 2, 4]! They share backing array

// Fix: use full slice expression to limit capacity
b := a[:2:2] // len=2, cap=2
b = append(b, 4) // forces new allocation
// a is still [1, 2, 3]
```

## Experimental: encoding/json/v2 (Go 1.25)

```go
// Enable with: GOEXPERIMENT=jsonv2

import (
    "encoding/json/jsontext"
    "encoding/json/v2"
)

// Offers: better performance, streaming, custom marshalers per call
// Existing encoding/json can use v2 engine internally
```

This is experimental and subject to change.

## Sources

- [Go FAQ](https://go.dev/doc/faq)
- [Go 1.22 Release Notes](https://go.dev/doc/go1.22)
- [Go 1.25 Release Notes](https://go.dev/doc/go1.25)
