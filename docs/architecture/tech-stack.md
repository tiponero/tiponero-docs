---
sidebar_position: 1
title: Technology Stack
---

# Technology Stack

Tiponero is a Go application that compiles to a single binary with all assets embedded. No Node.js runtime, no external database server, no separate frontend build at runtime.

## Runtime Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Language** | Go 1.26 (CGO enabled) | Core application, HTTP server, background workers |
| **Router** | [Chi v5](https://github.com/go-chi/chi) | HTTP routing and middleware |
| **Templating** | [templ](https://templ.guide/) | Type-safe Go HTML templates (`.templ` files compiled to Go) |
| **Frontend** | HTMX 2.x + Alpine.js 3.x | Interactive UI without a JavaScript framework |
| **Styling** | Tailwind CSS 3 | Utility-first CSS, compiled at build time |
| **Database** | SQLite (via `go-sqlite3`) | Embedded database, WAL mode, no external server |
| **Sessions** | `gorilla/sessions` | Cookie-based session management (7-day expiry) |
| **Auth** | `bcrypt` + TOTP | Password hashing and optional two-factor authentication |
| **Encryption** | AES-256-GCM + HKDF | Wallet password encryption at rest |
| **Monero RPC** | Custom JSON-RPC 2.0 client | HTTP Digest Auth via `icholy/digest` |
| **QR Codes** | `go-qrcode` | PNG QR code generation for `monero:` URIs |
| **Logging** | `zerolog` | Structured JSON/console logging |
| **Config** | `godotenv` + `os.Getenv` | `.env` file support, environment variable configuration |

## Build-Time Dependencies

These are only needed during compilation, not at runtime:

| Tool | Purpose |
|---|---|
| **templ CLI** | Compiles `.templ` templates to `*_templ.go` files |
| **Tailwind CSS CLI v3** | Compiles `input.css` to `output.css` with utility class extraction |
| **Go (CGO)** | Compiles the application, links SQLite via `go-sqlite3` |

## Development Tools

| Tool | Purpose |
|---|---|
| [Air](https://github.com/air-verse/air) | Hot-reload during development (watches `.go`, `.templ`, `.css` files) |
| [golangci-lint v2](https://golangci-lint.run/) | Go linting |

## Frontend Architecture

The admin panel and widget views use **server-side rendering** with HTMX for dynamic updates:

- **templ** generates Go functions that render HTML. No JavaScript templating at runtime.
- **HTMX** handles partial page updates, form submissions, and real-time polling without writing custom JavaScript.
- **Alpine.js** provides lightweight client-side state for interactive UI elements (preset amount selection, form validation).
- **Tailwind CSS** is compiled once at build time. The output CSS is embedded in the binary via `go:embed`.

There is no JavaScript build step at runtime. The JS libraries (HTMX, Alpine.js) are vendored as minified files in `static/js/` and embedded into the binary.
