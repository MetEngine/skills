---
name: metengine-data-agent
description: Real-time smart money analytics across Polymarket prediction markets, Hyperliquid perpetual futures, and Meteora Solana LP/AMM pools. 73 endpoints. Pay per request with USDC on Tempo (EVM) via MPP -- no API keys, no accounts.
triggers:
  - smart money
  - prediction market
  - polymarket
  - hyperliquid
  - perpetual futures
  - meteora
  - solana lp
  - whale trades
  - insider detection
  - wallet profile
  - capital flow
  - market intelligence
  - trading analytics
  - on-chain analytics
---

# MetEngine Data Agent

Real-time smart money analytics API. 73 endpoints across three platforms. Pay per request with USDC on Tempo (EVM) via MPP -- no API keys, no accounts needed.

**Base URL:** `https://agent.metengine.xyz`

## Agent Instructions

Everything you need is in this file. Do NOT explore, discover, or guess.

**Rules:**
1. The base URL is ALWAYS `https://agent.metengine.xyz` -- never guess other URLs
2. Do NOT run `npx mppx --help`, `npx mppx --llms`, or any discovery commands
3. Do NOT search the codebase for API docs -- this file IS the complete reference
4. Match the user's intent to an endpoint from the tables below
5. Construct the full URL with query parameters and run it directly:
   ```
   npx mppx "https://agent.metengine.xyz/api/v1/<endpoint>?<params>"
   ```
6. For POST endpoints, use: `npx mppx -X POST -d '{"key":"value"}' "https://agent.metengine.xyz/api/v1/<endpoint>"`
7. `mppx` handles 402 payment challenges automatically -- no manual signing needed
8. Parse the JSON response and present the data clearly to the user
9. Never truncate addresses or IDs. Always return full values (EVM hex, Solana base58, condition IDs, token IDs, pool addresses, tx hashes)

**Pre-flight (first request only):**
```bash
npx mppx account list
```
If no accounts exist, tell the user to run `npx mppx account create` and fund the wallet.

## Platforms

| Platform | Endpoints | Domain |
|----------|-----------|--------|
| Polymarket | 37 | Prediction markets (Polygon) |
| Hyperliquid | 18 | Perpetual futures (Hyperliquid L1) |
| Meteora | 18 | Solana LP/AMM pools |

## Quick Start

### 1. Create a wallet

```bash
npx mppx account create
```

This generates an EVM wallet and stores it locally (`~/.mppx/`).

### 2. Fund your wallet

```bash
# View your wallet address
npx mppx account view
```

Send USDC.e to this address on Tempo chain (chain ID 4217):
- **Bridge:** `https://bridge.tempo.xyz` (from Ethereum, Arbitrum, or Base)
- **USDC.e contract:** `0x20C000000000000000000000b9537d11c60E8b50`
- **RPC:** `https://rpc.presto.tempo.xyz`
- **Explorer:** `https://explore.tempo.xyz`
- **Native gas:** USD (negligible cost per transaction)
- **Recommended starting amount:** $5-10 USDC (covers 50-500+ requests)

There is no testnet -- all requests cost real USDC.

### 3. Make a request

```bash
# CLI (automatic 402 handling)
npx mppx "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5"
```

### 4. How payment works

```
1. Client: GET /api/v1/markets/trending
2. Server: 402 + WWW-Authenticate: Payment (challenge with amount, recipient)
3. Client: Sign a TIP-20 USDC transfer transaction (do NOT broadcast)
4. Client: Re-send with Authorization: Payment <credential>
5. Server: Execute query -> if data returned -> broadcast tx -> 200 + Payment-Receipt
```

**Settle-after-execute guarantee:** If the query fails, times out, or returns empty results, no payment is settled. You only pay when data is returned.

## Free Endpoints

These require no wallet or payment:

```bash
npx mppx https://agent.metengine.xyz/health
npx mppx https://agent.metengine.xyz/api/v1/pricing
npx mppx https://agent.metengine.xyz/.well-known/mpp
```

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Service health check |
| `GET /api/v1/pricing` | Full pricing schedule with all tiers, modifiers, and discounts |
| `GET /.well-known/mpp` | MPP discovery -- complete endpoint catalog with descriptions and prices |

## Pricing

### Tiers

| Tier | Base Cost | Description |
|------|-----------|-------------|
| light | $0.01 | Simple lookups, searches, feeds |
| medium | $0.02 | Aggregated analytics, trending data |
| heavy | $0.05 | Computed intelligence, leaderboards, scoring |
| whale | $0.08 | Multi-entity comparisons, complex scans |

### Modifiers

Prices scale dynamically based on query parameters:

- **Timeframe:** 1h=0.5x, 4h=0.7x, 12h=0.9x, 24h=1.0x, 7d=2.0x, 30d=3.0x, 90d=4.0x
- **Limit:** Proportional to default limit (requesting 2x default = 2x price)
- **Filter discounts:** category=0.7x, condition_id=0.5x, smart_money_only=0.7x, pool_address=0.5x, coin=0.7x, pool_type=0.7x
- **Floor:** $0.01 | **Ceiling:** $0.20

## Polymarket Endpoints (37)

### Markets

| Endpoint | Tier | Description |
|----------|------|-------------|
| `GET /api/v1/markets/trending` | $0.02 | Trending markets with volume spikes |
| `GET /api/v1/markets/search` | $0.01 | Search markets by keyword, category, status, or URL |
| `GET /api/v1/markets/categories` | $0.01 | All categories with activity stats |
| `POST /api/v1/markets/intelligence` | $0.05 | Full smart money intelligence report for a market |
| `GET /api/v1/markets/price-history` | $0.01 | OHLCV price/probability time series |
| `POST /api/v1/markets/sentiment` | $0.02 | Sentiment time series with smart money overlay |
| `POST /api/v1/markets/participants` | $0.02 | Participant summary by scoring tier |
| `POST /api/v1/markets/insiders` | $0.05 | Insider pattern detection (7-signal behavioral scoring) |
| `GET /api/v1/markets/trades` | $0.01 | Chronological trade feed for a market |
| `GET /api/v1/markets/similar` | $0.08 | Related markets by wallet overlap |
| `GET /api/v1/markets/opportunities` | $0.08 | Markets where smart money disagrees with price |
| `GET /api/v1/markets/high-conviction` | $0.05 | High-conviction smart money bets |
| `GET /api/v1/markets/capital-flow` | $0.02 | Capital flow by category (sector rotation) |
| `GET /api/v1/markets/resolutions` | $0.01 | Resolved markets with smart money accuracy |
| `GET /api/v1/markets/volume-heatmap` | $0.02 | Volume distribution across categories/hours/days |
| `GET /api/v1/markets/smart-signals` | $0.05 | Per-market aggregated smart wallet directional signals |
| `POST /api/v1/markets/smart-flow` | $0.02 | Smart money accumulation vs distribution time series |
| `GET /api/v1/markets/new-smart-interest` | $0.05 | Markets with first-time smart wallet positions |
| `GET /api/v1/markets/dumb-money` | $0.02 | Low-score trader positions (contrarian indicator) |
| `GET /api/v1/markets/dumb-consensus` | $0.05 | Global dumb money consensus (contrarian indicator) |
| `GET /api/v1/markets/capital-flow-comparison` | $0.02 | Multi-timeframe capital flow comparison |
| `GET /api/v1/markets/early-movers` | $0.02 | Earliest buyers for a market |
| `GET /api/v1/markets/ownership` | $0.02 | Token ownership distribution for a market |
| `GET /api/v1/markets/volume-by-group` | $0.02 | Volume grouped by category, hour, or day |

### Wallets

| Endpoint | Tier | Description |
|----------|------|-------------|
| `POST /api/v1/wallets/profile` | $0.05 | Full wallet dossier with score, positions, trades |
| `POST /api/v1/wallets/activity` | $0.02 | Recent trading activity for a wallet |
| `POST /api/v1/wallets/pnl-breakdown` | $0.02 | Per-market PnL breakdown |
| `POST /api/v1/wallets/compare` | $0.08 | Compare 2-5 wallets side-by-side |
| `POST /api/v1/wallets/copy-traders` | $0.08 | Detect wallets copying a target wallet |
| `GET /api/v1/wallets/top-performers` | $0.05 | Leaderboard by PnL, ROI, Sharpe, win rate, volume |
| `GET /api/v1/wallets/niche-experts` | $0.05 | Top wallets in a specific category |
| `GET /api/v1/wallets/alpha-callers` | $0.05 | Wallets that trade early on later-trending markets |
| `GET /api/v1/wallets/insiders` | $0.05 | Global insider candidates by behavioral score |
| `POST /api/v1/wallets/portfolio` | $0.08 | Aggregate portfolio view for a watchlist |
| `GET /api/v1/wallets/insiders/trend` | $0.02 | Insider activity trend over time |

### Other

| Endpoint | Tier | Description |
|----------|------|-------------|
| `GET /api/v1/platform/stats` | $0.01 | Platform-wide aggregate stats |
| `GET /api/v1/trades/whales` | $0.02 | Large whale trades |

## Hyperliquid Endpoints (18)

| Endpoint | Tier | Description |
|----------|------|-------------|
| `GET /api/v1/hl/platform/stats` | $0.01 | Platform aggregate stats |
| `GET /api/v1/hl/coins/trending` | $0.02 | Trending coins by activity |
| `GET /api/v1/hl/coins/list` | $0.01 | All traded coins with 7d stats |
| `GET /api/v1/hl/coins/volume-heatmap` | $0.02 | Volume by coin and hour |
| `GET /api/v1/hl/traders/leaderboard` | $0.05 | Ranked trader leaderboard |
| `POST /api/v1/hl/traders/profile` | $0.05 | Full trader dossier |
| `POST /api/v1/hl/traders/compare` | $0.08 | Compare 2-5 traders |
| `GET /api/v1/hl/traders/daily-pnl` | $0.02 | Daily PnL time series with streaks |
| `POST /api/v1/hl/traders/pnl-by-coin` | $0.02 | Per-coin PnL breakdown (closed PnL only) |
| `GET /api/v1/hl/traders/fresh-whales` | $0.05 | New high-volume wallets |
| `GET /api/v1/hl/trades/whales` | $0.02 | Large trades |
| `GET /api/v1/hl/trades/feed` | $0.01 | Chronological trade feed for a coin |
| `GET /api/v1/hl/trades/long-short-ratio` | $0.02 | Long/short volume ratio time series |
| `GET /api/v1/hl/smart-wallets/list` | $0.01 | Smart wallet list with scores |
| `GET /api/v1/hl/smart-wallets/activity` | $0.02 | Smart wallet recent trades |
| `GET /api/v1/hl/smart-wallets/signals` | $0.05 | Aggregated directional signals by coin |
| `GET /api/v1/hl/pressure/pairs` | $0.05 | Long/short pressure with smart positions |
| `GET /api/v1/hl/pressure/summary` | $0.02 | Pressure summary across all coins |

## Meteora Endpoints (18)

| Endpoint | Tier | Description |
|----------|------|-------------|
| `GET /api/v1/meteora/pools/trending` | $0.02 | Trending pools by volume spike |
| `GET /api/v1/meteora/pools/top` | $0.02 | Top pools by volume, fees, fee rate, or LP count |
| `GET /api/v1/meteora/pools/search` | $0.01 | Search pools by address or token name |
| `GET /api/v1/meteora/pools/detail` | $0.02 | Full pool detail |
| `GET /api/v1/meteora/pools/volume-history` | $0.01 | Volume time series |
| `GET /api/v1/meteora/pools/events` | $0.01 | Chronological event feed |
| `GET /api/v1/meteora/pools/fee-analysis` | $0.02 | Fee claiming analysis |
| `GET /api/v1/meteora/pools/smart-wallet` | $0.05 | Pools with highest smart wallet LP activity |
| `GET /api/v1/meteora/lps/top` | $0.05 | Top LPs leaderboard |
| `POST /api/v1/meteora/lps/profile` | $0.05 | Full LP dossier |
| `GET /api/v1/meteora/lps/whales` | $0.02 | Large LP events |
| `POST /api/v1/meteora/lps/compare` | $0.08 | Compare 2-5 LPs |
| `GET /api/v1/meteora/positions/active` | $0.02 | Active LP positions |
| `GET /api/v1/meteora/positions/history` | $0.01 | Position event history (DLMM only) |
| `GET /api/v1/meteora/platform/stats` | $0.01 | Platform-wide stats |
| `GET /api/v1/meteora/platform/volume-heatmap` | $0.02 | Volume by action/hour |
| `GET /api/v1/meteora/platform/metengine-share` | $0.01 | MetEngine routing share |
| `GET /api/v1/meteora/dca/pressure` | $0.02 | DCA accumulation pressure by token |

## Usage Examples

### Polymarket: Find smart money alpha

```bash
# What markets are trending?
npx mppx "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=10"

# Where is smart money disagreeing with the market price?
npx mppx "https://agent.metengine.xyz/api/v1/markets/opportunities?timeframe=7d&limit=10"

# Full intelligence report for a specific market
npx mppx "https://agent.metengine.xyz/api/v1/markets/intelligence?condition_id=0x..."

# Who are the top performers this week?
npx mppx "https://agent.metengine.xyz/api/v1/wallets/top-performers?timeframe=7d&sort_by=pnl&limit=10"

# Detect insider trading patterns
npx mppx "https://agent.metengine.xyz/api/v1/markets/insiders?condition_id=0x...&timeframe=30d"
```

### Hyperliquid: Track perpetual futures traders

```bash
# Trending coins right now
npx mppx "https://agent.metengine.xyz/api/v1/hl/coins/trending?timeframe=24h"

# Top trader leaderboard
npx mppx "https://agent.metengine.xyz/api/v1/hl/traders/leaderboard?timeframe=7d&sort_by=pnl"

# Smart wallet directional signals
npx mppx "https://agent.metengine.xyz/api/v1/hl/smart-wallets/signals?timeframe=24h"

# Long/short pressure across all coins
npx mppx "https://agent.metengine.xyz/api/v1/hl/pressure/summary?timeframe=24h"

# Fresh whales (new high-volume wallets)
npx mppx "https://agent.metengine.xyz/api/v1/hl/traders/fresh-whales?timeframe=7d"
```

### Meteora: Analyze Solana LP activity

```bash
# Trending pools by volume spike
npx mppx "https://agent.metengine.xyz/api/v1/meteora/pools/trending?timeframe=24h"

# Top LPs leaderboard
npx mppx "https://agent.metengine.xyz/api/v1/meteora/lps/top?timeframe=7d&sort_by=volume"

# Full LP profile
npx mppx "https://agent.metengine.xyz/api/v1/meteora/lps/profile?owner=<base58_address>"

# Pools where smart wallets are providing liquidity
npx mppx "https://agent.metengine.xyz/api/v1/meteora/pools/smart-wallet?timeframe=7d"

# DCA accumulation pressure by token
npx mppx "https://agent.metengine.xyz/api/v1/meteora/dca/pressure?timeframe=24h"
```

## Address Conventions

| Platform | Chain | Format |
|----------|-------|--------|
| Polymarket | Polygon (EVM) | 0x hex, MUST be lowercase |
| Hyperliquid | Hyperliquid L1 / Arbitrum (EVM) | 0x hex, case-insensitive |
| Meteora | Solana | Base58, case-sensitive |

Never truncate addresses or IDs. Always use full values.

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `TIP20 token error: Uninitialized` | Wallet has no USDC.e on Tempo mainnet | Fund wallet: bridge USDC to Tempo via `https://bridge.tempo.xyz` |
| `DNS_ERROR` / `Could not resolve host` | Wrong base URL | Use `https://agent.metengine.xyz` -- no other URL exists |
| `402 Payment Required` | Normal -- mppx handles this automatically | If mppx fails to handle it, check wallet has sufficient USDC.e balance |
| `Method Not Allowed` | Wrong HTTP method for endpoint | Check endpoint table: GET vs POST |
| `REQUEST_FAILED` after `mppx account fund` | `fund` provides testnet tokens only | MetEngine runs on mainnet -- bridge real USDC via `https://bridge.tempo.xyz` |

**Verify wallet is funded:**
```bash
npx mppx account view
# Look for a non-zero USDC balance (not testnet tokens)
```
