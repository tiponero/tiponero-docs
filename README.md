# Tiponero Docs

Documentation for [Tiponero](https://github.com/tiponero/tiponero-core), a self-hosted Monero payment engine. Built with [Docusaurus 3](https://docusaurus.io/).

**Live site:** [docs.tiponero.org](https://docs.tiponero.org)

## Local Development

```bash
npm install
npm start
```

This starts a local dev server at `http://localhost:3000`. Most changes are reflected live without restarting.

## Build

```bash
npm run build
```

Generates static output into the `build` directory.

## Project Structure

```
docs/
  intro.md                  # Landing page
  getting-started/          # Overview, Quick Start
  configuration/            # Environment variables
  architecture/             # Tech stack, project structure, database schema, payment lifecycle
  internals/                # Deep reference: system architecture, data model, flows
  api/                      # REST API reference (auth, widgets, transactions, wallet, stats, keys)
  widgets/                  # Embedding, badge, donation mode, payment mode
  admin/                    # Admin panel usage
  deployment/               # Wallet RPC setup, systemd, stagenet, Makefile, Docker, NixOS, reverse proxy, security
  development/              # Maintainer notes: templ toolchain, Nix packaging
src/
  css/custom.css            # Brand colors and fonts
  pages/index.tsx           # Redirect to /docs/intro
static/
  img/                      # Logo and favicon
```

## License

MIT
