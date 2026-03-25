# Testing Reference

Extended testing patterns and examples for Go.

## Complete Table-Driven Test Pattern

```go
func TestParseAmount(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int64
        wantErr error
    }{
        {name: "valid dollars", input: "$10.50", want: 1050},
        {name: "valid cents", input: "$0.99", want: 99},
        {name: "no symbol", input: "10.50", wantErr: ErrInvalidFormat},
        {name: "negative", input: "$-5.00", wantErr: ErrNegativeAmount},
        {name: "empty", input: "", wantErr: ErrInvalidFormat},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            got, err := ParseAmount(tt.input)

            if tt.wantErr != nil {
                if !errors.Is(err, tt.wantErr) {
                    t.Errorf("ParseAmount(%q) error = %v, want %v", tt.input, err, tt.wantErr)
                }
                return
            }

            if err != nil {
                t.Fatalf("ParseAmount(%q) unexpected error: %v", tt.input, err)
            }
            if got != tt.want {
                t.Errorf("ParseAmount(%q) = %d, want %d", tt.input, got, tt.want)
            }
        })
    }
}
```

## Test Helpers

```go
func newTestServer(t *testing.T) *Server {
    t.Helper()
    db := newTestDB(t)
    srv := NewServer(db)
    t.Cleanup(func() { srv.Close() })
    return srv
}

func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}
```

## Testing HTTP Handlers

```go
func TestGetUser(t *testing.T) {
    srv := newTestServer(t)

    req := httptest.NewRequest("GET", "/users/123", nil)
    w := httptest.NewRecorder()
    srv.ServeHTTP(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("status = %d, want %d", w.Code, http.StatusOK)
    }

    var got User
    if err := json.NewDecoder(w.Body).Decode(&got); err != nil {
        t.Fatalf("decode response: %v", err)
    }

    if diff := cmp.Diff(wantUser, got); diff != "" {
        t.Errorf("user mismatch (-want +got):\n%s", diff)
    }
}
```

## Benchmark Patterns (Go 1.24+)

```go
func BenchmarkSerialize(b *testing.B) {
    data := loadTestData()
    for b.Loop() {
        Serialize(data)
    }
}

func BenchmarkParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            doWork()
        }
    })
}
```

## synctest Patterns (Go 1.25+)

```go
// Testing timeout behavior
func TestClientTimeout(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        client := NewClient(WithTimeout(5 * time.Second))

        go func() {
            // Simulate slow server
            time.Sleep(10 * time.Second)
        }()

        time.Sleep(6 * time.Second)
        synctest.Wait()

        if !client.TimedOut() {
            t.Error("expected timeout")
        }
    })
}
```

## Sources

- [Go Blog - Using Subtests and Sub-benchmarks](https://go.dev/blog/subtests)
- [Go Wiki - Table Driven Tests](https://go.dev/wiki/TableDrivenTests)
- [go-cmp package](https://pkg.go.dev/github.com/google/go-cmp/cmp)
