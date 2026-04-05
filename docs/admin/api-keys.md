---
sidebar_position: 3
title: API Keys
---

# API Key Management

API keys provide programmatic access to the Tiponero REST API. They are created and managed through the admin panel at **Settings > API Keys**.

## Creating an API Key

1. Navigate to **Settings > API Keys**
2. Enter a descriptive name (e.g. "Production API", "CI Pipeline")
3. Click **Create API Key**
4. The raw key is displayed **once**: copy and store it securely

The key format is `tip_<32 hex characters>`:

```
tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

:::warning
The raw API key is shown only at creation time. It cannot be retrieved later. If you lose it, delete the key and create a new one.
:::

## Key Properties

| Property | Value |
|---|---|
| **Format** | `tip_` + 32 hex characters |
| **Expiry** | 365 days from creation |
| **Storage** | bcrypt-hashed (the raw key is not stored) |
| **Lookup** | 8-character prefix index for efficient candidate matching |

## Managing Keys

The API keys section shows all active keys with:

- **Name:** the descriptive name you assigned
- **Prefix:** first 12 characters for identification (e.g. `tip_a1b2c3d4...`)
- **Created:** creation date
- **Expires:** expiration date
- **Last used:** timestamp of the most recent API request using this key

### Deleting a Key

Click **Delete** next to any key to permanently revoke it. This takes effect immediately: any clients using this key will receive `401 Unauthorized` on their next request.

## Security Considerations

- API keys inherit the full permissions of the user who created them
- Keys are bcrypt-hashed before storage: even database access does not reveal the raw key
- Rotate keys periodically, especially if they may have been exposed
- Use descriptive names so you can identify which key is used where
- Delete unused keys promptly
