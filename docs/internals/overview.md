---
title: Internals Overview
sidebar_position: 1
---

# Internals Overview

These pages explain how Tiponero Core works. The documents use diagrams for the structure and the flows, and tables for the exact rules.

## Pages

| Page | Contents | Best for |
|---|---|---|
| [System architecture](/docs/internals/system-architecture) | Processes, protocols, deployment, secrets, failure modes | Operators and new maintainers |
| [Data model](/docs/internals/data-model) | SQLite tables, columns, relations, indexes, storage rules | Maintainers and integrators |
| [Flows and business logic](/docs/internals/flows) | Bootstrap, login, wallet setup, widget creation, payment flow, state machine, API auth | Maintainers and integrators |

## Reading paths

- **New maintainer:** architecture, then data model, then flows.
- **Operator:** architecture sections 5, 6, 10, and 11, then flows section 12.
- **Integrator:** flows sections 5, 6, and 9, then the [API Reference](/docs/api/authentication).
- **Reviewer:** flows section 13, then the data model notes.

## Conventions

| Term | Meaning |
|---|---|
| Widget | Public page at `/widget/{id}` that accepts a payment or a donation |
| Transaction | One payment or donation record with one subaddress |
| Subaddress | Unique Monero address that the wallet derives for one transaction |
| Piconero | Smallest XMR unit. 1 XMR = `1e12` piconero |
| Wallet RPC | The `monero-wallet-rpc` process that holds the wallet keys |
| Monitor | Background goroutine that polls the wallet every 30 seconds |
| Confirmation | A block added after the block that contains the transaction |
| Underpaid | Status for a transaction whose received amount is below the requested amount |
| Badge | SVG image with a QR code and live statistics |

Amounts appear in XMR in the UI and in piconero in the database. Widget preset amounts are stored in XMR. Each document states the unit when it gives a value.

Transaction status values are always lowercase: `pending`, `mempool`, `confirming`, `underpaid`, `confirmed`, `expired`.

## Diagrams

The diagrams use Mermaid. This site renders them. GitHub renders them in the repository view, but most Markdown viewers need a plugin.

## Source of truth

The Go source is the source of truth. These pages describe the behavior of the code in this repository. When the code and a page disagree, read the code and update the page.

The same pages also live in `docs/` of the [tiponero-core](https://github.com/tiponero/tiponero-core) repository. Change both when the behavior changes.

Useful entry points:

| Path | Contents |
|---|---|
| `cmd/tiponero/main.go` | Startup and shutdown |
| `internal/handlers/routes.go` | Complete route table |
| `internal/database/sqlite.go` | Database schema |
| `internal/services/monitor.go` | Payment monitor |
| `internal/views/widget/` | Public page templates |

## Related pages

- [Root README](https://github.com/tiponero/tiponero-core/blob/main/README.md)
- [License](https://github.com/tiponero/tiponero-core/blob/main/LICENSE)
- [API Reference](/docs/api/authentication)
