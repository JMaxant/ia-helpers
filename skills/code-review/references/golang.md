# Go Conventions

**Detection triggers**: presence of `go.mod`. Version read from the `go` directive in `go.mod`.

- Code must be `gofmt`/`goimports`-clean; do not flag pure formatting issues that these tools already fix.
- Idiomatic error handling: `if err != nil { return ... }` close to the call site; wrap with `fmt.Errorf("...: %w", err)` to preserve the chain rather than swallowing or re-stringifying it.
- Reserve `panic` for truly unrecoverable programmer errors, never for expected/handleable error conditions.
- Keep interfaces small (often a single method) and defined on the consumer side, not pre-emptively on the producer side.
- Package names: short, lowercase, no underscores or mixedCaps.
- Receiver names: short and consistent across all methods of the same type; be consistent about pointer vs. value receivers for a given type.
- Pass `context.Context` as the first parameter of functions that do I/O or may be cancelled; do not store it in a struct field.
- Avoid package-level mutable state; prefer explicit dependency passing.
- Use table-driven tests (`[]struct{...}` + subtests via `t.Run`) for functions with multiple input/output cases.
- Use `defer` for cleanup (closing files, unlocking mutexes) right after the resource is acquired.
- Avoid naked returns in functions longer than a few lines — they hurt readability at the call site.
