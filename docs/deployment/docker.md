---
sidebar_position: 5
title: Docker
---

# Docker Deployment

Tiponero supplies a multi-stage Dockerfile. The Dockerfile makes a small container image, and the image uses the distroless base from Google.

## Quick Start

```bash
docker run -d \
  --name tiponero \
  -p 8080:8080 \
  -e PORT=8080 \
  -e BASE_URL=http://localhost:8080 \
  -e DATABASE_PATH=/data/tiponero.db \
  -e ENCRYPTION_KEY=$(openssl rand -hex 16) \
  -e FIAT_CURRENCY=USD \
  -e CONFIRMATIONS=10 \
  -e TRANSACTION_EXPIRY=1h \
  -v tiponero-data:/data \
  ghcr.io/tiponero/tiponero-core:latest
```

## Docker Compose

For a full setup with `monerod` and `monero-wallet-rpc`, use Docker Compose:

```yaml title="docker-compose.yml"
services:
  monerod:
    image: sethsimmons/simple-monerod:latest
    restart: unless-stopped
    volumes:
      - monero-data:/home/monero/.bitmonero
    ports:
      - "18080:18080"   # P2P (required for sync)
    command:
      - --prune-blockchain
      - --db-sync-mode=safe
      - --rpc-bind-ip=0.0.0.0
      - --rpc-bind-port=18081
      - --confirm-external-bind
      - --rpc-login=rpc_user:rpc_password
      - --non-interactive

  wallet-rpc:
    image: sethsimmons/simple-monerod:latest
    restart: unless-stopped
    depends_on:
      - monerod
    volumes:
      - wallet-data:/wallet
    entrypoint: monero-wallet-rpc
    command:
      - --rpc-bind-ip=0.0.0.0
      - --rpc-bind-port=18088
      - --confirm-external-bind
      - --rpc-login=rpc_user:rpc_password
      - --daemon-address=monerod:18081
      - --daemon-login=rpc_user:rpc_password
      - --wallet-file=/wallet/wallet
      - --password-file=/wallet/wallet.passwd
      - --non-interactive

  tiponero:
    image: ghcr.io/tiponero/tiponero-core:latest
    restart: unless-stopped
    depends_on:
      - wallet-rpc
    ports:
      - "8080:8080"
    environment:
      PORT: "8080"
      BASE_URL: "https://pay.example.com"
      DATABASE_PATH: "/data/tiponero.db"
      ENCRYPTION_KEY: "change-me-to-a-random-32char-key"
      FIAT_CURRENCY: "USD"
      CONFIRMATIONS: "10"
      TRANSACTION_EXPIRY: "1h"
    volumes:
      - tiponero-data:/data

volumes:
  monero-data:
  wallet-data:
  tiponero-data:
```

:::warning
Replace `ENCRYPTION_KEY` and the RPC credentials with your own values before you deploy. Generate a key with `openssl rand -hex 16`.
:::

### Initial Wallet Setup

Before you start the stack for the first time, create a wallet file in the `wallet-data` volume:

```bash
# Start only monerod first and wait for sync
docker compose up -d monerod

# Create the wallet volume and generate a view-only wallet
docker compose run --rm wallet-rpc \
  monero-wallet-cli --generate-new-wallet /wallet/wallet --password your_wallet_password

# Create the password file
docker compose run --rm wallet-rpc \
  sh -c 'echo "your_wallet_password" > /wallet/wallet.passwd'

# Now start the full stack
docker compose up -d
```

After the stack starts, configure the wallet connection in the Tiponero admin panel:

| Field | Value |
|---|---|
| RPC URL | `http://wallet-rpc:18088` |
| RPC Username | `rpc_user` |
| RPC Password | `rpc_password` |
| Wallet File | `wallet` |
| Wallet Password | `your_wallet_password` |

## Building the Image Locally

If you want to build the image yourself:

```bash
git clone https://github.com/tiponero/tiponero-core.git
cd tiponero-core
docker build -t tiponero .
```

### Dockerfile Details

The Dockerfile has two build stages:

**Stage 1: Builder** (`golang:1.26-bookworm`)
- Downloads the Tailwind CSS standalone binary
- Downloads Go module dependencies
- Runs the templ CLI from `tools/go.mod` with `go -C tools tool templ generate`. No templ version appears in the Dockerfile
- Compiles the Tailwind CSS
- Builds the Go binary with `CGO_ENABLED=1`

**Stage 2: Runtime** (`gcr.io/distroless/base-debian12`)
- Copies only the compiled binary
- No shell, no package manager, minimal attack surface
- Exposes port 8080

The final image is approximately 20MB.

## Volumes

| Mount Point | Purpose |
|---|---|
| `/data` | SQLite database file. **Back up this directory regularly.** |

## Health Check

Tiponero has a health check endpoint at `/health`:

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

:::info
The distroless image does not contain `wget` or `curl`. If you need container health checks, use the TCP health check of Docker. You can also add a health check binary in the build stage.
:::

## Environment Variables

See the [Configuration Reference](/docs/configuration/environment) for a full list of environment variables.

## Updating

```bash
docker compose pull tiponero
docker compose up -d tiponero
```

The SQLite schema is idempotent (`CREATE TABLE IF NOT EXISTS`), so database migrations occur automatically at startup.
