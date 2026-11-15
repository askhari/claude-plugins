---
applies_to:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go conventions for this project

## Package layout

```
.
├── cmd/<binary>/main.go      # one main per binary, kept thin
├── internal/                  # private packages — not importable externally
│   ├── <domain>/
│   └── <domain>/<thing>_test.go
├── pkg/                       # public packages — only if external consumers exist
└── go.mod
```

- Anything that isn't part of a published API goes in `internal/`. Don't put domain code under `pkg/` "just in case".
- Package names are short, lowercase, no underscores. Avoid `util`, `common`, `helpers` — they always grow into junk drawers.

## Errors

- Errors are values. Return them, don't `panic`. Reserve `panic` for truly unrecoverable invariants.
- Wrap with `fmt.Errorf("doing X: %w", err)` so callers can `errors.Is` / `errors.As`.
- Sentinel errors live next to the function that returns them, exported as `Err...`.
- Don't log-and-return: log at the boundary (top of a handler), return everywhere else.

## Concurrency

- Goroutines must have a defined termination — accept a `context.Context` or a quit channel.
- Closing a channel signals "no more sends." The sender closes; never the receiver.
- `sync.Mutex` over channels for simple shared-state protection. Channels for handoffs.

## Tests

- `t.Run(name, func(t *testing.T) { ... })` subtests with table-driven cases.
- `testdata/` directory for fixtures; Go ignores it during builds.
- `t.Helper()` inside test helpers so failure lines point at the caller.
- No mocking frameworks for stdlib interfaces — write a tiny fake in the test file.

## Style

- `gofmt`/`goimports` formatted (the editor should do this; the hook ensures it).
- Imports in three groups: stdlib, third-party, local. Blank lines between.
- Receiver names are 1–2 letter abbreviations of the type, used consistently across all methods of that type.
- No `init()` with side effects. If a package needs setup, expose a `New...` function.

## Before declaring done

- [ ] `go build ./...`
- [ ] `go test ./...` (or `-race` if there's concurrency)
- [ ] `go vet ./...`
- [ ] `golangci-lint run` clean
- [ ] `go mod tidy` shows no diff

## Anti-patterns

- `interface{}` / `any` in exported signatures when a concrete type works. Generics over `any` for collections.
- Empty interfaces as "abstract base classes" — Go doesn't have those, stop trying.
- Returning `(value, ok, err)` triples. Pick one signaling mechanism per function.
- Comments that restate the function name. Comments explain *why*; the name explains *what*.
