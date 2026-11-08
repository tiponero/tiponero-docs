---
sidebar_position: 1
title: How It Works
---

# How It Works

Tiponero is one Go binary. The binary contains an HTTP server, an embedded SQLite database, an HTMX admin panel, and a background payment monitor. Tiponero connects to your `monero-wallet-rpc` instance over JSON-RPC. It makes subaddresses and tracks incoming transfers through that connection.

## Key Design Decisions

- **One binary:** the compiler embeds all static assets (CSS, JS, images) with `go:embed`. The runtime needs no external files.
- **SQLite with WAL:** you do not need a separate database server. WAL mode gives concurrent read access during writes.
- **View-only wallet:** the `monero-wallet-rpc` instance runs with a view-only wallet. Tiponero can see incoming payments, but it cannot spend funds.
- **One subaddress per transaction:** each payment or donation gets a unique Monero subaddress through the `create_address` RPC. The system never reuses an address.
- **Hot-swappable RPC client:** a change to the wallet configuration in the admin panel updates the live RPC connection immediately. You do not restart the server.
- **AES-256-GCM at rest:** Tiponero encrypts the wallet RPC passwords before storage. The keys come from `ENCRYPTION_KEY` through HKDF, with separate derivation contexts for the sessions, the wallet data, and the notification credentials.

## Read Next

| Question | Page |
|---|---|
| Which libraries does it use? | [Technology Stack](/docs/architecture/tech-stack) |
| Which files do what? | [Project Structure](/docs/architecture/project-structure) |
| Which processes and ports are there? | [System Architecture](/docs/architecture/system-architecture) |
| What do the tables hold? | [Data Model](/docs/architecture/data-model) |
| What happens during a payment? | [Flows and Business Logic](/docs/architecture/flows) |
