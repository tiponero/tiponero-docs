---
sidebar_position: 3
title: Testing on Stagenet
---

# Testing on Stagenet

Monero's **stagenet** is a test network that mirrors mainnet behavior without using real XMR. It's ideal for testing your Tiponero integration before going live.

- No real XMR is spent or received
- Same protocol, block times, and confirmation logic as mainnet
- Free test XMR available from a faucet
- Separate blockchain (much smaller, faster initial sync)

## Stagenet Ports

Stagenet uses different default ports to avoid conflicts with a mainnet node:

| Service | Mainnet Port | Stagenet Port |
|---|---|---|
| monerod P2P | 18080 | 38080 |
| monerod RPC | 18081 | 38081 |
| monero-wallet-rpc | 18083 | 38083 |

## Start a Stagenet Daemon

```bash
monerod \
  --stagenet \
  --rpc-bind-ip=127.0.0.1 \
  --rpc-bind-port=38081 \
  --rpc-login=rpc_user:rpc_password \
  --detach
```

## Create a Stagenet Wallet

```bash
monero-wallet-cli --stagenet --generate-new-wallet stagenet_wallet
```

Save the seed phrase, then exit the wallet CLI. For testing, you can use a full wallet (not view-only) since no real funds are at risk.

## Get Test XMR

Request free stagenet XMR from the faucet:

**[https://stagenet-faucet.xmr-tw.org](https://stagenet-faucet.xmr-tw.org)**

Paste your stagenet wallet address and you will receive test XMR within a few minutes.

## Start monero-wallet-rpc

```bash
monero-wallet-rpc \
  --stagenet \
  --rpc-bind-ip=127.0.0.1 \
  --rpc-bind-port=38083 \
  --rpc-login=rpc_user:rpc_password \
  --daemon-address=127.0.0.1:38081 \
  --daemon-login=rpc_user:rpc_password \
  --wallet-file=/path/to/stagenet_wallet \
  --password-file=/path/to/stagenet_wallet.passwd \
  --non-interactive
```

## Configure Tiponero

Set a low confirmation count for fast turnaround:

```bash title=".env"
PORT=8080
BASE_URL=http://localhost:8080
DATABASE_PATH=./tiponero.db
ENCRYPTION_KEY=change-me-to-a-random-32char-key
CONFIRMATIONS=1
```

Then configure the wallet connection in **Settings > Wallet**:

| Field | Value |
|---|---|
| RPC URL | `http://127.0.0.1:38083/json_rpc` |
| RPC Username | `rpc_user` |
| RPC Password | `rpc_password` |
| Wallet File | `stagenet_wallet` |
| Wallet Password | your wallet password |

You can now create widgets and send test XMR from the faucet to verify the full payment lifecycle.
