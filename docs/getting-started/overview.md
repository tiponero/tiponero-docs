---
sidebar_position: 1
title: How It Works
---

# How It Works

Tiponero is a single Go binary that bundles an HTTP server, an embedded SQLite database, an HTMX-based admin panel, and a background payment monitor. It connects to your `monero-wallet-rpc` instance over JSON-RPC to generate subaddresses and track incoming transfers.

## Key Design Decisions

- **Single binary:** the compiler embeds all static assets (CSS, JS, images) via `go:embed`. No external file dependencies at runtime.
- **SQLite with WAL:** no separate database server needed. WAL mode gives concurrent read access during writes.
- **View-only wallet:** the `monero-wallet-rpc` instance runs with a view-only wallet. Tiponero can see incoming payments but cannot spend funds.
- **Per-transaction subaddresses:** each payment/donation gets a unique Monero subaddress via `create_address` RPC. Addresses are never reused.
- **Hot-swappable RPC client:** changing wallet configuration in the admin panel immediately updates the live RPC connection without restarting the server.
- **AES-256-GCM at rest:** Tiponero encrypts wallet RPC passwords before storage. Keys come from `ENCRYPTION_KEY` via HKDF, with separate derivation contexts for sessions and wallet data.

## Read Next

| Question | Page |
|---|---|
| Which libraries does it use? | [Technology Stack](/docs/architecture/tech-stack) |
| Which files do what? | [Project Structure](/docs/architecture/project-structure) |
| Which processes and ports are there? | [System Architecture](/docs/architecture/system-architecture) |
| What do the tables hold? | [Data Model](/docs/architecture/data-model) |
| What happens during a payment? | [Flows and Business Logic](/docs/architecture/flows) |
