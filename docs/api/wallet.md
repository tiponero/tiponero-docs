---
sidebar_position: 5
title: Wallet
---

# Wallet API

You can manage the `monero-wallet-rpc` connection configuration. Each authenticated user has a maximum of one wallet configuration.

:::info
The API encrypts the wallet passwords (`rpc_password`, `wallet_password`) at rest with AES-256-GCM. The API **never returns** these passwords in a response.
:::

## Get Wallet

```
GET /api/v1/wallet
```

The response contains the current wallet configuration and the connection status.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/wallet
```

**Response `200 OK` (configured):**

```json
{
  "data": {
    "id": "d4e5f6a7-b8c9-0d1e-2f3a-4b5c6d7e8f90",
    "rpc_url": "http://127.0.0.1:18088",
    "rpc_user": "rpc_user",
    "wallet_file": "wallet",
    "connected": true,
    "created_at": 1700000000,
    "updated_at": 1700000000
  }
}
```

**Response `200 OK` (not configured):**

```json
{
  "data": {
    "id": "",
    "rpc_url": "",
    "rpc_user": "",
    "wallet_file": "",
    "connected": false,
    "created_at": 0,
    "updated_at": 0
  }
}
```

The `connected` field shows if Tiponero can successfully reach the `monero-wallet-rpc` instance. Tiponero calls `get_height` on the RPC endpoint and uses the result for this field.

## Create Wallet

```
POST /api/v1/wallet
```

This request creates the wallet configuration for the authenticated user. If the user already has a wallet configuration, the API returns `409 Conflict`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `rpc_url` | string | Yes | URL of the `monero-wallet-rpc` JSON-RPC endpoint |
| `rpc_user` | string | No | Username for HTTP Digest Auth |
| `rpc_password` | string | No | Password for HTTP Digest Auth (encrypted at rest) |
| `wallet_file` | string | No | Name of the wallet file on the RPC host |
| `wallet_password` | string | No | Password for the wallet file (encrypted at rest) |

**Example:**

```bash
curl -X POST \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "rpc_url": "http://127.0.0.1:18088",
    "rpc_user": "rpc_user",
    "rpc_password": "rpc_password",
    "wallet_file": "wallet",
    "wallet_password": "wallet_password"
  }' \
  https://your-instance.example.com/api/v1/wallet
```

**Response `201 Created`:**

```json
{"id": "d4e5f6a7-b8c9-0d1e-2f3a-4b5c6d7e8f90"}
```

**Error `409 Conflict`:**

```json
{"error": "wallet already exists"}
```

## Update Wallet

```
PATCH /api/v1/wallet
```

This request updates the wallet configuration. Include only the fields that you want to change. The API swaps the RPC client immediately, so you do not need a server restart.

**Example:**

```bash
curl -X PATCH \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "rpc_url": "http://127.0.0.1:18088",
    "rpc_user": "new_user",
    "wallet_file": "new_wallet"
  }' \
  https://your-instance.example.com/api/v1/wallet
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "d4e5f6a7-b8c9-0d1e-2f3a-4b5c6d7e8f90",
    "rpc_url": "http://127.0.0.1:18088",
    "rpc_user": "new_user",
    "wallet_file": "new_wallet",
    "connected": false,
    "created_at": 1700000000,
    "updated_at": 1700003600
  }
}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | UUID of the wallet configuration |
| `rpc_url` | string | URL of the `monero-wallet-rpc` endpoint |
| `rpc_user` | string | RPC authentication username |
| `wallet_file` | string | Wallet file name on the RPC host |
| `connected` | boolean | Whether the RPC endpoint is reachable now |
| `created_at` | integer | Unix timestamp of creation |
| `updated_at` | integer | Unix timestamp of last update |

:::note
The API never includes `rpc_password` or `wallet_password` in a response. They are write-only fields.
:::
