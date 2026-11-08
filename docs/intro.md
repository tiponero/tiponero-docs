---
sidebar_position: 1
slug: /intro
title: Introduction
---

# Tiponero

A self-hosted Monero payment engine. One Go binary: an embedded SQLite database, an HTMX frontend, and one subaddress per transaction for private payments and donations.

## Why Tiponero

Most Monero payment setups publish one static address and stop there. Every payer sends to the same address, so you cannot attribute a payment. You also do not use subaddresses the way the protocol intends.

Tiponero changes this. Each transaction gets a unique subaddress. Tiponero shows the subaddress as a QR code and monitors it in real time. Your `monero-wallet-rpc` runs in view-only mode, so the server can monitor incoming funds but cannot spend them.

## Key Features

- **One subaddress per transaction:** the Monero protocol uses subaddresses at its core. Tiponero uses them correctly, one per payment.
- **View key only:** your `monero-wallet-rpc` runs in view-only mode. The server can monitor incoming funds but cannot spend them.
- **Embeddable widget:** add an iframe to any page. Tiponero makes a unique QR code for each payment and detects the payment in real time.
- **SVG badge:** embed a live donation badge in any Markdown document or website.
- **Admin panel:** track transactions, manage many widgets, and monitor your setup from a private dashboard.
- **Developer REST API:** full CRUD for widgets, transactions, wallet configuration, user profiles, and API keys.
- **TOTP two-factor authentication:** protect your admin panel with time-based one-time passwords.
- **Fiat conversion:** automatic XMR-to-fiat snapshots from CoinGecko at transaction creation.
- **Fully self-hosted:** you run it on your infrastructure. No cloud, no account.
- **MIT license:** read it, audit it, fork it.

## Who This Is For

| Audience | Use case |
|---|---|
| **Open source developers** | Accept donations without a third-party custodian in the payment path. |
| **Journalists and activists** | Monero protects your transactions at the protocol level. Tiponero makes sure your payment setup uses it correctly. |
| **Privacy-conscious creators** | Writers, podcasters, independent researchers. Accept support without giving financial data to a platform. |
| **Vendors and merchants** | Sell digital goods or services without a payment processor that knows your customers. Fixed-amount payment requests with automatic redirect after confirmation. |

## Quick Links

- [Quick Start](/docs/getting-started/quick-start): start a running instance in minutes
- [How It Works](/docs/getting-started/overview): the parts of the system on one page
- [API Reference](/docs/api/authentication): full REST API documentation
- [Deployment Guides](/docs/deployment/prerequisites): Docker, NixOS, bare-metal deployment, and Monero wallet RPC setup
- [API Collection](https://github.com/tiponero/tiponero-api-collection): import the full API collection (Bruno, Postman, OpenAPI)
- [GitHub Repository](https://github.com/tiponero/tiponero-core): source code
