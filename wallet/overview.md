---
skill: wallet/overview
requires: [core/security]
triggers: ["wallet.*setup", "solana.*wallet", "evm.*wallet", "wallet.*provider", "phantom", "turnkey", "privy", "mppx.*setup"]
level: concept
---

# Wallet Provider Overview

MetEngine supports two payment protocols: **MPP** (Tempo EVM chain) and **x402** (Solana Mainnet). Both protocols are active simultaneously -- the server accepts either. Both use the same settle-after-execute guarantee. MetEngine is mainnet-only; all requests cost real USDC. Recommended initial funding: $5-10 USDC (covers 50-500+ requests).

**MPP requires Tempo mainnet (chain 4217).** The `mppx` CLI defaults to testnet -- you must always pass `-r https://rpc.presto.tempo.xyz` to every `mppx` command. The env var `MPPX_RPC_URL` only works for account subcommands; for payment requests the tempo plugin reads `RPC_URL` instead, so set both to be safe: `export RPC_URL=https://rpc.presto.tempo.xyz MPPX_RPC_URL=https://rpc.presto.tempo.xyz`.

## Protocol Comparison

| Feature | MPP (Tempo EVM) | x402 (Solana) |
|---------|-----------------|---------------|
| Chain | Tempo (EVM) | Solana Mainnet |
| Currency | USDC (TIP-20) | USDC (SPL) |
| Gas token | Tempo native | SOL |
| CLI tool | `npx mppx <url>` | None (SDK only) |
| SDK | `mppx/client` + `viem` | `@x402/core` + `@x402/svm` |
| Wallet setup | `npx mppx account create` then always pass `-r https://rpc.presto.tempo.xyz` | Solana CLI or Phantom export |
| Headers (402) | `WWW-Authenticate: Payment` | `PAYMENT-REQUIRED` / `X-PAYMENT-REQUIRED` |
| Headers (200) | `Payment-Receipt` | `PAYMENT-RESPONSE` / `X-PAYMENT-RESPONSE` |
| Discovery | `GET /.well-known/mpp` | `GET /api/v1/pricing` |

## Decision Guide

**Use MPP when:**
- Want the fastest setup (`npx mppx account create` + fund)
- Running CLI agents or scripts (automatic 402 handling via `mppx`)
- Building EVM-native applications
- Want machine-readable discovery via `/.well-known/mpp`

**Use x402 when:**
- Already have a funded Solana wallet
- Building Solana-native applications
- Using Phantom, Turnkey, or Privy wallet providers
- Need HSM-backed signing (Turnkey) or embedded wallets (Privy)

## MPP Quick Start

```bash
# 1. Set BOTH env vars (mppx has a bug: different commands read different vars)
export RPC_URL=https://rpc.presto.tempo.xyz
export MPPX_RPC_URL=https://rpc.presto.tempo.xyz

# 2. Create a wallet (one-time, stores locally)
npx mppx account create

# 3. View your wallet address
npx mppx account view

# 4. Fund wallet with USDC.e on Tempo mainnet (bridge via https://bridge.tempo.xyz)

# 5. Make requests -- ALWAYS pass -r to guarantee mainnet
npx mppx -r https://rpc.presto.tempo.xyz https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5

# Free endpoints (no wallet needed)
curl -sS https://agent.metengine.xyz/health
curl -sS https://agent.metengine.xyz/.well-known/mpp
curl -sS https://agent.metengine.xyz/api/v1/pricing
```

See `wallet/mpp-signing-flows` for programmatic TypeScript usage.

## x402 Provider Comparison

| Feature | Phantom | Turnkey | Privy |
|---------|---------|---------|-------|
| Key storage | Local file / browser extension | HSM-backed cloud | Embedded wallet (social login) |
| Signing | Local (fastest) | API call to Turnkey | Server-side via Privy SDK |
| Setup complexity | Low | Medium | Medium |
| Security model | User manages keys | Enterprise key management | Delegated custody |
| Best for | Development, CLI agents | Production, policy-controlled | Consumer apps, social onboarding |
| Latency per sign | <10ms | 100-300ms | 50-200ms |
| Key recovery | Manual (backup seed) | Organization recovery | Social recovery / email |

**Use Phantom/local keypair when:**
- Running a CLI agent locally with Solana
- Fast iteration and development
- Single-user setup, no policy requirements

**Use Turnkey when:**
- Production deployment with multiple agents
- Need policy engine (spending limits, allowlists)
- Enterprise compliance requirements

**Use Privy when:**
- Building a consumer-facing app with MetEngine integration
- Users authenticate via social login (Google, email, etc.)
- Want embedded wallets without users managing keys

## Prerequisites

### MPP (Tempo EVM)

| Detail | Value |
|--------|-------|
| Chain ID | 4217 (mainnet only -- do NOT use testnet 42431) |
| RPC | `https://rpc.presto.tempo.xyz` (mainnet) |
| Native gas | USD (Tempo uses USD as native gas, not ETH) |
| USDC.e contract | `0x20C000000000000000000000b9537d11c60E8b50` |
| Explorer | `https://explore.tempo.xyz` |
| Gas per tx | Negligible (fractions of a cent) |

1. EVM wallet with:
   - USDC.e on Tempo chain for API payments ($0.01-$0.20 per request)
   - USD (native gas) for transaction fees (negligible)
2. NPM packages: `mppx`, `viem`
3. Bun or Node.js runtime
4. See `wallet/mpp-signing-flows` for implementation details

**Funding your wallet:**
- Bridge USDC from Ethereum/Arbitrum/Base to Tempo via the Tempo bridge at `https://bridge.tempo.xyz`
- Or acquire USDC.e directly on Tempo via a DEX
- `npx mppx account view` shows your wallet address to send funds to

### x402 (Solana)
1. Solana wallet with:
   - SOL for transaction fees (~0.001 SOL per x402 payment)
   - USDC for API payments ($0.01-$0.20 per request)
2. NPM packages: `@x402/core`, `@x402/svm`, `@solana/kit`
3. Bun or Node.js runtime
4. See `wallet/signing-flows` for implementation details

## Wallet Security Reminders

- NEVER log, print, or store private keys
- ONLY store file paths and public addresses in memory files
- Pipe keypair bytes directly into the signer -- never into an intermediate variable that gets logged
- See `core/security` for full wallet security rules
