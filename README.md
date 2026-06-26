# claude-skills

Go best practices for AI coding agents. Distilled from Google Go Style Guide, Uber Go Style Guide, Effective Go, and Go Code Review Comments. Updated for Go 1.25.

**Install:** [skills.sh/saisudhir14/claude-skills](https://skills.sh/saisudhir14/claude-skills/golang)

```bash
npx skills add saisudhir14/claude-skills
```

## Skills

This repository provides **9 focused skills** that work across 20+ AI coding agents:

| Skill | Description |
|-------|-------------|
| **[golang](SKILL.md)** | Complete Go best practices guide (main skill) |
| **[go-error-handling](skills/go-error-handling/SKILL.md)** | Error wrapping, sentinel errors, custom types, error flow |
| **[go-concurrency](skills/go-concurrency/SKILL.md)** | Goroutines, channels, errgroup, mutexes, atomics |
| **[go-testing](skills/go-testing/SKILL.md)** | Table-driven tests, benchmarks, synctest, go-cmp |
| **[go-performance](skills/go-performance/SKILL.md)** | Memory allocation, string ops, GOMAXPROCS, weak pointers |
| **[go-code-review](skills/go-code-review/SKILL.md)** | Quick-reference checklist for Go code reviews |
| **[go-linting](skills/go-linting/SKILL.md)** | golangci-lint config, CI integration, linter guidance |
| **[go-project-layout](skills/go-project-layout/SKILL.md)** | Directory structure, Makefile, Dockerfile, module setup |
| **[go-security](skills/go-security/SKILL.md)** | SQL injection, path traversal, secrets, crypto, HTTP hardening |

## Supported Agents

Works with all SKILL.md-compatible agents:

| Agent | Status |
|-------|--------|
| Claude Code | Supported |
| Cursor | Supported |
| GitHub Copilot | Supported |
| Codex CLI | Supported |
| Gemini CLI | Supported |
| OpenCode | Supported |
| Amp | Supported |
| Windsurf | Supported |
| Zed | Supported |
| Goose | Supported |
| Roo Code | Supported |
| Kiro | Supported |
| Cline | Supported |
| Antigravity | Supported |
| Trae | Supported |
| Continue | Supported |
| Aider | Supported |
| Sourcegraph Cody | Supported |

## Topics Covered

- **Error Handling**: wrapping, sentinel errors, custom types, error joining (Go 1.20+)
- **Concurrency**: goroutine lifecycle, errgroup, mutexes, typed atomics (Go 1.19+), sync.Map (Go 1.24+)
- **Naming**: MixedCaps, initialisms, receivers, packages
- **Generics**: type constraints, generic aliases (Go 1.24+)
- **Iterators**: range-over-func (Go 1.23+), string/bytes iterators (Go 1.24+)
- **Testing**: table-driven, T.Context/T.Chdir (Go 1.24+), b.Loop (Go 1.24+), synctest (Go 1.25+)
- **Performance**: strconv, preallocating, strings.Builder, struct alignment
- **Resource Management**: runtime.AddCleanup, weak pointers, os.Root (Go 1.24+)
- **Patterns**: functional options, graceful shutdown, dependency injection
- **Runtime**: container-aware GOMAXPROCS (Go 1.25+), encoding/json/v2 (Go 1.25 experimental)
- **Linting**: golangci-lint setup, recommended linters, CI integration, suppression patterns
- **Project Layout**: cmd/, internal/, Makefile, Dockerfile, module setup
- **Security**: SQL injection, path traversal, secrets management, crypto, HTTP hardening, govulncheck

## Repository Structure

```
.
├── SKILL.md                          # Main skill (comprehensive guide)
├── skills/
│   ├── go-error-handling/SKILL.md    # Error handling deep dive
│   ├── go-concurrency/SKILL.md       # Concurrency patterns
│   ├── go-testing/SKILL.md           # Testing and benchmarks
│   ├── go-performance/SKILL.md       # Performance optimization
│   ├── go-code-review/SKILL.md       # Code review checklist
│   ├── go-linting/SKILL.md          # golangci-lint configuration
│   ├── go-project-layout/SKILL.md   # Project structure and setup
│   └── go-security/SKILL.md         # Secure coding patterns
├── references/
│   ├── error-handling.md             # Extended error handling examples
│   ├── concurrency.md                # Extended concurrency patterns
│   ├── testing.md                    # Extended testing patterns
│   ├── performance.md                # Extended performance patterns
│   ├── patterns.md                   # Extended Go patterns
│   └── gotchas.md                    # Common pitfalls reference
├── LICENSE
└── README.md
```

## Sources

- [Google Go Style Guide](https://google.github.io/styleguide/go/)
- [Uber Go Style Guide](https://github.com/uber-go/guide)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)
- [Go 1.25 Release Notes](https://go.dev/doc/go1.25)

## License

MIT
