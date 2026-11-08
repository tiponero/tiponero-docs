---
sidebar_position: 4
title: User
---

# User API

You can read and update the profile of the authenticated user.

## Get User

```
GET /api/v1/user
```

The response contains the profile of the authenticated user.

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

This request updates the profile of the authenticated user. Include only the fields that you want to change.

**Request body:**

| Field | Type | Description |
|---|---|---|
| `display_name` | string | Display name that appears on widgets |
| `bio` | string | Short biography that appears on widgets |
| `avatar_url` | string | URL of an avatar image that appears on widgets |

:::info
You cannot change the `username` or the `password` through the API. Use the admin panel (**Settings > Profile** and **Settings > Password**) for those.
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
| `username` | string | Username for login |
| `display_name` | string\|null | Display name that appears publicly on widgets |
| `bio` | string\|null | Short biography that appears on widgets |
| `avatar_url` | string\|null | URL of an avatar image |
| `has_2fa` | boolean | Whether TOTP two-factor authentication is enabled |
| `created_at` | integer | Unix timestamp of account creation |
| `updated_at` | integer | Unix timestamp of last profile update |
