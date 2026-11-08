---
sidebar_position: 1
title: Authentication
---

# API Authentication

All `/api/v1/*` endpoints require a Bearer token in the `Authorization` header for authentication.

## API Collection

You can import the full API collection from [github.com/tiponero/tiponero-api-collection](https://github.com/tiponero/tiponero-api-collection). It includes:

- **[Bruno](https://www.usebruno.com/)** collection: clone the repository and open it directly in Bruno
- **[Postman export](https://github.com/tiponero/tiponero-api-collection/blob/main/Tiponero%20API.json)**: import `Tiponero API.json` into Postman
- **[OpenAPI spec](https://github.com/tiponero/tiponero-api-collection/blob/main/Tiponero%20API.yml)**: import `Tiponero API.yml` into any OpenAPI-compatible tool

## API Key Format

API keys use the format `tip_<32 hex characters>`:

```
tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

You generate keys in the admin panel (**Settings > API Keys**). The admin panel shows the raw key **exactly once** at creation time. You cannot get the key again afterward. Store it securely.

## Usage

Send the key in the `Authorization` header as a Bearer token:

```bash
curl -H "Authorization: Bearer tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4" \
  https://your-instance.example.com/api/v1/stats
```

## Key Properties

| Property | Value |
|---|---|
| Format | `tip_` + 32 hex characters |
| Storage | Hashed with bcrypt in the database |
| Lookup | 8-character prefix index for fast candidate selection |
| Expiry | Maximum 365 days from creation |
| Last used | The server updates it asynchronously on each authenticated request |

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

All API responses use the same JSON envelope:

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

The API allows 240 requests per minute per IP. The limit is in memory and resets on restart. Add a reverse proxy limit if the instance is public.
