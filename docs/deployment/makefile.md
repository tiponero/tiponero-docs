---
sidebar_position: 4
title: Build from Source
---

# Build from Source

Build Tiponero from source using the provided Makefile. This produces a single static binary with all assets embedded.

## Requirements

| Tool | Version | Purpose |
|---|---|---|
| [Go](https://go.dev/dl/) | 1.25+ | Compiler (CGO must be enabled) |
| [templ](https://templ.guide/) | the version in `tools/go.mod` | Template code generation |
| [Tailwind CSS CLI](https://tailwindcss.com/blog/standalone-cli) | 3.x | CSS compilation |
| C compiler (gcc/clang) | *any* | Required for CGO (SQLite binding) |

### Installing Dependencies

```bash
# Go (via official installer or package manager)
# Verify CGO is available:
go env CGO_ENABLED  # should print "1"

# templ CLI (optional: make generate installs the pinned version on demand)
go -C tools install github.com/a-h/templ/cmd/templ

# Tailwind CSS CLI (via npm)
npm install -D tailwindcss
# Or download the standalone binary:
# https://github.com/tailwindlabs/tailwindcss/releases
```

### Using Nix (Alternative)

If you have Nix installed, all dependencies are provided by the dev shell:

```bash
nix develop
# Drops you into a shell with go, templ, tailwindcss, air, golangci-lint, sqlite
```

## Building

Clone the repository and run the build:

```bash
git clone https://github.com/tiponero/tiponero-core.git
cd tiponero-core
make build
```

The `make build` target runs three steps:

1. **`make generate`:** compiles `.templ` files to Go code
2. **`make css`:** compiles Tailwind CSS to `static/css/output.css`
3. **`go build`:** compiles the Go binary to `bin/tiponero`

The output binary is at `bin/tiponero`.

The Makefile runs each tool from the `tools` module. The versions of templ, Air, and golangci-lint come from `tools/go.mod`, not from your `PATH`. Keep the templ version in `tools/go.mod` equal to the `github.com/a-h/templ` version in `go.mod`. See [templ Toolchain](/docs/development/templ-toolchain).

## Makefile Targets

| Target | Command | Description |
|---|---|---|
| `make generate` | `go -C tools tool templ generate -path ..` | Compile `.templ` templates to Go, with the templ version from `tools/go.mod` |
| `make css` | `tailwindcss -i ... -o ... --minify` | Compile and minify Tailwind CSS |
| `make build` | generate + css + `go build` | Full production build |
| `make dev` | `go -C tools tool air` | Start with hot-reload (development) |
| `make lint` | `go -C tools tool golangci-lint run` | Run linter |
| `make clean` | `rm -rf bin/ tmp/ ...` | Remove build artifacts |

## Running the Binary

Create a `.env` file:

```bash title=".env"
PORT=8080
BASE_URL=http://localhost:8080
DATABASE_PATH=./tiponero.db
ENCRYPTION_KEY=$(openssl rand -hex 16)
FIAT_CURRENCY=USD
CONFIRMATIONS=10
TRANSACTION_EXPIRY=1h
```

Start the server:

```bash
./bin/tiponero
```

The server starts on the configured port. Open `http://localhost:8080` to access the admin panel.

## Running as a systemd Service

For production, run Tiponero as a systemd service:

```ini title="/etc/systemd/system/tiponero.service"
[Unit]
Description=Tiponero - Monero payment engine
After=network.target monero-wallet-rpc.service
Wants=monero-wallet-rpc.service

[Service]
Type=simple
User=tiponero
Group=tiponero
WorkingDirectory=/var/lib/tiponero
EnvironmentFile=/etc/tiponero/env
ExecStart=/usr/local/bin/tiponero
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Set up the user and directories:

```bash
# Create system user
sudo useradd -r -s /usr/sbin/nologin -d /var/lib/tiponero tiponero
sudo mkdir -p /var/lib/tiponero
sudo chown tiponero:tiponero /var/lib/tiponero

# Copy binary
sudo cp bin/tiponero /usr/local/bin/tiponero

# Create environment file
sudo mkdir -p /etc/tiponero
sudo tee /etc/tiponero/env << 'EOF'
PORT=8080
BASE_URL=https://pay.example.com
DATABASE_PATH=/var/lib/tiponero/tiponero.db
ENCRYPTION_KEY=your-random-32-character-key-here
FIAT_CURRENCY=USD
CONFIRMATIONS=10
TRANSACTION_EXPIRY=1h
EOF
sudo chmod 600 /etc/tiponero/env

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now tiponero
```

## Development Mode

For development with hot-reload:

```bash
make dev
```

This uses [Air](https://github.com/air-verse/air) to watch for file changes and automatically rebuild. The Air proxy runs on port 8081 by default, forwarding to the application on port 8080.

## Cross-Compilation

Since Tiponero uses CGO (for SQLite), cross-compilation requires a C cross-compiler. For most use cases, building on the target platform or using Docker is simpler:

```bash
# Build for linux/amd64 on linux/amd64 (no cross-compilation needed)
CGO_ENABLED=1 GOOS=linux GOARCH=amd64 go build -o bin/tiponero ./cmd/tiponero
```

For cross-platform builds, use the [Docker build method](/docs/deployment/docker) instead.
