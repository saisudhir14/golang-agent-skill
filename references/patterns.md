# Patterns Reference

Extended Go patterns and idioms for production code.

## Functional Options (Complete Example)

```go
type Server struct {
    addr    string
    timeout time.Duration
    logger  *slog.Logger
    tls     *tls.Config
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func WithLogger(l *slog.Logger) Option {
    return func(s *Server) { s.logger = l }
}

func WithTLS(cfg *tls.Config) Option {
    return func(s *Server) { s.tls = cfg }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{
        addr:    addr,
        timeout: 30 * time.Second,
        logger:  slog.Default(),
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// Usage
func main() {
    srv := NewServer("localhost:8080",
        WithTimeout(60*time.Second),
        WithLogger(logger),
        WithTLS(tlsConfig),
    )
    _ = srv
}
```

## Interface Compliance Verification

```go
// Compile-time check that Handler implements http.Handler
var _ http.Handler = (*Handler)(nil)

// Works for any interface
var _ io.ReadWriteCloser = (*MyConn)(nil)
var _ fmt.Stringer = (*Status)(nil)
```

## Graceful Shutdown (Complete Example)

```go
func main() {
    ctx, stop := signal.NotifyContext(context.Background(),
        syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    srv := &http.Server{Addr: ":8080", Handler: handler}

    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            slog.Error("server error", "err", err)
            stop()
        }
    }()

    slog.Info("server started", "addr", ":8080")
    <-ctx.Done()
    slog.Info("shutting down")

    shutdownCtx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(shutdownCtx); err != nil {
        slog.Error("shutdown error", "err", err)
    }
}
```

## Enum Pattern with String Representation

```go
type Status int

const (
    StatusUnknown Status = iota
    StatusActive
    StatusInactive
    StatusDeleted
)

func (s Status) String() string {
    switch s {
    case StatusActive:
        return "active"
    case StatusInactive:
        return "inactive"
    case StatusDeleted:
        return "deleted"
    default:
        return "unknown"
    }
}
```

## Secure File Access with os.Root (Go 1.24+)

```go
func ServeUserFiles(userDir string) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        root, err := os.OpenRoot(userDir)
        if err != nil {
            http.Error(w, "directory not found", http.StatusNotFound)
            return
        }
        defer root.Close()

        // Safe: paths resolved relative to root
        // "../etc/passwd" attacks are rejected
        f, err := root.Open(r.URL.Query().Get("file"))
        if err != nil {
            http.Error(w, "file not found", http.StatusNotFound)
            return
        }
        defer f.Close()
        io.Copy(w, f)
    }
}
```

## Dependency Injection Pattern

```go
// Wrong: mutable global
var db *sql.DB

func init() {
    db, _ = sql.Open("postgres", os.Getenv("DSN"))
}

// Correct: dependency injection
type Server struct {
    db     *sql.DB
    cache  Cache
    logger *slog.Logger
}

func NewServer(db *sql.DB, cache Cache, logger *slog.Logger) *Server {
    return &Server{db: db, cache: cache, logger: logger}
}
```

## Embed Static Files (Go 1.16+)

```go
import "embed"

//go:embed templates/*
var templates embed.FS

//go:embed config.json
var configData []byte

//go:embed version.txt
var version string
```

## Sources

- [Google Go Style Guide - Best Practices](https://google.github.io/styleguide/go/best-practices)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Effective Go](https://go.dev/doc/effective_go)
