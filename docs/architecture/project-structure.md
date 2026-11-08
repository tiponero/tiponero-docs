---
sidebar_position: 3
title: Project Structure
---

# Project Structure

The codebase follows the standard Go project layout. All application code is under `internal/`, so external packages cannot import it.

```
tiponero-core/
├── cmd/tiponero/
│   └── main.go                     # Entry point: config, DB init, router, server
│
├── internal/
│   ├── config/
│   │   └── config.go               # Typed env var loading (PORT, BASE_URL, etc.)
│   │
│   ├── crypto/
│   │   └── aes.go                  # AES-256-GCM encrypt/decrypt, HKDF key derivation
│   │
│   ├── database/
│   │   ├── models.go               # All struct types + enums (User, Widget, Transaction, etc.)
│   │   ├── sqlite.go               # DB initialization, schema creation, WAL mode
│   │   ├── user.go                 # User CRUD operations
│   │   ├── wallet.go               # Wallet config get/save (with AES encryption)
│   │   ├── widget.go               # Widget CRUD operations
│   │   ├── transaction.go          # Transaction CRUD, stats queries, expiration
│   │   └── apikey.go               # API key CRUD, prefix-based lookup, touch
│   │
│   ├── monero/
│   │   ├── client.go               # JSON-RPC 2.0 client (create_address, get_transfers, etc.)
│   │   ├── holder.go               # Thread-safe hot-swappable client (sync.RWMutex)
│   │   └── types.go                # RPC request/response type definitions
│   │
│   ├── auth/
│   │   ├── auth.go                 # Session service (gorilla/sessions CookieStore)
│   │   ├── middleware.go           # RequireAuth middleware (session cookie)
│   │   └── api.go                  # RequireAPIKey middleware (Bearer token, bcrypt)
│   │
│   ├── services/
│   │   ├── monitor.go              # Payment monitor goroutine (30s polling interval)
│   │   ├── price.go                # CoinGecko XMR/fiat price service (5min cache)
│   │   ├── qrcode.go               # QR code PNG generation (monero: URI)
│   │   ├── badge.go                # SVG badge generation (widget stats, QR)
│   │   └── totp.go                 # TOTP secret generation, QR, validation
│   │
│   ├── handlers/
│   │   ├── routes.go               # Chi router wiring (all routes defined here)
│   │   ├── admin.go                # Admin UI handlers (HTML responses)
│   │   ├── api.go                  # REST API handlers (JSON responses)
│   │   ├── widget.go               # Public widget handlers
│   │   └── toast.go                # HX-Trigger toast notification helper
│   │
│   └── views/
│       ├── admin/                   # templ templates: dashboard, login, settings, etc.
│       ├── widget/                  # templ templates: home, payment, status, error
│       ├── layouts/                 # Page shell templates (admin, auth, widget)
│       └── components/              # Shared UI components (stats card, table, etc.)
│
├── static/
│   ├── embed.go                     # go:embed directive for all static assets
│   ├── css/
│   │   ├── input.css                # Tailwind CSS source (directives + custom styles)
│   │   └── output.css               # Compiled CSS (generated at build time)
│   ├── js/
│   │   ├── htmx.min.js             # Vendored HTMX 2.x
│   │   └── alpine.min.js           # Vendored Alpine.js 3.x
│   └── images/
│       ├── logo.svg                 # Tiponero mark, transparent
│       ├── login-lockup.svg         # Mark and wordmark for the login page
│       └── favicon/                 # Favicon set
│
├── tools/
│   └── go.mod                       # Separate module for dev tools (templ, air, golangci-lint)
│
├── Dockerfile                       # Multi-stage: golang:1.26-bookworm → distroless
├── flake.nix                        # Nix build + devShell (4 platforms)
├── Makefile                         # generate, css, build, dev, lint, clean
├── .env.example                     # Environment variable template
├── .air.toml                        # Air hot-reload configuration
├── go.mod                           # Go module definition
├── package.json                     # Tailwind CSS devDependency
└── tailwind.config.mjs              # Tailwind theme (Monero orange, MD3 tokens)
```

## Key Entry Points

### `cmd/tiponero/main.go`

The `main()` function starts the process in this order:

1. Load the `.env` file and parse the configuration
2. Derive the session key and the wallet encryption key with HKDF from `ENCRYPTION_KEY`
3. Open the SQLite database, run the schema migrations, and create the default `admin/admin` user if none exists
4. Load the wallet config from the database and decrypt the stored passwords, then create the Monero RPC client
5. Wire all HTTP handlers with `handlers.NewRouter()`
6. Start the `PaymentMonitor` background goroutine
7. Start the HTTP server on `PORT`
8. Wait for `SIGINT` or `SIGTERM`, then shut down with a 10-second timeout

### `internal/handlers/routes.go`

One file defines all HTTP routes. The routes fall into four groups:

- **Root:** `/` redirect, `/health`, `/static/*`
- **Admin:** `/admin/*` (session-cookie auth with the `RequireAuth` middleware)
- **API:** `/api/v1/*` (Bearer token auth with the `RequireAPIKey` middleware)
- **Widget:** `/widget/*` (no auth, public-facing)

### `internal/services/monitor.go`

The `PaymentMonitor` runs in a background goroutine with a 30-second ticker:

1. Expire old transactions: mark `pending` transactions past `expires_at` as `expired`
2. Call `get_transfers` on `monero-wallet-rpc` for `in`, `pending`, and `pool` transfers
3. Index all transfers by subaddress
4. Update transaction status: `pending` → `mempool` → `confirming` → `confirmed`
5. Match on the confirmation count against the `RequiredConfirmations` config value
