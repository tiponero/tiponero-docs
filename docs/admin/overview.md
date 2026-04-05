---
sidebar_position: 1
title: Dashboard Overview
---

# Admin Panel

The admin panel is a server-rendered web interface accessible at `/admin`. It provides a dashboard for monitoring transactions, managing widgets, and configuring your Tiponero instance.

## Accessing the Admin Panel

Navigate to `https://your-instance.example.com/admin` in your browser. The root URL (`/`) redirects to `/admin` automatically.

### Default Credentials

On first startup, Tiponero creates a default admin user:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `admin` |

:::danger
Change the default password immediately after your first login via **Settings > Password**.
:::

### Two-Factor Authentication

If TOTP 2FA is enabled, you will be prompted for a 6-digit code after entering your username and password. See [Settings](/docs/admin/settings) for setup instructions.

## Dashboard

The dashboard (`/admin/`) shows:

- **Stats cards:** total transactions, total XMR received, pending count, today's count and amount
- **Wallet status:** connection status of the `monero-wallet-rpc` instance (green = connected, red = disconnected)
- **Recent transactions:** the last 10 transactions with status, amount, donor name, and timestamp

## Transactions

The transactions page (`/admin/transactions`) provides:

- **Paginated list:** 10 transactions per page
- **Status filter:** filter by `pending`, `mempool`, `confirming`, `confirmed`, or `expired`
- **Transaction detail:** click any transaction to see full details (subaddress, tx hash, confirmations, donor info)
- **CSV export:** download all transactions as a CSV file via the **Export** button

## Widgets

The widgets page (`/admin/widgets`) allows you to:

- **View all widgets:** see name, mode, theme, and creation date
- **Create a widget:** set name, mode, preset amounts, button text, colors, and redirect URL
- **Delete a widget:** removes the widget (existing transactions are preserved with `widget_id` set to NULL)

## Navigation

| Path | Page |
|---|---|
| `/admin/` | Dashboard |
| `/admin/transactions` | Transaction list |
| `/admin/transactions/{id}` | Transaction detail |
| `/admin/widgets` | Widget list |
| `/admin/settings` | Settings (profile, password, wallet, 2FA, API keys) |
| `/admin/login` | Login page |
