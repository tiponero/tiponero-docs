---
sidebar_position: 1
title: Dashboard Overview
---

# Admin Panel

The admin panel is a web interface that the server renders at `/admin`. It gives you a dashboard to monitor transactions, manage widgets, and configure your Tiponero instance.

## Accessing the Admin Panel

Navigate to `https://your-instance.example.com/admin` in your browser. The root URL (`/`) redirects to `/admin` automatically.

### Default Credentials

On the first startup, Tiponero creates a default admin user:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `admin` |

:::danger
Change the default password immediately after your first login. Use **Settings > Password**.
:::

### Two-Factor Authentication

If TOTP 2FA is enabled, the login page asks for a 6-digit code after you enter your username and password. See [Settings](/docs/admin/settings) for the setup steps.

## Dashboard

The dashboard (`/admin/`) shows:

- **Stats cards:** total transactions, total XMR received, pending count, and the count and amount for today
- **Wallet status:** the connection status of the `monero-wallet-rpc` instance (green = connected, red = disconnected)
- **Recent transactions:** the last 10 transactions with status, amount, donor name, and timestamp

## Transactions

The transactions page (`/admin/transactions`) gives you:

- **Paginated list:** 10 transactions per page
- **Status filter:** filter by `pending`, `confirming`, `partial`, `confirmed`, `reverted`, or `expired`
- **Transaction detail:** click any transaction to see full details (subaddress, tx hash, confirmations, donor info)
- **CSV export:** download all transactions as a CSV file with the **Export** button

## Widgets

On the widgets page (`/admin/widgets`) you can:

- **Create a widget:** full-width form with mode, price, confirmations, goal, and appearance
- **Preview and edit:** expand a widget to see it live, edit every field, and copy the embed code
- **Delete a widget:** Tiponero keeps the existing transactions and sets `widget_id` to NULL

## Navigation

| Path | Page |
|---|---|
| `/admin/` | Dashboard |
| `/admin/transactions` | Transaction list |
| `/admin/transactions/{id}` | Transaction detail |
| `/admin/widgets` | Widget list |
| `/admin/notifications` | Webhooks and notification channels |
| `/admin/settings` | Settings (profile, wallet, account security, API keys) |
| `/admin/login` | Login page |
