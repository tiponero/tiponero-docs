---
sidebar_position: 3
title: API Keys
---

# API Key Management

API keys give programmatic access to the Tiponero REST API. You create and manage them in the admin panel at **Settings > API Keys**.

## Creating an API Key

1. Navigate to **Settings > API Keys**
2. Enter a descriptive name (for example, "Production API" or "CI Pipeline")
3. Click **Create API Key**
4. Tiponero shows the raw key **once**: copy it and store it in a safe location

The key format is `tip_<32 hex characters>`:

```
tip_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

:::warning
Tiponero shows the raw API key only at creation time. You cannot get it later. If you lose it, delete the key and create a new one.
:::

## Key Properties

| Property | Value |
|---|---|
| **Format** | `tip_` + 32 hex characters |
| **Expiry** | 365 days from creation |
| **Storage** | bcrypt-hashed (Tiponero does not store the raw key) |
| **Lookup** | 8-character prefix index that finds the candidate keys quickly |

## Managing Keys

The API keys section shows all active keys. For each key, it shows:

- **Name:** the descriptive name you assigned
- **Prefix:** first 12 characters for identification (for example, `tip_a1b2c3d4...`)
- **Created:** creation date
- **Expires:** expiration date
- **Last used:** the time of the most recent API request that used this key

### Deleting a Key

Click **Delete** next to a key to revoke it permanently. The change is immediate: a client that uses this key gets `401 Unauthorized` on the next request.

## Security Considerations

- An API key has all the permissions of the user who created it
- Tiponero hashes the keys with bcrypt before storage: access to the database does not show the raw key
- Rotate the keys regularly, in particular if an exposure is possible
- Use descriptive names to identify where each key belongs
- Delete unused keys promptly
