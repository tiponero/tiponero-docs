---
sidebar_position: 7
title: Reverse Proxy
---

# Reverse Proxy & TLS

Tiponero listens on a local HTTP port and should be placed behind a reverse proxy for TLS termination, compression, and access control. This guide covers nginx, Caddy, and Cloudflare Tunnel.

## nginx

```nginx title="/etc/nginx/sites-available/tiponero"
server {
    listen 80;
    server_name pay.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pay.example.com;

    ssl_certificate     /etc/letsencrypt/live/pay.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/pay.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Obtain a certificate with Certbot:

```bash
sudo certbot --nginx -d pay.example.com
```

## Caddy

Caddy handles TLS automatically via Let's Encrypt:

```text title="Caddyfile"
pay.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

Start Caddy:

```bash
caddy run --config Caddyfile
```

That's it. Caddy provisions and renews TLS certificates automatically.

## Cloudflare Tunnel

Cloudflare Tunnel provides a zero-inbound-port deployment. The tunnel daemon (`cloudflared`) establishes an outbound connection to Cloudflare's edge network. No ports need to be opened in your firewall.

### Manual Setup

1. [Install cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)
2. Authenticate: `cloudflared tunnel login`
3. Create a tunnel: `cloudflared tunnel create tiponero`
4. Configure routing:

```yaml title="~/.cloudflared/config.yml"
tunnel: <your-tunnel-id>
credentials-file: /home/cloudflared/.cloudflared/<your-tunnel-id>.json

ingress:
  - hostname: pay.example.com
    service: http://127.0.0.1:8080
  - service: http_status:404
```

5. Route DNS: `cloudflared tunnel route dns tiponero pay.example.com`
6. Run: `cloudflared tunnel run tiponero`

### NixOS Module

For NixOS deployments, use the built-in `cloudflared` module:

```nix title="cloudflared.nix"
{ config, pkgs, ... }:
{
  # Encrypted tunnel credentials
  age.secrets.cloudflare-tunnel = {
    file = ./secrets/cloudflare-tunnel.age;
    owner = "cloudflared";
    mode = "0400";
  };

  # Dedicated system user
  users.users.cloudflared = {
    isSystemUser = true;
    group = "cloudflared";
    home = "/var/lib/cloudflared";
    createHome = true;
  };
  users.groups.cloudflared = {};

  # Tunnel configuration
  services.cloudflared = {
    enable = true;
    tunnels."<your-tunnel-id>" = {
      credentialsFile = config.age.secrets.cloudflare-tunnel.path;
      ingress = {
        "pay.example.com" = "http://127.0.0.1:3001";
      };
      default = "http_status:404";
    };
  };
}
```

This approach means:
- **No inbound firewall ports** needed for web traffic
- TLS is handled by Cloudflare's edge
- Your server's IP address is not exposed
- DDoS protection is included

### Important: BASE_URL

When using a reverse proxy or tunnel, set `BASE_URL` to your public HTTPS URL:

```bash
BASE_URL=https://pay.example.com
```

This ensures:
- Session cookies are marked `Secure` (HTTPS-only)
- Widget URLs and QR codes use the correct public URL
- Redirects work correctly
