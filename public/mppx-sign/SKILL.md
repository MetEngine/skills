---
name: mppx-sign
description: Sign a payment challenge and output the Authorization header. Run `mppx sign --help` for usage details.
requires_bin: mppx
command: mppx sign
---

# mppx sign

Sign a payment challenge and output the Authorization header

## Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--account` | `string` |  | Account name (env: MPPX_ACCOUNT) |
| `--challenge` | `string` |  | WWW-Authenticate challenge value |
| `--config` | `string` |  | Path to config file |
| `--dryRun` | `boolean` |  | Validate and parse the challenge without signing |
| `--methodOpt` | `array` |  | Method-specific option (key=value, repeatable) |
| `--rpcUrl` | `string` |  | RPC endpoint, defaults to public RPC for chain (env: MPPX_RPC_URL) |
