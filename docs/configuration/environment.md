---
sidebar_position: 1
title: Environment Variables
---

# Environment Variables

Tiponero takes all configuration from environment variables. You can set them in your shell, in a `.env` file that `godotenv` loads automatically, or through your deployment system (Docker, NixOS, systemd `EnvironmentFile`, and others).

## Reference

| Variable | Type | Default | Description |
|---|---|---|---|
| `PORT` | string | `8080` | HTTP listen port. |
| `BASE_URL` | string | `http://localhost:<PORT>` | Public URL (for example, `https://pay.example.com`). Tiponero uses it to make widget URLs and QR codes, and to set cookie security. If the URL starts with `https://`, Tiponero marks the cookies `Secure`. |
| `DATABASE_PATH` | string | *in-memory* | Path to the SQLite database file (for example, `/data/tiponero.db`). If you do not set it, Tiponero uses an in-memory database and loses the data at restart. **Set it for every deployment that must keep data.** |
| `ENCRYPTION_KEY` | string | *none* | Master key. Tiponero derives the encryption keys and the session keys from it with HKDF. Minimum length 32 characters. Tiponero encrypts the wallet RPC passwords with it at rest (AES-256-GCM). **The application starts without the key but writes a warning: the sessions and the wallet configuration are not secure.** |
| `FIAT_CURRENCY` | string | `USD` | ISO 4217 currency code (for example, `USD`, `EUR`, `GBP`). Tiponero converts the XMR amounts to fiat at transaction creation through CoinGecko. |
| `CONFIRMATIONS` | integer | `10` | Number of Monero block confirmations before Tiponero marks a transaction `confirmed`. |
| `TRANSACTION_EXPIRY` | duration | `1h` | Go duration string (for example, `30m`, `2h`, `45m`). Time before a pending transaction expires. |

:::warning
All variables have a default or are optional. But set `DATABASE_PATH` and `ENCRYPTION_KEY` in every real deployment. Without them, you lose the data at restart and Tiponero stores the wallet credentials insecurely.
:::

## Example `.env` File

A minimal `.env` file needs only the two variables that have no built-in default:

```bash title=".env"
DATABASE_PATH=./tiponero.db
ENCRYPTION_KEY=change-me-to-a-random-32char-key
```

A full example that overrides all defaults:

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

Tiponero never uses `ENCRYPTION_KEY` directly. It derives two sub-keys with HKDF-SHA256:

| Derived Key | Context | Purpose |
|---|---|---|
| Session key | `"session"` | Signs and encrypts the session cookies (`gorilla/sessions` CookieStore) |
| Wallet key | `"wallet"` | Encrypts `rpc_password` and `wallet_password` in the database (AES-256-GCM) |

:::warning
If you change `ENCRYPTION_KEY` after the initial setup:

- All active sessions become invalid. Users must log in again.
- The stored wallet passwords become unreadable. You must enter the wallet configuration again in Settings.

Store this key in a safe location. If you lose the key, you must configure the wallet RPC credentials again.
:::

## Wallet RPC Configuration

The `monero-wallet-rpc` connection details (URL, username, password, wallet file, wallet password) are **not** environment variables. You set them in the admin panel (**Settings > Wallet**) or through the [Wallet API](/docs/api/wallet). Tiponero stores them encrypted in the database.

This design keeps the sensitive wallet credentials out of the environment files and out of the process listings.

## Production Recommendations

| Variable | Recommendation |
|---|---|
| `BASE_URL` | Set it to your public HTTPS URL. This action enables the secure cookies and the correct URL generation. |
| `ENCRYPTION_KEY` | Make it with `openssl rand -hex 32` or an equivalent command. Never use the same key on two instances. |
| `CONFIRMATIONS` | `10` is a good default. Use a lower value for small donations and a higher value for large payments. |
| `TRANSACTION_EXPIRY` | `1h` works for most use cases. Increase it for payment flows where users need more time. |
| `DATABASE_PATH` | Use an absolute path on a persistent volume. Back up this file regularly. |
