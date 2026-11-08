---
sidebar_position: 8
title: Security
---

# Security

Tiponero handles financial data and cryptographic credentials. This guide covers the security model and the best practices for production deployments.

## Encryption Key

The `ENCRYPTION_KEY` environment variable is the master secret. The application derives two sub-keys from it with HKDF-SHA256:

| Derived Key | Context String | Purpose |
|---|---|---|
| Session key | `"session"` | Signs and encrypts session cookies |
| Wallet key | `"wallet"` | Encrypts wallet RPC passwords at rest (AES-256-GCM) |
| Notification key | `"notifications"` | Encrypts webhook signing secrets and channel credentials (AES-256-GCM) |

### Generating a Key

```bash
# 32 random hex characters (recommended)
openssl rand -hex 16

# Or 32 random alphanumeric characters
openssl rand -base64 24 | tr -dc 'a-zA-Z0-9' | head -c 32
```

### Key Rotation

A change to `ENCRYPTION_KEY` has these effects:

| Impact | Recovery |
|---|---|
| All active sessions are invalidated | Users log in again |
| Stored wallet passwords become unreadable | Re-enter wallet config in Settings |
| API keys continue to work | API keys use bcrypt, not the encryption key |

There is no automated key rotation mechanism. To rotate the key, update the variable, restart the service, and enter the wallet credentials again.

## Password Storage

| Credential | Storage Method |
|---|---|
| Admin password | bcrypt (default cost) |
| API keys | bcrypt (default cost), prefix-indexed for lookup |
| Wallet RPC password | AES-256-GCM (key from HKDF) |
| Wallet file password | AES-256-GCM (key from HKDF) |
| TOTP secret | Stored in plaintext in the database |

:::info
API keys use a two-step check. An 8-character prefix finds the candidate rows in the database, then a bcrypt comparison confirms the match. This method does not scan the full table for each request.
:::

## Two-Factor Authentication (TOTP)

Enable TOTP 2FA in **Settings > Two-Factor Authentication**:

- Algorithm: SHA1
- Digits: 6
- Period: 30 seconds
- Compatible with Google Authenticator, Authy, and all standard TOTP apps

When TOTP is active, the login needs the password and the TOTP code. The application checks the TOTP code before it makes a session.

## Session Security

| Property | Value |
|---|---|
| Store | Cookie (signed + encrypted via `gorilla/sessions`) |
| Max age | 7 days |
| `HttpOnly` | Yes (not accessible via JavaScript) |
| `SameSite` | Lax |
| `Secure` | Yes, if `BASE_URL` starts with `https://` |

:::warning
Set `BASE_URL` to an `https://` URL in production. With this setting, cookies travel only over encrypted connections.
:::

## Network Security

### Bind Addresses

| Service | Recommended Bind | Exposed To |
|---|---|---|
| Tiponero | `127.0.0.1:<port>` | Reverse proxy only |
| monero-wallet-rpc | `127.0.0.1:18088` | Tiponero only |
| monerod RPC | `127.0.0.1:18081` | monero-wallet-rpc only |
| monerod P2P | `0.0.0.0:18080` | Internet (required for sync) |

Never expose the Tiponero, `monero-wallet-rpc`, or `monerod` RPC ports to the internet.

### Firewall Rules

Minimal firewall for a Tiponero server:

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow Monero P2P (required for blockchain sync)
sudo ufw allow 18080/tcp

# Allow HTTPS (if using nginx/Caddy, not needed for Cloudflare Tunnel)
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable
```

If you use Cloudflare Tunnel, you do not open port 443, because the traffic goes through the outbound tunnel.

## View-Only Wallet

Tiponero uses a **view-only wallet**. This restriction is a critical security boundary:

- The server can **see incoming payments** (via the view key)
- The server **cannot spend funds** (no spend key)
- Even if an attacker compromises the server, your funds are safe

Never use a full wallet (with spend key) on the Tiponero server.

## Database Security

The SQLite database file contains:

- User credentials (bcrypt-hashed passwords)
- Transaction records (amounts, subaddresses, donor info)
- Widget configurations
- Encrypted wallet credentials
- API key hashes

Protect the database file:

```bash
# Restrict file permissions
chmod 600 /var/lib/tiponero/tiponero.db

# Regular backups
cp /var/lib/tiponero/tiponero.db /backup/tiponero-$(date +%Y%m%d).db
```

## Secret Management

### Environment Files

If you use environment files (`.env` or the systemd `EnvironmentFile`), restrict the permissions:

```bash
chmod 600 /etc/tiponero/env
chown tiponero:tiponero /etc/tiponero/env
```

### agenix (NixOS)

For NixOS deployments, [agenix](https://github.com/ryantm/agenix) encrypts secrets at rest with the SSH host key of the server:

- Secrets are encrypted in the git repository
- Decrypted at boot time to `/run/agenix/`
- Accessible only to the specified owner/group
- Never written to the Nix store

See the [NixOS deployment guide](/docs/deployment/nixos) for configuration examples.

## Checklist

Before you go to production:

- [ ] Change the default `admin`/`admin` credentials
- [ ] Set `ENCRYPTION_KEY` to a random 32+ character string
- [ ] Set `BASE_URL` to your public `https://` URL
- [ ] Use a view-only wallet (not a full wallet)
- [ ] Enable TOTP 2FA on the admin account
- [ ] Bind Tiponero to `127.0.0.1` (not `0.0.0.0`)
- [ ] Put Tiponero behind a reverse proxy with TLS
- [ ] Restrict the database file permissions to the tiponero user
- [ ] Make regular database backups
- [ ] Close unnecessary firewall ports
