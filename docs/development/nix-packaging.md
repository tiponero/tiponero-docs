---
sidebar_position: 2
title: Nix Packaging
---

# Nix Packaging

The flake builds Tiponero with `pkgs.buildGoModule`. Nix supplies the Go toolchain, the templ command line tool, and the Tailwind CSS tool. The derivation compiles vendored Go dependencies, so the build does not need network access.

## Derivation

```nix title="flake.nix"
tiponero = pkgs.buildGoModule {
  pname = "tiponero";
  version = "1.0.0";
  src = pkgs.lib.cleanSource ./.;

  vendorHash = null;

  nativeBuildInputs = with pkgs; [ templ tailwindcss ];

  env.CGO_ENABLED = "1";

  preBuild = ''
    templ generate
    tailwindcss -i static/css/input.css -o static/css/output.css --minify
  '';

  subPackages = [ "cmd/tiponero" ];

  meta = {
    description = "Self-hosted Monero donation platform";
    mainProgram = "tiponero";
  };
};
```

| Attribute | Effect |
|---|---|
| `vendorHash = null` | Nix uses the committed `vendor/` directory. No fixed-output hash is necessary. |
| `nativeBuildInputs` | `templ` and `tailwindcss` come from nixpkgs, at the versions of the selected nixpkgs revision. |
| `env.CGO_ENABLED` | Must stay `"1"`. `go-sqlite3` needs a C compiler. |
| `preBuild` | Regenerates the templates and the CSS before `go build`. |
| `subPackages` | Builds `cmd/tiponero` only, and installs `bin/tiponero`. |

## The vendor directory

The repository commits `vendor/`. The `vendorHash = null` setting tells Nix to use that directory. Change the directory and the setting together:

1. Change a Go dependency.
2. Run `GOFLAGS=-mod=mod go mod vendor`.
3. Commit `go.mod`, `go.sum`, `vendor/`, and `vendor/modules.txt`.

A mismatch between `go.mod` and `vendor/modules.txt` stops the build in the `preBuild` phase with a vendor consistency error. If the repository stops committing `vendor/`, set `vendorHash` to a real hash and remove the directory from git.

## The templ version in nixpkgs

`nativeBuildInputs` takes the templ binary from nixpkgs. Thus a nixpkgs update can move the command line tool ahead of the Go module in `go.mod`. The build then fails with `undefined: templ.ResolveAttributeValue`. See [templ Toolchain](/docs/development/templ-toolchain).

Examine the templ version of a nixpkgs revision before an update:

```bash
nix eval --raw --impure --expr \
  '(builtins.getFlake "github:nixos/nixpkgs/<rev>").legacyPackages.x86_64-linux.templ.version'
```

Compare the output with `go list -m github.com/a-h/templ`. The two values must agree.

## Test against the nixpkgs of a consumer

A NixOS flake that imports Tiponero with `nixpkgs.follows` builds this package with its own nixpkgs. Test that combination before you update the consumer:

```bash
nix build --option sandbox false \
  --override-input nixpkgs \
  github:nixos/nixpkgs/0bb2eedc20e4739f29e8045034f2828ad7054f8f \
  .#default
```

`--option sandbox false` is necessary on an unprivileged LXC host, where the Nix sandbox cannot set up its namespaces.

The consumer test in the example above moved the nixpkgs input from `6201e203` to `0bb2eedc`. The templ version went from 0.3.1001 to 0.3.1020 in the same update. Reproduce the failure and the fix with these two commands:

```bash
nix eval --raw --impure --expr \
  '(builtins.getFlake "github:nixos/nixpkgs/6201e203d09599479a3b3450ed24fa81537ebc4e").legacyPackages.x86_64-linux.templ.version'
nix eval --raw --impure --expr \
  '(builtins.getFlake "github:nixos/nixpkgs/0bb2eedc20e4739f29e8045034f2828ad7054f8f").legacyPackages.x86_64-linux.templ.version'
```

## Remove the dependency on the nixpkgs templ

Two options remove the drift:

| Option | Result |
|---|---|
| Remove `templ generate` from `preBuild` | The build compiles the committed `*_templ.go` files. The templ version in nixpkgs no longer matters. A stale generated file then passes the build unnoticed. |
| Take the tool from `tools/go.mod` | Add `go -C tools tool templ generate -path ..` to `preBuild`. The build needs network access for the tools module, so it no longer works in a sandbox without a fetch. |

The current derivation uses neither option. It regenerates with the nixpkgs binary and needs a `go.mod` that matches.

## Docker

The `Dockerfile` installs templ with `go install github.com/a-h/templ/cmd/templ@<version>`. That line is a third place that names the templ version. The other two are `go.mod` and `tools/go.mod`. Update it in the same commit.
