---
sidebar_position: 2
title: Quick Start
---

# Quick Start

Get Tiponero running in under five minutes using Docker Compose.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- A running `monero-wallet-rpc` instance (see [Monero Wallet RPC Setup](/docs/deployment/prerequisites))

If you don't have `monero-wallet-rpc` set up yet, you can still start Tiponero and configure the wallet connection later through the admin panel.

## 1. Clone the Repository

```bash
git clone https://github.com/tiponero/tiponero-core.git
cd tiponero-core
```

The repository includes a `docker-compose.yml` ready to use:

```yaml title="docker-compose.yml"
services:
  tiponero:
    build: .
    restart: unless-stopped
    ports:
      - "8080:8080"
    env_file:
      - .env
    environment:
      DATABASE_PATH: /data/tiponero.db
    volumes:
      - tiponero-data:/data

volumes:
  tiponero-data:
```

The Compose file builds the image from the Dockerfile, loads secrets from a `.env` file, and overrides `DATABASE_PATH` to point at the persistent volume.

## 2. Create Your `.env` File

Copy the example and set your encryption key:

```bash
cp .env.example .env
```

At minimum, set `ENCRYPTION_KEY`:

```bash title=".env"
ENCRYPTION_KEY=change-me-to-a-random-32char-key
```

:::warning
Replace `ENCRYPTION_KEY` with a random string of at least 32 characters. You can generate one with `openssl rand -hex 16`. This key encrypts wallet RPC passwords at rest. Losing it means you will need to re-enter your wallet configuration.
:::

## 3. Start the Service

```bash
docker compose up -d
```

## 4. Log In

Open [http://localhost:8080](http://localhost:8080) in your browser. The default credentials are:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `admin` |

:::danger
Change the default password immediately after your first login via **Settings > Password**.
:::

## 5. Configure Wallet Connection

Navigate to **Settings > Wallet** and enter your `monero-wallet-rpc` connection details:

| Field | Description |
|---|---|
| RPC URL | e.g. `http://127.0.0.1:18083/json_rpc` |
| RPC Username | Your `monero-wallet-rpc` `--rpc-login` username |
| RPC Password | Your `monero-wallet-rpc` `--rpc-login` password |
| Wallet File | Name of the wallet file on the `monero-wallet-rpc` host |
| Wallet Password | Password for the wallet file |

Once saved, the dashboard will show a green "Connected" indicator if the RPC connection succeeds.

## 6. Create Your First Widget

Navigate to **Widgets** and click **Create Widget**:

1. Give it a name (e.g. "My Donations")
2. Choose **Donation** mode (open amounts) or **Payment** mode (fixed amount with redirect)
3. Customize the button text, colors, and preset amounts
4. Save

Your widget is now live at `http://localhost:8080/widget/<widget-id>`.

## 7. Embed the Widget

Add the widget to any website with an iframe:

```html
<iframe
  src="http://localhost:8080/widget/<widget-id>"
  width="400"
  height="600"
  frameborder="0"
  style="border: none; border-radius: 8px;"
></iframe>
```

Or embed an SVG badge in markdown:

```markdown
[![Donate with Monero](http://localhost:8080/widget/<widget-id>/badge.svg)](http://localhost:8080/widget/<widget-id>)
```

## Next Steps

- [Configuration Reference](/docs/configuration/environment): all environment variables explained
- [API Reference](/docs/api/authentication): automate widget and transaction management
- [Deployment Guides](/docs/deployment/prerequisites): production deployment with Docker, NixOS, or bare-metal
- [Security](/docs/deployment/security): HTTPS, encryption keys, and 2FA setup
