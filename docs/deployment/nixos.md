---
sidebar_position: 6
title: NixOS
---

# NixOS Deployment

Tiponero provides a Nix flake that builds the application for `x86_64-linux`, `aarch64-linux`, `x86_64-darwin`, and `aarch64-darwin`. This guide shows how to deploy Tiponero as a NixOS service using the flake, following a production-tested pattern.

The configuration examples below are based on a real production deployment running on a NixOS Proxmox LXC container with Cloudflare Tunnel for public access.

## Flake Setup

Add Tiponero as an input to your NixOS flake:

```nix title="flake.nix"
{
  description = "My NixOS server";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";

    agenix = {
      url = "github:ryantm/agenix";
      inputs.nixpkgs.follows = "nixpkgs";
    };

    tiponero = {
      url = "github:tiponero/tiponero-core";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, agenix, tiponero }: {
    nixosConfigurations.my-server = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      specialArgs = { inherit tiponero; };
      modules = [
        agenix.nixosModules.default
        ./configuration.nix
      ];
    };
  };
}
```

Key points:
- `tiponero` input points to the GitHub repo and follows your nixpkgs to avoid duplicate builds
- `specialArgs` passes the `tiponero` flake to all NixOS modules
- [agenix](https://github.com/ryantm/agenix) handles encrypted secrets (recommended but optional)

## Tiponero Service Module

Create `tiponero.nix` in your NixOS configuration:

```nix title="tiponero.nix"
{ config, pkgs, tiponero, ... }:

let
  tiponeroPkg = tiponero.packages.${pkgs.system}.default;
in {
  # Secret environment variables (ENCRYPTION_KEY, etc.)
  age.secrets.tiponero = {
    file = ./secrets/tiponero.age;
    owner = "tiponero";
    mode = "0400";
  };

  # Dedicated system user
  users.users.tiponero = {
    isSystemUser = true;
    group = "tiponero";
    home = "/var/lib/tiponero";
    createHome = true;
  };
  users.groups.tiponero = {};

  # Systemd service
  systemd.services.tiponero = {
    description = "Tiponero - Monero payment engine";
    wantedBy = [ "multi-user.target" ];
    after = [ "network.target" "monero-wallet-rpc.service" ];
    wants = [ "monero-wallet-rpc.service" ];
    serviceConfig = {
      Type = "simple";
      User = "tiponero";
      Group = "tiponero";
      WorkingDirectory = "/var/lib/tiponero";
      EnvironmentFile = config.age.secrets.tiponero.path;
      ExecStart = "${tiponeroPkg}/bin/tiponero";
      Restart = "on-failure";
      RestartSec = 10;
    };
    environment = {
      PORT = "3001";
      DATABASE_PATH = "/var/lib/tiponero/tiponero.db";
      MONERO_RPC_URL = "http://127.0.0.1:18083/json_rpc";
      FIAT_CURRENCY = "USD";
      REQUIRED_CONFIRMATIONS = "5";
      BASE_URL = "https://pay.example.com";
    };
  };
}
```

The `EnvironmentFile` injects sensitive variables (like `ENCRYPTION_KEY`) that are decrypted at boot by agenix. Non-secret variables are set directly in `environment`.

### Secret File Format

The encrypted `tiponero.age` file should contain:

```bash
ENCRYPTION_KEY=your-random-32-character-key-here
```

Encrypt it with agenix:

```bash
agenix -e secrets/tiponero.age
```

## Monero Backend Module

Create `monero.nix` for the full node and wallet RPC:

```nix title="monero.nix"
{ config, pkgs, ... }:
{
  # Encrypted RPC credentials
  age.secrets.monero-rpc = {
    file = ./secrets/monero-rpc.age;
    mode = "0440";
    group = "monero";
  };

  # Monero full node (pruned)
  services.monero = {
    enable = true;
    rpc = {
      address = "127.0.0.1";
      user = "$MONERO_RPC_USER";
      password = "$MONERO_RPC_PASSWORD";
    };
    extraConfig = ''
      db-sync-mode=safe
      prune-blockchain=1
    '';
    environmentFile = config.age.secrets.monero-rpc.path;
  };

  # Wallet RPC service
  systemd.services.monero-wallet-rpc = {
    description = "Monero Wallet RPC";
    after = [ "monero.service" ];
    wants = [ "monero.service" ];
    wantedBy = [ "multi-user.target" ];
    serviceConfig = {
      User = "monero";
      WorkingDirectory = "/var/lib/monero";
      EnvironmentFile = config.age.secrets.monero-rpc.path;
      ExecStart = pkgs.writeShellScript "monero-wallet-rpc-start" ''
        exec ${pkgs.monero-cli}/bin/monero-wallet-rpc \
          --rpc-bind-ip=127.0.0.1 \
          --rpc-bind-port=18083 \
          --rpc-login="$MONERO_RPC_USER:$MONERO_RPC_PASSWORD" \
          --daemon-address=127.0.0.1:18081 \
          --daemon-login="$MONERO_RPC_USER:$MONERO_RPC_PASSWORD" \
          --wallet-file=/var/lib/monero/wallet \
          --password-file=/var/lib/monero/wallet.passwd \
          --non-interactive
      '';
      Restart = "on-failure";
    };
  };
}
```

The `monero-rpc.age` secret file should contain:

```bash
MONERO_RPC_USER=rpc_user
MONERO_RPC_PASSWORD=rpc_password
```

## Import Modules

In your `configuration.nix`, import both modules:

```nix title="configuration.nix"
{ config, pkgs, ... }:
{
  imports = [
    ./hardware.nix
    ./networking.nix
    ./tiponero.nix
    ./monero.nix
    # ./cloudflared.nix   # Optional: see Reverse Proxy guide
  ];

  # ... rest of your system config
}
```

## Secrets Setup (agenix)

Define which secrets exist and who can decrypt them:

```nix title="secrets/secrets.nix"
let
  server = "age1..."; # Your server's SSH host key as an age public key
in {
  "tiponero.age".publicKeys = [ server ];
  "monero-rpc.age".publicKeys = [ server ];
}
```

Get your server's age public key:

```bash
ssh-keyscan your-server | ssh-to-age
```

Create and encrypt secrets:

```bash
agenix -e secrets/tiponero.age
agenix -e secrets/monero-rpc.age
```

## Deploying

Apply the configuration:

```bash
# From the server itself
sudo nixos-rebuild switch --flake .#my-server

# Or remotely
nixos-rebuild switch --flake .#my-server --target-host root@your-server
```

## Firewall

Open only the ports you need:

```nix
networking.firewall.allowedTCPPorts = [
  22     # SSH
  18080  # Monero P2P (required for blockchain sync)
];
```

Tiponero runs on `127.0.0.1:3001` and does not need a firewall rule: it is accessed through a reverse proxy or Cloudflare Tunnel.

## Updating Tiponero

Update the flake input and rebuild:

```bash
nix flake update tiponero
sudo nixos-rebuild switch --flake .#my-server
```

The systemd service will restart automatically with the new binary.

## Proxmox LXC Notes

If running NixOS in a Proxmox LXC container, your `hardware.nix` should include:

```nix title="hardware.nix"
{
  boot.isContainer = true;
  boot.loader.grub.enable = false;

  # Disable debugfs mount (not available in unprivileged LXC)
  boot.specialFileSystems."/sys/kernel/debug".options = [ "noauto" ];
}
```

This configuration works with standard unprivileged LXC containers: no special privileges required.

## Build Errors After a nixpkgs Update

The flake takes the templ CLI from nixpkgs, so a nixpkgs update can move the CLI ahead of the Go module in `go.mod`. The build then stops with `undefined: templ.ResolveAttributeValue`.

The [Nix Packaging](/docs/development/nix-packaging) page gives the test command for a specific nixpkgs revision and the update procedure. The [templ Toolchain](/docs/development/templ-toolchain) page gives the version bump steps.
