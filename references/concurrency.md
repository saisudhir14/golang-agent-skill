# Concurrency Reference

Extended concurrency patterns and examples for Go.

## Worker Pool Pattern

```go
func WorkerPool(ctx context.Context, jobs <-chan Job, workers int) error {
    g, ctx := errgroup.WithContext(ctx)

    for i := 0; i < workers; i++ {
        g.Go(func() error {
            for {
                select {
                case <-ctx.Done():
                    return ctx.Err()
                case job, ok := <-jobs:
                    if !ok {
                        return nil
                    }
                    if err := process(ctx, job); err != nil {
                        return fmt.Errorf("process job %s: %w", job.ID, err)
                    }
                }
            }
        })
    }

    return g.Wait()
}
```

## Fan-Out / Fan-In Pattern

```go
func FanOutFanIn(ctx context.Context, urls []string) ([]Result, error) {
    g, ctx := errgroup.WithContext(ctx)
    results := make([]Result, len(urls))

    for i, url := range urls {
        g.Go(func() error {
            r, err := fetch(ctx, url)
            if err != nil {
                return err
            }
            results[i] = r
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err
    }
    return results, nil
}
```

## Rate-Limited Operations

```go
func RateLimited(ctx context.Context, items []Item) error {
    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(10) // max 10 concurrent

    for _, item := range items {
        g.Go(func() error {
            return process(ctx, item)
        })
    }

    return g.Wait()
}
```

## Context Cancellation Pattern

```go
func longOperation(ctx context.Context) error {
    resultCh := make(chan result, 1)

    go func() {
        // On context cancellation, this goroutine still runs to completion
        // and sends to the buffered channel (then gets GC'd).
        // Pass ctx into expensiveWork if it supports cancellation.
        resultCh <- expensiveWork()
    }()

    select {
    case <-ctx.Done():
        return ctx.Err()
    case r := <-resultCh:
        return r.err
    }
}
```

## Sources

- [Go Blog - Pipelines and Cancellation](https://go.dev/blog/pipelines)
- [Go Blog - Context](https://go.dev/blog/context)
- [errgroup package](https://pkg.go.dev/golang.org/x/sync/errgroup)
