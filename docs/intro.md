---
sidebar_position: 1
slug: /intro
title: Introduction
---

# Tiponero

A self-hosted Monero payment engine. Single Go binary with embedded SQLite, HTMX frontend, and per-transaction subaddress generation for privacy-preserving payments and donations.

## Why Tiponero

Most Monero payment setups paste a static address and call it done. That means every payer sends to the same address, you have no way to attribute payments, and you are not using subaddresses the way Monero was designed to work.

Tiponero fixes this. Each transaction gets a unique subaddress, displayed as a QR code, monitored in real time. Your `monero-wallet-rpc` runs in view-only mode, so the server can monitor incoming funds but cannot spend them.

## Key Features

- **Unique subaddress per transaction:** the Monero protocol was built around subaddresses. Tiponero uses them correctly, one per payment.
- **View-key only:** your `monero-wallet-rpc` runs in view-only mode. The server can monitor incoming funds but cannot spend them.
- **Embeddable widget:** drop an iframe into any page. A unique QR code is generated per payment, with real-time detection.
- **SVG badge:** embed a live donation badge in any markdown document or website.
- **Admin panel:** track transactions, manage multiple widgets, and monitor your setup from a private dashboard.
- **Developer REST API:** full CRUD for widgets, transactions, wallet config, user profiles, and API keys.
- **TOTP two-factor authentication:** secure your admin panel with time-based one-time passwords.
- **Fiat conversion:** automatic XMR-to-fiat snapshots via CoinGecko at transaction creation time.
- **Fully self-hosted:** you run it on your infrastructure. No cloud, no account.
- **MIT licensed:** read it, audit it, fork it. No strings attached.

## Who Is This For

| Audience | Use case |
|---|---|
| **Open source developers** | Accept donations without routing payments through a third-party custodian. |
| **Journalists and activists** | Monero protects your transactions at the protocol level. Tiponero makes sure your payment setup uses it correctly. |
| **Privacy-conscious creators** | Writers, podcasters, independent researchers. Accept support without handing financial data to a platform. |
| **Vendors and merchants** | Sell digital goods or services without a payment processor that knows your customers. Fixed-amount payment requests with automatic redirect on confirmation. |

## Quick Links

- [Quick Start](/docs/getting-started/quick-start): get a running instance in minutes
- [How It Works](/docs/getting-started/overview): the parts of the system in one page
- [API Reference](/docs/api/authentication): full REST API documentation
- [Deployment Guides](/docs/deployment/prerequisites): Docker, NixOS, bare-metal deployment, and Monero wallet RPC setup
- [API Collection](https://github.com/tiponero/tiponero-api-collection): import the full API collection (Bruno, Postman, OpenAPI)
- [GitHub Repository](https://github.com/tiponero/tiponero-core): source code
