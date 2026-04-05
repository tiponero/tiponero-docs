---
sidebar_position: 1
title: Authentication
---

# API Authentication

All `/api/v1/*` endpoints require authentication via a Bearer token in the `Authorization` header.

## API Collection

A ready-to-use API collection is available at [github.com/tiponero/tiponero-api-collection](https://github.com/tiponero/tiponero-api-collection). It includes:

- **[Bruno](https://www.usebruno.com/)** collection: clone the repo and open it directly in Bruno
- **[Postman export](https://github.com/tiponero/tiponero-api-collection/blob/main/Tiponero%20API.json)**: import `Tiponero API.json` into Postman
- **[OpenAPI spec](https://github.com/tiponero/tiponero-api-collection/blob/main/Tiponero%20API.yml)**: import `Tiponero API.yml` into any OpenAPI-compatible tool

## API Key Format

API keys follow the format `tip_<32 hex characters>`:

```
tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

Keys are generated through the admin panel (**Settings > API Keys**) or via the admin UI. The raw key is shown **exactly once** at creation time: it cannot be retrieved afterward. Store it securely.

## Usage

Include the key in the `Authorization` header as a Bearer token:

```bash
curl -H "Authorization: Bearer tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4" \
  https://your-instance.example.com/api/v1/stats
```

## Key Properties

| Property | Value |
|---|---|
| Format | `tip_` + 32 hex characters |
| Storage | bcrypt-hashed in database |
| Lookup | 8-character prefix index for fast candidate selection |
| Expiry | Maximum 365 days from creation |
| Last used | Updated asynchronously on each authenticated request |

## Error Responses

### Missing or malformed header

```bash
curl https://your-instance.example.com/api/v1/stats
```

```json
// 401 Unauthorized
{"error": "missing or invalid Authorization header"}
```

### Invalid key format

```bash
curl -H "Authorization: Bearer invalid_key" \
  https://your-instance.example.com/api/v1/stats
```

```json
// 401 Unauthorized
{"error": "invalid API key format"}
```

### Expired key

```json
// 401 Unauthorized
{"error": "API key has expired"}
```

## Response Format

All API responses use a consistent JSON envelope:

### Success

```json
{
  "data": { ... }
}
```

For list endpoints:

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 42,
    "total_pages": 3
  }
}
```

### Errors

```json
{
  "error": "description of what went wrong"
}
```

## Rate Limiting

There is no built-in rate limiting. If you expose the API publicly, configure rate limiting in your reverse proxy (nginx, Caddy, or Cloudflare).
