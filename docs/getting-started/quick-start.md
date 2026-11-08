---
sidebar_position: 2
title: Quick Start
---

# Quick Start

Start Tiponero in less than five minutes with Docker Compose.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- A running `monero-wallet-rpc` instance (see [Monero Wallet RPC Setup](/docs/deployment/prerequisites))

If `monero-wallet-rpc` is not ready, you can start Tiponero and set the wallet connection later in the admin panel.

## 1. Clone the Repository

```bash
git clone https://github.com/tiponero/tiponero-core.git
cd tiponero-core
```

The repository contains a `docker-compose.yml` file that you can use directly:

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

The Compose file builds the image from the Dockerfile, loads the secrets from a `.env` file, and sets `DATABASE_PATH` to the persistent volume.

## 2. Create Your `.env` File

Copy the example file and set your encryption key:

```bash
cp .env.example .env
```

At a minimum, set `ENCRYPTION_KEY`:

```bash title=".env"
ENCRYPTION_KEY=change-me-to-a-random-32char-key
```

:::warning
Replace `ENCRYPTION_KEY` with a random string of 32 characters or more. You can make one with `openssl rand -hex 16`. This key encrypts the wallet RPC passwords at rest. If you lose the key, you must enter the wallet configuration again.
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
Change the default password immediately after your first login. Use **Settings > Password**.
:::

## 5. Configure the Wallet Connection

Go to **Settings > Wallet** and enter your `monero-wallet-rpc` connection details:

| Field | Description |
|---|---|
| RPC URL | For example, `http://127.0.0.1:18083/json_rpc` |
| RPC Username | Your `monero-wallet-rpc` `--rpc-login` username |
| RPC Password | Your `monero-wallet-rpc` `--rpc-login` password |
| Wallet File | Name of the wallet file on the `monero-wallet-rpc` host |
| Wallet Password | Password for the wallet file |

After you save the values, the dashboard shows a green "Connected" indicator when the RPC connection succeeds.

## 6. Create Your First Widget

Go to **Widgets** and click **Create Widget**:

1. Give the widget a name (for example, "My Donations").
2. Select **Donation** mode (open amounts) or **Payment** mode (fixed amount with redirect).
3. Change the button text, the colors, and the preset amounts.
4. Save the widget.

The widget is live at `http://localhost:8080/widget/<widget-id>`.

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

To embed an SVG badge in Markdown, use this code:

```markdown
[![Donate with Monero](http://localhost:8080/widget/<widget-id>/badge.svg)](http://localhost:8080/widget/<widget-id>)
```

## Next Steps

- [Configuration Reference](/docs/configuration/environment): all environment variables explained
- [API Reference](/docs/api/authentication): automate widget and transaction management
- [Deployment Guides](/docs/deployment/prerequisites): production deployment with Docker, NixOS, or bare metal
- [Security](/docs/deployment/security): HTTPS, encryption keys, and 2FA setup
