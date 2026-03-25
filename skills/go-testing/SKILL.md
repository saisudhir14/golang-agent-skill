---
name: go-testing
description: >-
  Use when writing, reviewing, or debugging Go tests and benchmarks. Covers
  table-driven tests, parallel execution, go-cmp, T.Context, T.Chdir, b.Loop,
  synctest for deterministic concurrency testing, and test failure messages.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go testing benchmark table-driven synctest go-cmp
  category: languages
---

# Go Testing

Production-grade testing patterns for Go.

## Table-Driven Tests with Parallel Execution

```go
func TestSplit(t *testing.T) {
    tests := []struct {
        name  string
        input string
        sep   string
        want  []string
    }{
        {name: "simple", input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        {name: "empty", input: "", sep: "/", want: []string{""}},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            got := strings.Split(tt.input, tt.sep)
            if diff := cmp.Diff(tt.want, got); diff != "" {
                t.Errorf("Split() mismatch (-want +got):\n%s", diff)
            }
        })
    }
}
```

## T.Context and T.Chdir (Go 1.24+)

```go
func TestWithContext(t *testing.T) {
    // T.Context returns a context canceled after test completes
    // but before cleanup functions run
    ctx := t.Context()
    result, err := doWork(ctx)
    if err != nil {
        t.Fatal(err)
    }
}

func TestWithChdir(t *testing.T) {
    // T.Chdir changes working directory for duration of test
    // and automatically restores it after
    t.Chdir("testdata")
    data, err := os.ReadFile("input.txt")
    // ...
}
```

## Benchmark with b.Loop (Go 1.24+)

```go
// Old way - error prone
func BenchmarkOld(b *testing.B) {
    input := setupInput() // counted in benchmark time!
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        process(input)    // compiler might optimize away
    }
}

// Go 1.24+ - preferred
func BenchmarkNew(b *testing.B) {
    input := setupInput() // setup runs once, excluded from timing
    for b.Loop() {
        process(input)    // compiler cannot optimize away
    }
}
```

Benefits of `b.Loop()`:
- Setup code runs exactly once per `-count`, automatically excluded from timing
- No need to call `b.ResetTimer()`
- Function call parameters and results are kept alive, preventing compiler optimization

## Testing Concurrent Code with synctest (Go 1.25+)

The `testing/synctest` package provides deterministic testing for concurrent code using synthetic time.

```go
import "testing/synctest"

func TestTimeout(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        ctx, cancel := context.WithTimeout(t.Context(), 5*time.Second)
        defer cancel()

        time.Sleep(4 * time.Second) // instant in real time
        if err := ctx.Err(); err != nil {
            t.Fatalf("unexpected timeout: %v", err)
        }

        time.Sleep(2 * time.Second)
        if ctx.Err() != context.DeadlineExceeded {
            t.Fatal("expected deadline exceeded")
        }
    })
}
```

Key concepts:
- `synctest.Test` creates an isolated "bubble" with synthetic time
- Time only advances when all goroutines in the bubble are blocked
- `synctest.Wait()` waits for all goroutines to be durably blocked

```go
func TestPeriodicTask(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        var count atomic.Int64
        ctx, cancel := context.WithCancel(t.Context())

        go func() {
            ticker := time.NewTicker(100 * time.Millisecond)
            defer ticker.Stop()
            for {
                select {
                case <-ctx.Done():
                    return
                case <-ticker.C:
                    count.Add(1)
                }
            }
        }()

        time.Sleep(350 * time.Millisecond)
        synctest.Wait()
        cancel()
        synctest.Wait()

        if got := count.Load(); got != 3 {
            t.Errorf("got %d ticks, want 3", got)
        }
    })
}
```

**Restrictions in synctest bubbles:** Do not call `t.Run()`, `t.Parallel()`, or `t.Deadline()`.

## Use go-cmp for Comparisons

Prefer `github.com/google/go-cmp/cmp` over `reflect.DeepEqual`.

```go
if diff := cmp.Diff(want, got); diff != "" {
    t.Errorf("mismatch (-want +got):\n%s", diff)
}

// With options
if diff := cmp.Diff(want, got, cmpopts.IgnoreUnexported(User{})); diff != "" {
    t.Errorf("mismatch (-want +got):\n%s", diff)
}
```

## Useful Test Failures

Include: what was wrong, inputs, actual result, expected result.

```go
// Wrong
if got != want { t.Error("wrong result") }

// Correct
if got != want { t.Errorf("Foo(%q) = %d; want %d", input, got, want) }
```

## Use t.Fatal for Setup Failures

```go
f, err := os.CreateTemp("", "test")
if err != nil {
    t.Fatal("failed to set up test")
}
```

## Interfaces Belong to Consumers

Define interfaces in the package that uses them, not the package that implements them.

```go
// Producer returns concrete type
package producer
type Thinger struct{}
func NewThinger() *Thinger { return &Thinger{} }

// Consumer defines interface it needs
package consumer
type Thinger interface { Thing() bool }
func Process(t Thinger) { }
```
