---
sidebar_position: 1
title: Environment Variables
---

# Environment Variables

Tiponero is configured entirely through environment variables. You can set them directly in your shell, in a `.env` file (loaded automatically via `godotenv`), or through your deployment system (Docker, NixOS, systemd `EnvironmentFile`, etc.).

## Reference

| Variable | Type | Default | Description |
|---|---|---|---|
| `PORT` | string | `8080` | HTTP listen port. |
| `BASE_URL` | string | `http://localhost:<PORT>` | Public-facing URL (e.g. `https://pay.example.com`). Used for generating widget URLs, QR codes, and cookie security. If set to an `https://` URL, cookies are marked `Secure`. |
| `DATABASE_PATH` | string | *in-memory* | Path to the SQLite database file (e.g. `/data/tiponero.db`). If unset, uses an in-memory database (data lost on restart). **Strongly recommended for any non-ephemeral deployment.** |
| `ENCRYPTION_KEY` | string | *none* | Master key for deriving encryption and session keys via HKDF. Must be at least 32 characters. Used to encrypt wallet RPC passwords at rest (AES-256-GCM). **The application starts without it but logs a warning: sessions and wallet config will not be secure.** |
| `FIAT_CURRENCY` | string | `USD` | ISO 4217 currency code (e.g. `USD`, `EUR`, `GBP`). XMR amounts are converted to fiat at transaction creation time via CoinGecko. |
| `CONFIRMATIONS` | integer | `10` | Number of Monero block confirmations required before a transaction is marked `confirmed`. |
| `TRANSACTION_EXPIRY` | duration | `1h` | Go duration string (e.g. `30m`, `2h`, `45m`). How long a pending transaction remains active before expiring. |

:::warning
While all variables have defaults or are optional, you should always set `DATABASE_PATH` and `ENCRYPTION_KEY` in any real deployment. Running without them means data is lost on restart and wallet credentials are stored insecurely.
:::

## Example `.env` File

A minimal `.env` file only needs the two variables without built-in defaults:

```bash title=".env"
DATABASE_PATH=./tiponero.db
ENCRYPTION_KEY=change-me-to-a-random-32char-key
```

A full example overriding all defaults:

```bash title=".env"
PORT=8080
BASE_URL=https://pay.example.com
DATABASE_PATH=/var/lib/tiponero/tiponero.db
ENCRYPTION_KEY=change-me-to-a-random-32char-key
FIAT_CURRENCY=EUR
CONFIRMATIONS=10
TRANSACTION_EXPIRY=1h
```

## Key Derivation

The `ENCRYPTION_KEY` is never used directly. Two sub-keys are derived using HKDF-SHA256:

| Derived Key | Context | Purpose |
|---|---|---|
| Session key | `"session"` | Signs and encrypts session cookies (`gorilla/sessions` CookieStore) |
| Wallet key | `"wallet"` | Encrypts `rpc_password` and `wallet_password` in the database (AES-256-GCM) |

:::warning
Changing `ENCRYPTION_KEY` after initial setup will:
- Invalidate all active sessions (users must log in again)
- Make stored wallet passwords unreadable (you will need to re-enter wallet configuration in Settings)

Store this key securely. If you lose it, wallet RPC credentials must be re-configured.
:::

## Wallet RPC Configuration

The `monero-wallet-rpc` connection details (URL, username, password, wallet file, wallet password) are **not** environment variables. They are configured through the admin panel (**Settings > Wallet**) or the [Wallet API](/docs/api/wallet), and stored encrypted in the database.

This design keeps sensitive wallet credentials out of environment files and process listings.

## Production Recommendations

| Variable | Recommendation |
|---|---|
| `BASE_URL` | Set to your public HTTPS URL. This enables secure cookies and correct URL generation. |
| `ENCRYPTION_KEY` | Generate with `openssl rand -hex 32` or equivalent. Never reuse across instances. |
| `CONFIRMATIONS` | `10` is a reasonable default. Lower for small donations, higher for large payments. |
| `TRANSACTION_EXPIRY` | `1h` works for most use cases. Increase for payment flows where users may need more time. |
| `DATABASE_PATH` | Use an absolute path on a persistent volume. Back up this file regularly. |
