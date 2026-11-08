---
sidebar_position: 2
title: Running as systemd Services
---

# Running as systemd Services

For production deployments, run `monero-wallet-rpc` (and optionally `monerod`) as systemd services. The services then start automatically and restart after a failure.

## Remote Node Setup

If you use a [remote node](/docs/deployment/prerequisites#option-a-remote-node-recommended), you need one service file for `monero-wallet-rpc`:

```ini title="/etc/systemd/system/monero-wallet-rpc.service"
[Unit]
Description=Monero Wallet RPC
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=monero
WorkingDirectory=/var/lib/monero
ExecStart=/usr/bin/monero-wallet-rpc \
  --rpc-bind-ip=127.0.0.1 \
  --rpc-bind-port=18083 \
  --rpc-login=rpc_user:rpc_password \
  --daemon-address=node.moneroworld.com:18089 \
  --wallet-file=/var/lib/monero/wallet \
  --password-file=/var/lib/monero/wallet.passwd \
  --non-interactive
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now monero-wallet-rpc
```

## Local Node Setup

If you run your own [local node](/docs/deployment/prerequisites#option-b-local-node), you need two services. The wallet RPC depends on `monerod`:

### monerod.service

```ini title="/etc/systemd/system/monerod.service"
[Unit]
Description=Monero Full Node
After=network.target

[Service]
Type=simple
User=monero
WorkingDirectory=/var/lib/monero
ExecStart=/usr/bin/monerod \
  --prune-blockchain \
  --db-sync-mode=safe \
  --rpc-bind-ip=127.0.0.1 \
  --rpc-bind-port=18081 \
  --rpc-login=rpc_user:rpc_password \
  --non-interactive
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### monero-wallet-rpc.service

```ini title="/etc/systemd/system/monero-wallet-rpc.service"
[Unit]
Description=Monero Wallet RPC
After=monerod.service
Wants=monerod.service

[Service]
Type=simple
User=monero
WorkingDirectory=/var/lib/monero
ExecStart=/usr/bin/monero-wallet-rpc \
  --rpc-bind-ip=127.0.0.1 \
  --rpc-bind-port=18083 \
  --rpc-login=rpc_user:rpc_password \
  --daemon-address=127.0.0.1:18081 \
  --daemon-login=rpc_user:rpc_password \
  --wallet-file=/var/lib/monero/wallet \
  --password-file=/var/lib/monero/wallet.passwd \
  --non-interactive
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now monerod
sudo systemctl enable --now monero-wallet-rpc
```

## Network Requirements

| Port | Service | Direction | Notes |
|---|---|---|---|
| 18080 | monerod P2P | Inbound | Local node only. Open in firewall for blockchain sync. |
| 18081 | monerod RPC | Local only | Local node only. Used by `monero-wallet-rpc`. Do not expose. |
| 18083 | monero-wallet-rpc | Local only | Used by Tiponero. Do not expose. |

:::tip
With a remote node, you do not open inbound ports. Only port 18083 matters, and it is bound to localhost. With a local node, the internet must reach port 18080 (P2P) for the blockchain sync.
:::
