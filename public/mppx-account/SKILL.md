---
name: mppx-account
description: Manage mppx accounts (create, default, delete, fund, list, view). Run `mppx account --help` for usage details.
requires_bin: mppx
command: mppx account
---

# mppx account create

Create new account

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name (env: MPPX_ACCOUNT) |
| `--rpcUrl` | `string` |  | RPC endpoint (env: MPPX_RPC_URL) |

---

# mppx account default

Set default account

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name |

---

# mppx account delete

Delete account

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name |
| `--yes` | `boolean` |  | DANGER!! Skip confirmation prompts |

---

# mppx account fund

Fund account with testnet tokens

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name (env: MPPX_ACCOUNT) |
| `--rpcUrl` | `string` |  | RPC endpoint (env: MPPX_RPC_URL) |

---

# mppx account list

List all accounts

---

# mppx account view

View account address

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name (env: MPPX_ACCOUNT) |
| `--rpcUrl` | `string` |  | RPC endpoint (env: MPPX_RPC_URL) |
