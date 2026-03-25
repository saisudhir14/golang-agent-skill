---
name: go-project-layout
description: >-
  Use when starting a new Go project, organizing packages, or restructuring
  an existing Go codebase. Covers standard directory layout, package design,
  Makefile targets, Dockerfile patterns, and module setup.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go project-layout structure makefile dockerfile module
  category: languages
---

# Go Project Layout

Standard project structure and setup patterns for Go.

## Directory Structure

Small projects (libraries, CLIs) should stay flat. Only add structure when the project warrants it. Do not create directories speculatively.

### Application Layout

```
myapp/
├── cmd/
│   └── myapp/
│       └── main.go          # entry point, minimal logic
├── internal/
│   ├── server/
│   │   └── server.go        # HTTP server setup
│   ├── handler/
│   │   └── user.go          # HTTP handlers
│   ├── service/
│   │   └── user.go          # business logic
│   └── store/
│       └── postgres.go       # data access
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
├── .golangci.yml
└── README.md
```

### Library Layout

```
mylib/
├── mylib.go                  # primary package API
├── mylib_test.go
├── internal/
│   └── parse/                # unexported helpers
│       └── parse.go
├── go.mod
└── README.md
```

## Key Directories

### `cmd/`

Each subdirectory is an executable. Keep `main.go` minimal: parse flags, build dependencies, call `run()`.

```go
// cmd/myapp/main.go
package main

import (
    "context"
    "fmt"
    "os"

    "myapp/internal/server"
)

func main() {
    if err := run(context.Background()); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}

func run(ctx context.Context) error {
    srv, err := server.New()
    if err != nil {
        return fmt.Errorf("create server: %w", err)
    }
    return srv.Start(ctx)
}
```

### `internal/`

Packages under `internal/` cannot be imported by other modules. Use this for code that is not part of your public API. The Go toolchain enforces this.

### When NOT to use certain directories

- **`pkg/`**: Avoid. If code is meant to be imported, put it at the module root or in a named package. `pkg/` adds a directory with no meaning.
- **`src/`**: Not a Go convention. Do not use.
- **`models/`** / **`types/`** / **`utils/`**: These become dumping grounds. Name packages by what they do, not what they contain.

## Module Setup

```bash
mkdir myapp && cd myapp
go mod init github.com/yourorg/myapp
```

### go.mod with Tool Dependencies (Go 1.24+)

```go
module github.com/yourorg/myapp

go 1.25

tool (
    github.com/golangci/golangci-lint/cmd/golangci-lint
    golang.org/x/tools/cmd/stringer
)
```

## Makefile

```makefile
.PHONY: build test lint run clean

build: ## Build the binary
	go build -o bin/myapp ./cmd/myapp

test: ## Run tests
	go test -race -count=1 ./...

lint: ## Run linters
	go tool golangci-lint run ./...

run: build ## Build and run
	./bin/myapp

clean: ## Remove build artifacts
	rm -rf bin/
```

## Dockerfile

Multi-stage build for small, secure images:

```dockerfile
FROM golang:1.25 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /bin/myapp ./cmd/myapp

FROM gcr.io/distroless/static-debian12
COPY --from=build /bin/myapp /bin/myapp
ENTRYPOINT ["/bin/myapp"]
```

Key points:
- `CGO_ENABLED=0` for static binary (no libc dependency)
- Distroless base image: no shell, no package manager, smaller attack surface
- Copy `go.mod` and `go.sum` first for Docker layer caching

## Package Design Rules

1. **Name by purpose, not contents**: `store` not `models`, `auth` not `utils`
2. **One package, one idea**: a package should do one thing well
3. **Avoid circular imports**: if A imports B and B needs A, extract the shared type into a third package
4. **internal for private code**: anything under `internal/` is hidden from external importers
5. **Keep cmd/ thin**: main.go builds dependencies and calls into internal packages
6. **Accept interfaces, return structs**: define interfaces where they are used, return concrete types
