---
sidebar_position: 4
title: User
---

# User API

Read and update the authenticated user's profile information.

## Get User

```
GET /api/v1/user
```

Returns the profile of the authenticated user.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/user
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "username": "alice",
    "display_name": "Alice",
    "bio": "Open source developer",
    "avatar_url": "https://example.com/avatar.png",
    "has_2fa": true,
    "created_at": 1700000000,
    "updated_at": 1700000000
  }
}
```

## Update User

```
PATCH /api/v1/user
```

Updates the authenticated user's profile. Only include the fields you want to change.

**Request body:**

| Field | Type | Description |
|---|---|---|
| `display_name` | string | Display name shown on widgets |
| `bio` | string | Short biography shown on widgets |
| `avatar_url` | string | URL to an avatar image shown on widgets |

:::info
The `username` and `password` cannot be changed through the API. Use the admin panel (**Settings > Profile** and **Settings > Password**) for those.
:::

**Example:**

```bash
curl -X PATCH \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "Alice",
    "bio": "Open source developer and privacy advocate",
    "avatar_url": "https://example.com/new-avatar.png"
  }' \
  https://your-instance.example.com/api/v1/user
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "username": "alice",
    "display_name": "Alice",
    "bio": "Open source developer and privacy advocate",
    "avatar_url": "https://example.com/new-avatar.png",
    "has_2fa": true,
    "created_at": 1700000000,
    "updated_at": 1700003600
  }
}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | UUID of the user |
| `username` | string | Login username |
| `display_name` | string\|null | Display name shown publicly on widgets |
| `bio` | string\|null | Short biography shown on widgets |
| `avatar_url` | string\|null | URL to an avatar image |
| `has_2fa` | boolean | Whether TOTP two-factor authentication is enabled |
| `created_at` | integer | Unix timestamp of account creation |
| `updated_at` | integer | Unix timestamp of last profile update |
