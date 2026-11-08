---
sidebar_position: 7
title: API Keys
---

# API Keys

You can list and delete API keys. You create keys in the admin panel (**Settings > API Keys**).

:::info
You can create API keys only in the admin panel, not through the API itself. This is a deliberate security decision: an active admin session is necessary to create a key.
:::

## List API Keys

```
GET /api/v1/keys
```

The response contains all API keys of the authenticated user. The API never returns the raw key value, only the prefix and the metadata.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/keys
```

**Response `200 OK`:**

```json
{
  "data": [
    {
      "id": "e1f2a3b4-c5d6-7890-abcd-ef1234567890",
      "name": "Production API",
      "prefix": "tip_a1b2c3d4...",
      "created_at": 1700000000,
      "expires_at": 1731536000,
      "last_used_at": 1700100000
    },
    {
      "id": "f2a3b4c5-d6e7-8901-bcde-f12345678901",
      "name": "CI/CD Pipeline",
      "prefix": "tip_e5f6a7b8...",
      "created_at": 1700050000,
      "expires_at": 1731586000,
      "last_used_at": null
    }
  ]
}
```

## Delete API Key

```
DELETE /api/v1/keys/{id}
```

This request deletes an API key permanently. You cannot undo this action. Any client that uses this key loses access immediately.

**Example:**

```bash
curl -X DELETE \
  -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/keys/e1f2a3b4-c5d6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{"deleted": "e1f2a3b4-c5d6-7890-abcd-ef1234567890"}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | UUID of the API key |
| `name` | string | Human-readable name for the key |
| `prefix` | string | First 12 characters of the key (for identification only) |
| `created_at` | integer | Unix timestamp of creation |
| `expires_at` | integer | Unix timestamp of expiration (max 365 days from creation) |
| `last_used_at` | integer\|null | Unix timestamp of last use, or `null` if never used |

## Endpoint Summary

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/v1/keys` | List all API keys |
| `DELETE` | `/api/v1/keys/{id}` | Delete an API key |
