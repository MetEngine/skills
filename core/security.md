---
skill: core/security
requires: []
triggers: ["security", "address", "wallet.*key", "rate.*limit", "display.*rule"]
level: concept
---

# Security and Display Rules

Required by every skill in the graph. Load this file first on every task.

## Non-Negotiable Display Rule

Never truncate or trim wallet addresses, contract addresses, condition IDs, token IDs, pool addresses, position addresses, or transaction hashes. Always return full values.

## Address Conventions

| Platform | Format | Case Sensitivity |
|----------|--------|-----------------|
| Polymarket | 0x hex (Polygon) | Must be lowercase |
| Hyperliquid | 0x hex | Case-insensitive |
| Meteora | Base58 (Solana pubkey) | Case-sensitive |

Wallet addresses for Polymarket API calls MUST be lowercase. Meteora addresses are base58 and case-sensitive. Hyperliquid accepts any case but normalizes to lowercase.

## Wallet Security Rules

- NEVER read, log, print, or display the contents of any keypair file.
- NEVER store private keys in memory files, chat, logs, or any persistent storage.
- ONLY store the file path and the public address.
- Load the keypair at runtime using `Bun.file(path).text()` or `fs.readFileSync(path)` -- pipe directly into the signer, never into a variable that gets logged.
- NEVER pass private key material as a function argument that may be serialized.

## Payment Security

- Payment IS authentication. No API keys, no accounts.
- Settlement occurs ONLY on successful `2xx` responses (settle-after-execute).
- If a query fails (timeout, server error), no payment is settled. The agent keeps funds.
- Two protocols supported: **MPP** (Tempo EVM) and **x402** (Solana).

### MPP (Tempo EVM)

- Uses standard HTTP headers: `WWW-Authenticate: Payment` (challenge on 402), `Authorization: Payment` (credential), `Payment-Receipt` (settlement proof on 200).
- Pull mode only: client signs a TIP-20 USDC transfer but does NOT broadcast. Server broadcasts after successful query.
- CLI: `npx mppx <url>` handles the 402 handshake automatically.
- SDK: `createClient()` from `mppx/client` with a viem `WalletClient`.
- Discovery: `GET /.well-known/mpp` returns full endpoint catalog and pricing.

### x402 (Solana)

- `PAYMENT-REQUIRED` / `X-PAYMENT-REQUIRED` headers contain payment requirements on `402`.
- `PAYMENT-RESPONSE` / `X-PAYMENT-RESPONSE` headers contain settlement proof on `200`.
- Uses `@x402/core` + `@x402/svm` with a Solana wallet signer.

### Settlement Edge Cases

| Header | Meaning |
|--------|---------|
| `X-Payment-Settlement: failed` (x402) | Query succeeded but settlement failed. Data returned, no charge. Rare. |
| `Payment-Receipt` absent on 200 (MPP) | Settlement not attempted (should not happen in normal flow). |
| No `PAYMENT-RESPONSE` on 200 (x402) | Settlement not attempted (should not happen in normal flow). |

## Rate Limits

| Metric | Value |
|--------|-------|
| p50 latency | 800ms |
| p95 latency | 3s |
| p99 latency | 8s |
| Handler timeout | 60s (no charge on timeout) |
| Max concurrent paid requests | 50 |
| Payment verification timeout | 5s |
| Rate limit (unpaid 402 requests) | IP-based, generous |

## Pricing Bounds

- Floor: $0.01 USDC per request
- Ceiling: $0.20 USDC per request
- All prices in USDC (Tempo EVM for MPP, Solana Mainnet for x402)

### Tier Base Costs

| Tier | Base (USDC) | Description |
|------|------------|-------------|
| light | $0.01 | Simple lookups, searches, feeds |
| medium | $0.02 | Aggregated analytics, trending |
| heavy | $0.05 | Computed intelligence, leaderboards, scoring |
| whale | $0.08 | Multi-entity comparisons, complex scans |

### Key Modifiers

- **Timeframe**: 1h=0.5x, 24h=1.0x, 7d=2.0x, 30d=3.0x, 90d=4.0x, 365d=5.0x
- **Limit**: `price *= max(1, requested_limit / default_limit)`
- **Filter discounts**: category=0.7x, condition_id/pool_address=0.5x, smart_money_only=0.7x

Machine-readable pricing: `GET /api/v1/pricing` (free).

## Session Memory

Check `~/.claude/agents/metengine-memory.md` before any API call. If it exists, read it first for wallet config, package status, working bootstrap code, endpoint history, and fallbacks learned.

### Memory Update Rules

1. After first successful setup -- record wallet path, public address, installed packages, working bootstrap.
2. After every API call -- append to Endpoint History (keep last 10 rows).
3. When a fallback is used -- record in Fallbacks Learned.
4. When a new quirk is discovered -- record in Quirks Encountered.
5. At session end -- update `Last Updated` timestamp.

## Data Freshness

| Platform | Freshness |
|----------|-----------|
| Polymarket trades | Sub-minute (continuous indexing) |
| Polymarket wallet scores | Daily recompute |
| Hyperliquid trades | Sub-minute |
| Hyperliquid smart scores | Continuous (formula-based) |
| Meteora events | Sub-minute |
| Meteora LP scores | Daily recompute |

## Limitations

1. Read-only analytics. No trade execution.
2. Request/response only. No WebSocket streaming.
3. No historical backfill on demand.
4. No unrealized PnL for Hyperliquid (closed/realized only).
5. No mark-to-market for Meteora positions.
6. No Polymarket order book data (trades only).
7. Wallet scores use MetEngine proprietary models (not customizable).
8. No cross-platform wallet linking.
9. Not a price oracle. Polymarket prices = implied probabilities (0-1).
10. 73 endpoints only. If a path is not in this skill graph, it does not exist.
