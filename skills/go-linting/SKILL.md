---
name: go-linting
description: >-
  Use when setting up linting, configuring golangci-lint, or fixing linter
  warnings in Go projects. Provides recommended linter sets, golangci-lint
  configuration, CI integration, and Makefile targets.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go linting golangci-lint staticcheck revive govet ci
  category: languages
---

# Go Linting

Recommended linters and configuration for Go projects.

## golangci-lint Setup

golangci-lint is the standard linter aggregator for Go. Install it as a tool dependency in your module:

```bash
# Add to go.mod (Go 1.24+)
go get -tool github.com/golangci/golangci-lint/cmd/golangci-lint

# Or install directly
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

## Recommended Configuration

Place `.golangci.yml` at the project root. This uses the golangci-lint v2 config format:

```yaml
version: "2"

run:
  timeout: 5m
  go: "1.25"

linters:
  enable:
    - errcheck       # unchecked errors
    - govet          # go vet checks
    - staticcheck    # comprehensive static analysis (includes gosimple)
    - revive         # flexible linter, replaces golint
    - ineffassign    # unused assignments
    - unused         # unused code
    - misspell       # spelling in comments and strings
    - unconvert      # unnecessary type conversions
    - gocritic       # opinionated style and performance checks
    - errname        # error naming conventions (Err prefix)
    - errorlint      # error wrapping patterns
    - copyloopvar    # loop variable copy issues (pre-Go 1.22)
    - nilerr         # returning nil when err is not nil
    - bodyclose      # unclosed HTTP response bodies
    - prealloc       # slice preallocation
  settings:
    revive:
      rules:
        - name: exported
          arguments:
            - "checkPrivateReceivers"
        - name: blank-imports
        - name: context-as-argument
        - name: context-keys-type
        - name: error-return
        - name: error-strings
        - name: error-naming
        - name: increment-decrement
        - name: var-naming
        - name: package-comments
        - name: range
        - name: receiver-naming
        - name: indent-error-flow
        - name: empty-block
        - name: superfluous-else
        - name: unreachable-code
        - name: redefines-builtin-id
    gocritic:
      enabled-tags:
        - diagnostic
        - style
        - performance
    errcheck:
      check-type-assertions: true
      check-blank: true
    govet:
      enable-all: true
  exclusions:
    rules:
      # Allow unused parameters in interface implementations
      - linters:
          - revive
        text: "unused-parameter"
      # Test files can use dot imports
      - path: _test\.go
        linters:
          - revive
        text: "dot-imports"

formatters:
  enable:
    - goimports      # import formatting and grouping
  settings:
    goimports:
      local-prefixes:
        - yourcompany.com

issues:
  max-issues-per-linter: 0
  max-same-issues: 0
```

Update `local-prefixes` under `formatters.settings.goimports` to match your module path.

Note: this config uses golangci-lint v2 format (`version: "2"`). If you are on golangci-lint v1, run `golangci-lint migrate` to convert, or remove the `version` line and move `formatters` back into `linters`.

## Makefile Integration

```makefile
.PHONY: lint lint-fix

lint: ## Run linters
	go tool golangci-lint run ./...

lint-fix: ## Run linters with auto-fix
	go tool golangci-lint run --fix ./...
```

If golangci-lint is not a tool dependency:

```makefile
lint:
	golangci-lint run ./...
```

## CI Integration

### GitHub Actions

```yaml
name: lint
on: [push, pull_request]
jobs:
  golangci-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: "1.25"
      - uses: golangci/golangci-lint-action@v6
        with:
          version: latest
```

## Per-Linter Guidance

### errcheck

Finds unchecked errors. Fix by handling or explicitly ignoring:

```go
// Wrong: error ignored silently
json.Unmarshal(data, &v)

// Correct: handle the error
if err := json.Unmarshal(data, &v); err != nil {
    return fmt.Errorf("unmarshal config: %w", err)
}
```

### errorlint

Enforces proper error wrapping and comparison:

```go
// errorlint flags this: use errors.Is instead
if err == ErrNotFound { }

// Correct
if errors.Is(err, ErrNotFound) { }
```

### bodyclose

Catches unclosed HTTP response bodies:

```go
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close() // bodyclose checks for this
```

### govet

Runs the same checks as `go vet` plus additional analyzers. Key checks:

- **printf**: format string / argument mismatch
- **shadow**: variable shadowing
- **structtag**: malformed struct tags
- **copylocks**: passing locks by value

## Suppressing False Positives

```go
//nolint:errcheck // intentionally ignoring close error on read-only file
_ = f.Close()

//nolint:gocritic // hugeParam: passing by value is intentional here
func process(cfg Config) { }
```

Use `//nolint` comments sparingly. Prefer fixing the issue over suppressing it.
