---
title: Architecture Overview
sidebar_position: 1
---

# Architecture Overview

These pages explain how Tiponero Core works: the parts of a deployed system, the database, and the request flows. Diagrams show the structure. Tables give the exact rules.

## Where to Start

| Page | Contents | Best for |
|---|---|---|
| [Technology Stack](/docs/architecture/tech-stack) | Libraries, tools, and build dependencies | Everyone |
| [Project Structure](/docs/architecture/project-structure) | File tree and key entry points | New maintainers |
| [System Architecture](/docs/architecture/system-architecture) | Processes, protocols, deployment, secrets, failure modes | Operators and new maintainers |
| [Data Model](/docs/architecture/data-model) | SQLite tables, columns, relations, indexes, storage rules | Maintainers and integrators |
| [Flows and Business Logic](/docs/architecture/flows) | Bootstrap, login, wallet setup, widget creation, payment flow, state machine, API auth | Maintainers and integrators |

## Reading Paths

- **New maintainer:** System Architecture, then Data Model, then Flows and Business Logic.
- **Operator:** System Architecture sections 5, 6, 10, and 11, then Flows section 12.
- **Integrator:** Flows sections 5, 6, and 9, then the [API Reference](/docs/api/authentication).
- **Reviewer:** Flows section 13, then the Data Model notes.

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
| Partial | Status of a transaction when the received amount is below the requested amount |
| Badge | SVG image with a QR code and live statistics |

The UI shows amounts in XMR. The database stores them in piconero. The database stores widget preset amounts in XMR. Each page gives the unit with the value.

Transaction status values are always lowercase: `pending`, `mempool`, `confirming`, `partial`, `confirmed`, `expired`.

## Source of Truth

The Go source is the source of truth. If a page and the code disagree, read the code and then update the page.

Some of these pages started in the `docs/` directory of the [tiponero-core](https://github.com/tiponero/tiponero-core) repository. The pages here are the maintained copies.

## Related Pages

- [Root README](https://github.com/tiponero/tiponero-core/blob/main/README.md)
- [License](https://github.com/tiponero/tiponero-core/blob/main/LICENSE)
- [API Reference](/docs/api/authentication)
