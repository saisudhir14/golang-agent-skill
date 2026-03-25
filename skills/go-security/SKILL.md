---
name: go-security
description: >-
  Use when writing, reviewing, or auditing Go code for security. Covers input
  validation, SQL injection prevention, path traversal, secrets management,
  cryptography, HTTP security headers, and dependency scanning.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go security owasp injection xss sql-injection path-traversal crypto
  category: languages
---

# Go Security

Secure coding patterns for production Go. Covers OWASP top risks as they apply to Go.

## SQL Injection Prevention

Always use parameterized queries. Never interpolate user input into SQL strings.

```go
// Wrong: SQL injection
query := "SELECT * FROM users WHERE id = '" + id + "'"
db.Query(query)

// Correct: parameterized query
db.QueryContext(ctx, "SELECT * FROM users WHERE id = $1", id)
```

This applies to all database drivers. The placeholder syntax varies (`$1` for postgres, `?` for mysql).

## Path Traversal Prevention

### os.Root (Go 1.24+)

Use `os.Root` for scoped file access. Paths are resolved within the root directory and cannot escape it.

```go
root, err := os.OpenRoot("/var/data/uploads")
if err != nil {
    return err
}
defer root.Close()

// Safe: "../etc/passwd" is rejected
f, err := root.Open(userProvidedFilename)
if err != nil {
    return err
}
defer f.Close()
```

### Pre-Go 1.24

Use `filepath.Clean` and verify the result stays within the intended directory:

```go
cleaned := filepath.Clean(userInput)
absPath := filepath.Join(baseDir, cleaned)

// Verify the path is still under baseDir
if !strings.HasPrefix(absPath, filepath.Clean(baseDir)+string(os.PathSeparator)) {
    return fmt.Errorf("path traversal: %s", userInput)
}
```

## Input Validation

Validate all external input at system boundaries. Internal code can trust validated data.

```go
func parseUserID(raw string) (int64, error) {
    id, err := strconv.ParseInt(raw, 10, 64)
    if err != nil {
        return 0, fmt.Errorf("invalid user ID %q: %w", raw, err)
    }
    if id <= 0 {
        return 0, fmt.Errorf("user ID must be positive, got %d", id)
    }
    return id, nil
}
```

For structured input, decode into typed structs and validate fields:

```go
var req CreateUserRequest
if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
    http.Error(w, "invalid request body", http.StatusBadRequest)
    return
}
if err := req.Validate(); err != nil {
    http.Error(w, err.Error(), http.StatusBadRequest)
    return
}
```

## Secrets Management

Never hardcode secrets. Never log them.

```go
// Wrong: secret in source code
const apiKey = "do-not-hardcode-secrets"

// Wrong: secret in error message or log
slog.Info("connecting", "dsn", os.Getenv("DATABASE_URL"))

// Correct: read from environment, treat as opaque
dsn := os.Getenv("DATABASE_URL")
if dsn == "" {
    return errors.New("DATABASE_URL is required")
}
slog.Info("connecting to database") // no secret in log
```

Use `type Secret string` with a custom `String()` method to prevent accidental logging:

```go
type Secret string

func (s Secret) String() string { return "[REDACTED]" }

func (s Secret) GoString() string { return "[REDACTED]" }

// The actual value is accessible via explicit conversion
dsn := string(cfg.DatabaseURL)
```

## Cryptography

Use the standard library. Do not implement your own crypto.

```go
import "crypto/rand"

// Generate random bytes (for tokens, nonces)
token := make([]byte, 32)
if _, err := rand.Read(token); err != nil {
    return err
}

// Do NOT use math/rand for security-sensitive values
```

### Password Hashing

Use bcrypt or argon2. Never store passwords as plaintext or simple hashes.

```go
import "golang.org/x/crypto/bcrypt"

// Hash a password
hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)

// Verify a password
err := bcrypt.CompareHashAndPassword(hash, []byte(password))
```

## HTTP Security

### Timeouts

Always set timeouts on HTTP servers and clients. Default zero value means no timeout.

```go
srv := &http.Server{
    Addr:         ":8080",
    Handler:      handler,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
}

client := &http.Client{
    Timeout: 10 * time.Second,
}
```

### Response Headers

```go
func securityHeaders(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("Content-Security-Policy", "default-src 'self'")
        next.ServeHTTP(w, r)
    })
}
```

### Close Response Bodies

Unclosed HTTP response bodies leak connections:

```go
resp, err := client.Do(req)
if err != nil {
    return err
}
defer resp.Body.Close()
```

## Dependency Scanning

Keep dependencies up to date and scan for known vulnerabilities:

```bash
# Check for known vulnerabilities (built into Go)
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# Update all dependencies
go get -u ./...
go mod tidy
```

Run `govulncheck` in CI. It checks your code against the Go vulnerability database and only reports vulnerabilities in functions you actually call.

## Concurrency Safety

Shared mutable state without synchronization is a data race and a security risk. Use the race detector in tests:

```bash
go test -race ./...
```

Always run tests with `-race` in CI. Data races can cause memory corruption, which in certain contexts is exploitable.
