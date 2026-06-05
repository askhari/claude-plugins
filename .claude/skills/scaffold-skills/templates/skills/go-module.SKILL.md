---
name: go-module
description: Use when creating a new Go module from scratch or refactoring an existing one to match this project's conventions — go.mod, cmd/internal/pkg layout, table tests, and lint setup.
context: fork
---

# Create a Go module

This skill walks through creating a new module that matches the conventions defined in `.claude/rules/go.md`.

## Step 1 — Clarify scope

Before writing anything, confirm with the user:

1. What does this module do? (one logical responsibility)
2. Is this a library, a binary, or both?
3. What is the module path? (`github.com/<org>/<repo>` — must match the import path consumers will use)
4. Minimum Go version? (`go 1.22` or newer unless told otherwise.)

## Step 2 — Scaffold the layout

For a library + binary:

```bash
mkdir -p cmd/<binary> internal/<domain> pkg/<public>  # pkg only if you have external consumers
go mod init <module-path>
touch cmd/<binary>/main.go internal/<domain>/<thing>.go internal/<domain>/<thing>_test.go README.md
```

For a library only:

```bash
mkdir -p <package> internal/<helper>
go mod init <module-path>
```

- `cmd/` holds one main package per binary. Keep these thin — they parse flags, wire dependencies, and call into `internal/`.
- `internal/` is for code that must not be imported externally. The Go toolchain enforces this.
- `pkg/` only when you have published external consumers.

## Step 3 — Write `cmd/<binary>/main.go`

Thin entrypoint. Example:

```go
package main

import (
    "context"
    "log/slog"
    "os"

    "<module-path>/internal/app"
)

func main() {
    ctx := context.Background()
    if err := app.Run(ctx, os.Args[1:]); err != nil {
        slog.Error("run failed", "err", err)
        os.Exit(1)
    }
}
```

The real work lives in `internal/app/app.go` (or your domain package), which is testable without spawning a process.

## Step 4 — Write the first package

Example `internal/<domain>/<thing>.go`:

```go
// Package <domain> does X.
package <domain>

// Thing is one cohesive abstraction in this domain.
type Thing struct {
    // ...
}

// New returns a Thing configured with the given options.
func New(opts ...Option) (*Thing, error) {
    // ...
}
```

- Package comments on `package <name>` only.
- Exported types and functions get a doc comment that starts with the identifier name.
- No `init()` with side effects.

## Step 5 — Write a table test

Example `internal/<domain>/<thing>_test.go`:

```go
package <domain>

import "testing"

func TestThing_Behavior(t *testing.T) {
    cases := []struct {
        name string
        in   string
        want string
    }{
        {"empty", "", ""},
        {"basic", "x", "X"},
    }
    for _, tc := range cases {
        t.Run(tc.name, func(t *testing.T) {
            // ...
        })
    }
}
```

## Step 6 — Configure `golangci-lint`

`.golangci.yml` at the repo root:

```yaml
run:
  timeout: 5m
linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - revive
    - gosec
    - bodyclose
    - rowserrcheck
```

Don't enable every linter — pick a deliberate set and keep it green.

## Step 7 — Validate

```bash
go mod tidy
go build ./...
go test ./...
go vet ./...
golangci-lint run
```

All five must pass before the scaffold is "done."

## Step 8 — Hand back to the user

Summarize what was created, the module path, the binary command (if any), and the package public surface. Note follow-up steps: setting up the CI workflow, adding the module to the workspace if applicable.
