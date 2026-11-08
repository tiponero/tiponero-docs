---
sidebar_position: 2
title: Technology Stack
---

# Technology Stack

Tiponero is a Go application. It compiles to a single binary with all assets embedded. It needs no Node.js runtime, no external database server, and no separate frontend build at runtime.

## Runtime Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Language** | Go 1.26 (CGO enabled) | Core application, HTTP server, background workers |
| **Router** | [Chi v5](https://github.com/go-chi/chi) | HTTP routing and middleware |
| **Templating** | [templ](https://templ.guide/) | Type-safe Go HTML templates (`.templ` files compiled to Go) |
| **Frontend** | HTMX 2.x + Alpine.js 3.x | Interactive UI without a JavaScript framework |
| **Styling** | Tailwind CSS 3 | Utility-first CSS, compiled at build time |
| **Database** | SQLite (through `go-sqlite3`) | Embedded database, WAL mode, no external server |
| **Sessions** | `gorilla/sessions` | Cookie-based session management (7-day expiry) |
| **Auth** | `bcrypt` + TOTP | Password hashing and optional two-factor authentication |
| **Encryption** | AES-256-GCM + HKDF | Wallet password encryption at rest |
| **Monero RPC** | Custom JSON-RPC 2.0 client | HTTP Digest Auth with `icholy/digest` |
| **QR Codes** | `go-qrcode` | PNG QR code generation for `monero:` URIs |
| **Logging** | `zerolog` | Structured JSON/console logging |
| **Config** | `godotenv` + `os.Getenv` | `.env` file support, environment variable configuration |

## Build-Time Dependencies

You need these tools only for the build. The runtime does not need them:

| Tool | Purpose |
|---|---|
| **templ CLI** | Compiles `.templ` templates to `*_templ.go` files. The version comes from `tools/go.mod`. It must equal the `github.com/a-h/templ` version in `go.mod`. See [templ Toolchain](/docs/development/templ-toolchain). |
| **Tailwind CSS CLI v3** | Compiles `input.css` to `output.css` with utility class extraction. |
| **Go (CGO)** | Compiles the application and links SQLite through `go-sqlite3` |

## Development Tools

| Tool | Purpose |
|---|---|
| [Air](https://github.com/air-verse/air) | Hot-reload during development (watches `.go`, `.templ`, `.css` files) |
| [golangci-lint v2](https://golangci-lint.run/) | Go linting |

## Frontend Architecture

The admin panel and the widget views use **server-side rendering**. HTMX supplies the dynamic updates:

- **templ** generates Go functions that render HTML. No JavaScript templating runs at runtime.
- **HTMX** does partial page updates, form submissions, and real-time polling without custom JavaScript.
- **Alpine.js** gives lightweight client-side state for interactive UI elements (preset amount selection, form validation).
- **Tailwind CSS** compiles once at build time. The binary embeds the output CSS with `go:embed`.

The runtime has no JavaScript build step. The repository stores the JS libraries (HTMX, Alpine.js) as minified files in `static/js/`. The binary embeds them.
