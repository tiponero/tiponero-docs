---
sidebar_position: 2
title: Settings
---

# Settings

The settings page (`/admin/settings`) has sections for your profile, your security, and the wallet configuration.

## Profile

Update the profile information that the public sees. The widgets show these fields:

| Field | Description |
|---|---|
| **Username** | Login username (the API cannot change it) |
| **Display Name** | Shown on widgets as the recipient name |
| **Bio** | Short description shown on widgets |
| **Avatar URL** | URL to your profile picture (shown on widgets) |

Tiponero saves each change and shows an inline toast notification. The page does not reload because HTMX sends the request.

## Password

Change your admin password:

1. Enter your current password
2. Enter the new password
3. Confirm the new password
4. Click **Update Password**

Tiponero hashes the passwords with bcrypt at the default cost factor.

## Wallet Configuration

Configure the connection to your `monero-wallet-rpc` instance:

| Field | Description |
|---|---|
| **RPC URL** | JSON-RPC endpoint (for example, `http://127.0.0.1:18083/json_rpc`) |
| **RPC Username** | HTTP Digest Auth username for `monero-wallet-rpc` |
| **RPC Password** | HTTP Digest Auth password (encrypted at rest with AES-256-GCM) |
| **Wallet File** | Name of the wallet file on the RPC host |
| **Wallet Password** | Password for the wallet file (encrypted at rest) |

After you save, Tiponero **hot-swaps** the RPC client immediately. The dashboard shows the connection status (Connected/Disconnected) and you do not need to restart the server.

:::info
Tiponero encrypts the wallet passwords before storage. It uses AES-256-GCM with a key that comes from the `ENCRYPTION_KEY` environment variable. After you save a password, the UI and the API responses never show it.
:::

## Two-Factor Authentication (TOTP)

### Enabling 2FA

1. Click **Setup 2FA** in the settings page
2. Scan the QR code with your authenticator app (Google Authenticator, Authy, and other apps)
3. Enter the 6-digit code displayed in your authenticator
4. Click **Confirm**

The TOTP secret uses SHA1, 6 digits, and a 30-second period. All standard authenticator apps work with it.

### Disabling 2FA

1. Click **Disable 2FA**
2. Enter a valid 6-digit TOTP code from your authenticator to confirm
3. Tiponero removes 2FA from your account

### Login with 2FA

When 2FA is enabled, the login flow adds a second step:

1. Enter username and password
2. If the credentials are valid, a TOTP input form appears
3. Enter the 6-digit code from your authenticator
4. Tiponero creates the session when the code is correct

## API Keys

See [API Keys](/docs/admin/api-keys) for how to create and manage API keys.
