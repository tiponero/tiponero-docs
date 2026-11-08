---
sidebar_position: 1
title: templ Toolchain
---

# templ Toolchain

Tiponero compiles `.templ` files to Go code, then compiles the Go code. Two parts of templ take part in a build:

| Part | Version source |
|---|---|
| Command line tool (`templ generate`) | `tools/go.mod` |
| Go module `github.com/a-h/templ` | `go.mod`, plus the copy in `vendor/` |

The command line tool writes calls into the generated `*_templ.go` files. The Go module must contain each function that the tool writes. If the two versions disagree, the build stops at compile time.

## One version, two modules

Use the same version in the main module and in the tools module. The tools module declares the tool with the Go `tool` directive, so `go -C tools tool templ generate` always runs the version from `tools/go.mod`. You do not need a global `templ` binary.

Examine the two versions:

```bash
go list -m github.com/a-h/templ          # runtime module
go -C tools list -m github.com/a-h/templ # command line tool
```

The output must show the same version.

## Bump templ

1. Set the version in the main module:

   ```bash
   GOFLAGS=-mod=mod go get github.com/a-h/templ@v0.3.1020
   go mod tidy
   ```

2. Set the same version in the tools module:

   ```bash
   GOFLAGS=-mod=mod go -C tools get github.com/a-h/templ@v0.3.1020
   go -C tools mod tidy
   ```

3. Regenerate the templates with the pinned tool:

   ```bash
   go -C tools tool templ generate -path ..
   ```

4. Refresh the vendored copy:

   ```bash
   GOFLAGS=-mod=mod go mod vendor
   ```

5. Build:

   ```bash
   go build ./...
   ```

Commit `go.mod`, `go.sum`, `tools/go.mod`, `tools/go.sum`, `vendor/`, and the regenerated `*_templ.go` files in one commit. Do not edit the generated files by hand.

`GOFLAGS=-mod=mod` is necessary while `vendor/` exists. Go otherwise uses vendor mode and refuses to change the module graph.

## Troubleshooting

### `undefined: templ.ResolveAttributeValue`

```text
# github.com/tiponero/tiponero-core/internal/views/components
internal/views/components/ui_templ.go:262:52: undefined: templ.ResolveAttributeValue
```

The command line tool is newer than the Go module. The generated files call a function that the vendored runtime does not have.

The tool probably came from outside the module: nixpkgs, the Dockerfile, or a global binary. Run the bump procedure above.

Examine the vendored runtime after the bump:

```bash
grep -rn "func ResolveAttributeValue" vendor/github.com/a-h/templ/
```

### The generated files change on every build

The Go module and the command line tool are out of step in the other direction. Run the bump procedure to align them. A Nix build regenerates the templates with the templ binary from nixpkgs, so a stale module shows up as a diff after the build. See [Nix Packaging](/docs/development/nix-packaging).

### A global `templ` binary

`make generate` ignores the global binary, because it runs `go -C tools tool templ generate`. Use the Make target or the `go -C tools` command in CI and in editor tasks. A global `templ generate` can write files that match a different runtime version.
