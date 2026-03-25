---
name: go-error-handling
description: >-
  Use when writing, reviewing, or debugging Go error handling code. Covers error
  wrapping, sentinel errors, custom error types, error joining, single handling,
  and error flow patterns. Based on Google and Uber style guides.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go errors error-handling wrapping sentinel
  category: languages
---

# Go Error Handling

Comprehensive error handling patterns for production Go code.

## Return Errors, Do Not Panic

Production code must avoid panics. Return errors and let callers decide how to handle them.

```go
// Wrong
func run(args []string) {
    if len(args) == 0 {
        panic("an argument is required")
    }
}

// Correct
func run(args []string) error {
    if len(args) == 0 {
        return errors.New("an argument is required")
    }
    return nil
}

func main() {
    if err := run(os.Args[1:]); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}
```

## Error Wrapping

Use `%w` when callers need to inspect the underlying error with `errors.Is` or `errors.As`. Use `%v` when you want to hide implementation details or at system boundaries.

```go
// Preserve error chain for programmatic inspection
if err != nil {
    return fmt.Errorf("load config: %w", err)
}

// Hide internal details at API boundaries
if err != nil {
    return fmt.Errorf("database unavailable: %v", err)
}
```

Keep context succinct. Avoid phrases like "failed to" that pile up as errors propagate.

```go
// Wrong: produces "failed to x: failed to y: failed to create store: the error"
return fmt.Errorf("failed to create new store: %w", err)

// Correct: produces "x: y: new store: the error"
return fmt.Errorf("new store: %w", err)
```

## Joining Multiple Errors (Go 1.20+)

Use `errors.Join` when multiple operations can fail independently.

```go
var (
    ErrNameRequired  = errors.New("name required")
    ErrEmailRequired = errors.New("email required")
)

func validateUser(u User) error {
    var errs []error
    if u.Name == "" {
        errs = append(errs, ErrNameRequired)
    }
    if u.Email == "" {
        errs = append(errs, ErrEmailRequired)
    }
    return errors.Join(errs...)
}

// errors.Is works on joined errors
if err := validateUser(u); err != nil {
    if errors.Is(err, ErrNameRequired) {
        // matches even when joined with other errors
    }
}
```

## Error Types

Choose based on caller needs:

| Caller needs to match? | Message type | Approach |
|---|---|---|
| No | Static | `errors.New("something bad")` |
| No | Dynamic | `fmt.Errorf("file %q not found", file)` |
| Yes | Static | Exported `var ErrNotFound = errors.New("not found")` |
| Yes | Dynamic | Custom error type with `Error()` method |

## Sentinel Errors and errors.Is

```go
var (
    ErrNotFound    = errors.New("not found")
    ErrInvalidUser = errors.New("invalid user")
)

if errors.Is(err, ErrNotFound) {
    // handles ErrNotFound even when wrapped
}

var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("failed path:", pathErr.Path)
}
```

## Error Naming

Exported error variables use `Err` prefix. Custom error types use `Error` suffix.

```go
var ErrNotFound = errors.New("not found")

type NotFoundError struct {
    Resource string
}
func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s not found", e.Resource)
}
```

## Handle Errors Once

Do not log an error and also return it. The caller will likely log it again.

```go
// Wrong: logs and returns, causing duplicate logs
if err != nil {
    log.Printf("could not get user %q: %v", id, err)
    return err
}

// Correct: wrap and return, let caller decide
if err != nil {
    return fmt.Errorf("get user %q: %w", id, err)
}

// Also correct: log and degrade gracefully without returning error
if err := emitMetrics(); err != nil {
    log.Printf("could not emit metrics: %v", err)
}
```

## Error Strings

Do not capitalize error strings or end with punctuation. They often appear mid-sentence in logs.

```go
// Wrong
fmt.Errorf("Something bad happened.")

// Correct
fmt.Errorf("something bad happened")
```

## Indent Error Flow

Keep the happy path at minimal indentation. Handle errors first.

```go
// Wrong
if err != nil {
    // error handling
} else {
    // normal code
}

// Correct
if err != nil {
    return err
}
// normal code continues
```
