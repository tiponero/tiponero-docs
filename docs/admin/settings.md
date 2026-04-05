---
sidebar_position: 2
title: Settings
---

# Settings

The settings page (`/admin/settings`) is divided into several sections for managing your profile, security, and wallet configuration.

## Profile

Update your public-facing profile information. These fields are displayed on your widgets:

| Field | Description |
|---|---|
| **Username** | Login username (cannot be changed via API) |
| **Display Name** | Shown on widgets as the recipient name |
| **Bio** | Short description shown on widgets |
| **Avatar URL** | URL to your profile picture (shown on widgets) |

Changes are saved with an inline toast notification (no page reload, powered by HTMX).

## Password

Change your admin password:

1. Enter your current password
2. Enter the new password
3. Confirm the new password
4. Click **Update Password**

Passwords are hashed with bcrypt at the default cost factor.

## Wallet Configuration

Configure the connection to your `monero-wallet-rpc` instance:

| Field | Description |
|---|---|
| **RPC URL** | JSON-RPC endpoint (e.g. `http://127.0.0.1:18083/json_rpc`) |
| **RPC Username** | HTTP Digest Auth username for `monero-wallet-rpc` |
| **RPC Password** | HTTP Digest Auth password (encrypted at rest with AES-256-GCM) |
| **Wallet File** | Name of the wallet file on the RPC host |
| **Wallet Password** | Password for the wallet file (encrypted at rest) |

After saving, the RPC client is **hot-swapped** immediately. The dashboard will show the connection status (Connected/Disconnected) without requiring a server restart.

:::info
Wallet passwords are encrypted before storage using AES-256-GCM with a key derived from the `ENCRYPTION_KEY` environment variable. They are never shown in the UI or API responses after being saved.
:::

## Two-Factor Authentication (TOTP)

### Enabling 2FA

1. Click **Setup 2FA** in the settings page
2. Scan the QR code with your authenticator app (Google Authenticator, Authy, etc.)
3. Enter the 6-digit code displayed in your authenticator
4. Click **Confirm**

The TOTP secret uses SHA1, 6 digits, and a 30-second period: compatible with all standard authenticator apps.

### Disabling 2FA

1. Click **Disable 2FA**
2. Enter a valid 6-digit TOTP code from your authenticator to confirm
3. 2FA is removed from your account

### Login with 2FA

When 2FA is enabled, the login flow adds a second step:

1. Enter username and password
2. If credentials are valid, a TOTP input form appears
3. Enter the 6-digit code from your authenticator
4. Session is created on successful verification

## API Keys

See [API Keys](/docs/admin/api-keys) for details on generating and managing API keys.
