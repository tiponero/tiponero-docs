---
sidebar_position: 9
title: Tor
---

# Serve Tiponero over Tor

Tiponero works as a Tor onion service. The browser never contacts a third party, and the server can reach its wallet over loopback. This page shows the setup and the leaks you must close.

## What Tiponero does and does not request

| Request | Default | Can it be removed? |
|---|---|---|
| HTML, CSS, JS, fonts | From your own server | Not applicable |
| XMR price | CoinGecko, then CoinMarketCap | Yes, route it through Tor or accept the leak |
| Wallet | `monero-wallet-rpc` over your network | Keep it on loopback |
| Analytics or fonts from a CDN | None | Not applicable |

Tiponero serves its fonts and icons from `/static`. The admin panel and the payment widgets make no request to Google, Cloudflare, or any other third party. That is deliberate: a donor who opens a widget over Tor must not reveal the visit to anyone.

## 1. Run a hidden service

Add this to `/etc/tor/torrc`:

```text
HiddenServiceDir /var/lib/tor/tiponero/
HiddenServicePort 80 127.0.0.1:8080
```

Restart Tor and read the address:

```bash
sudo systemctl restart tor
sudo cat /var/lib/tor/tiponero/hostname
```

The file contains an address like `abcdefghijklmnop.onion`.

## 2. Set the public base URL

Tiponero builds payment links, badge links, and webhook URLs from `BASE_URL`. Set it to the onion address:

```bash
BASE_URL=http://abcdefghijklmnop.onion
```

A plain HTTP onion address is correct. Tor provides the encryption, and a `.onion` certificate cannot exist.

## 3. Route the price request through Tor

The price service contacts CoinGecko. Without a proxy, that request comes from your server's IP at the moment of a payment. Two options:

**Route it through Tor.** Go honors `HTTPS_PROXY`, including a SOCKS5 URL:

```bash
HTTPS_PROXY=socks5://127.0.0.1:9050
```

Set `NO_PROXY=127.0.0.1,localhost` so the wallet RPC call stays direct.

**Or disable fiat pricing.** Leave `FIAT_CURRENCY` unset and do not set a fiat price on a widget. Tiponero then never contacts a price provider. Donation widgets with XMR-only presets work without it.

## 4. Keep the wallet local

`monero-wallet-rpc` holds the keys. Do not publish its port. Bind it to loopback:

```bash
monero-wallet-rpc --rpc-bind-ip=127.0.0.1 --rpc-bind-port=18088 --wallet-dir=/var/lib/monero/wallets
```

If the wallet runs on another host, use a private network or a WireGuard tunnel. Never expose port 18088 to the internet.

## 5. Restrict the admin panel

The admin panel has a password, optional TOTP, CSRF protection, and rate limiting. That is not a reason to publish it on the same address as the widgets.

Two safe options:

- Use a second hidden service for the admin panel and point it at a reverse proxy that allows only `/admin` and `/api`.
- Keep the admin panel on a LAN address and publish only `/widget/*`, `/pay/*`, and `/static/*` through Tor.

## 6. Check for leaks

From the server, confirm that no third-party host appears in the rendered HTML:

```bash
curl -s http://127.0.0.1:8080/admin/login | grep -E "https?://" | grep -v "127.0.0.1"
```

The command should print nothing. If it prints a font or script URL, a template still loads a remote asset.

## Limitations

- Tor adds latency. A payment page may take a few seconds to load. The status poll runs every 3 seconds and is not affected by the initial load time.
- Webhook deliveries go to the URL you configure. A `.onion` receiver needs Tor on the Tiponero host. The delivery client uses the same `HTTPS_PROXY` setting.
- A hidden service does not hide the fact that a payment happened. It hides who runs the merchant site. On-chain privacy comes from Monero, not from Tor.
