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

## Performance and scalability

- Bound goroutine creation with a worker pool or a semaphore rather than spawning one per item over an input whose size you do not control.
- Every goroutine needs a termination path: check for leaks when the context is cancelled or when a channel is never drained.
- Preallocate slices and maps with a known capacity (`make([]T, 0, n)`) in hot loops instead of growing them by repeated `append`.
- `context` propagation is covered above; verify the cancellation actually reaches the blocking call, not just the function signature.
