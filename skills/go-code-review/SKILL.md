---
name: go-code-review
description: >-
  Use when reviewing Go code or preparing code for review. Quick-reference
  checklist covering naming, error handling, concurrency, testing, imports,
  documentation, and common pitfalls. Based on Go Wiki CodeReviewComments.
version: "2.0.0"
license: MIT
metadata:
  author: saisudhir14
  tags: golang go code-review checklist linting style
  category: languages
---

# Go Code Review Checklist

Quick-reference checklist for reviewing Go code. Each item links to deeper guidance in the parent skill.

## Naming

- [ ] MixedCaps used (no underscores)
- [ ] Initialisms are consistent case (URL, ID, HTTP)
- [ ] Variable names match scope (short for local, descriptive for global)
- [ ] Receiver names are 1-2 letters, consistent across methods
- [ ] Package names are lowercase single words, no util/common/misc
- [ ] No name repetition (package.Method, not package.PackageMethod)

## Error Handling

- [ ] Errors returned, not panicked (production code)
- [ ] Error wrapping uses `%w` or `%v` appropriately
- [ ] Error context is succinct (no "failed to" chains)
- [ ] Errors handled once (not logged AND returned)
- [ ] Error strings are lowercase, no trailing punctuation
- [ ] Happy path at minimal indentation (error-first)
- [ ] Sentinel errors use `Err` prefix, error types use `Error` suffix

## Concurrency

- [ ] Channel buffers are 0 or 1 (or justified)
- [ ] Goroutine lifetimes are documented
- [ ] errgroup used for error-returning goroutines
- [ ] Functions are synchronous unless concurrency is essential
- [ ] Mutexes are zero-value, unexported, not embedded in public structs
- [ ] Typed atomics used (Go 1.19+)

## Testing

- [ ] Table-driven tests with named subtests
- [ ] Subtests run in parallel where safe
- [ ] go-cmp used for struct comparisons
- [ ] Failure messages include input, got, want
- [ ] t.Fatal for setup errors, t.Error for test assertions
- [ ] Interfaces defined in consumer packages

## Imports

- [ ] Three groups: stdlib, external, internal
- [ ] No unnecessary renames
- [ ] No dot imports (except circular dep tests)
- [ ] Blank imports only in main/tests

## Structs

- [ ] Field names used in initialization (no positional)
- [ ] Zero value fields omitted
- [ ] Types not embedded in public structs
- [ ] JSON field tags on marshaled structs

## Slices and Maps

- [ ] Nil slices preferred over empty slices
- [ ] Copied at boundaries to prevent mutation
- [ ] Capacity preallocated when size is known
- [ ] Standard library slices/maps packages used

## Performance

- [ ] strconv used over fmt for conversions
- [ ] No repeated string-to-byte conversions
- [ ] Map and slice capacity preallocated
- [ ] strings.Builder used for concatenation

## Documentation

- [ ] Exported declarations have doc comments
- [ ] Comments are full sentences starting with declared name
- [ ] Package has package comment

## Patterns

- [ ] Functional options for complex constructors
- [ ] Interface compliance verified at compile time
- [ ] defer used for resource cleanup
- [ ] Context is first parameter
- [ ] No mutable globals (dependency injection instead)
- [ ] Type assertions use two-value form
- [ ] time.Duration used instead of raw integers
- [ ] Enums start at one (zero = invalid)

## Common Gotchas

- [ ] No loop variable capture bugs (Go 1.22+ or shadowed)
- [ ] Defer argument evaluation understood
- [ ] Nil interface vs nil pointer handled correctly
- [ ] Error checked before using result
- [ ] No map iteration order dependency
- [ ] Slice append backing array understood
