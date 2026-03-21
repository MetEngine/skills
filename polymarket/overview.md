---
skill: polymarket/overview
requires: [core/security]
triggers: ["polymarket", "condition_id", "prediction", "outcome", "market.*search"]
level: concept
---

# Polymarket Overview

Prediction market analytics on Polygon. 28 endpoints covering market discovery, smart money intelligence, insider detection, wallet profiling, and capital flow tracking. MetEngine indexes 574M+ trades and scores millions of wallets using a proprietary composite model.

## Endpoint Summary (28 endpoints)

| # | Method | Path | Tier | Description |
|---|--------|------|------|-------------|
| 1 | GET | /api/v1/markets/trending | medium | Trending markets with volume spikes |
| 2 | GET | /api/v1/markets/search | light | Search by keyword, category, status, or URL |
| 3 | GET | /api/v1/markets/categories | light | All categories with activity stats |
| 4 | GET | /api/v1/platform/stats | light | Platform-wide aggregate stats |
| 5 | POST | /api/v1/markets/intelligence | heavy | Full smart money intelligence report |
| 6 | GET | /api/v1/markets/price-history | light | OHLCV price/probability time series |
| 7 | POST | /api/v1/markets/sentiment | medium | Sentiment time series + smart money overlay |
| 8 | POST | /api/v1/markets/participants | medium | Participant summary by scoring tier |
| 9 | POST | /api/v1/markets/insiders | heavy | Per-market insider detection (full two-phase 7-signal scoring) |
| 10 | GET | /api/v1/markets/trades | light | Chronological trade feed |
| 11 | GET | /api/v1/markets/similar | whale | Related markets by wallet overlap |
| 12 | GET | /api/v1/markets/opportunities | whale | Smart money vs price disagreement |
| 13 | GET | /api/v1/markets/high-conviction | heavy | High-conviction smart money bets |
| 14 | GET | /api/v1/markets/capital-flow | medium | Capital flow by category (sector rotation) |
| 15 | GET | /api/v1/trades/whales | medium | Large whale trades |
| 16 | GET | /api/v1/markets/volume-heatmap | medium | Volume distribution by category/hour/day |
| 17 | POST | /api/v1/wallets/profile | heavy | Full wallet dossier |
| 18 | POST | /api/v1/wallets/activity | medium | Recent trading activity |
| 19 | POST | /api/v1/wallets/pnl-breakdown | medium | Per-market PnL breakdown |
| 20 | POST | /api/v1/wallets/compare | whale | Compare 2-5 wallets |
| 21 | POST | /api/v1/wallets/copy-traders | whale | Detect copy-trading wallets |
| 22 | GET | /api/v1/wallets/top-performers | heavy | Leaderboard by PnL/ROI/Sharpe |
| 23 | GET | /api/v1/wallets/niche-experts | heavy | Top wallets in a category |
| 24 | GET | /api/v1/markets/resolutions | light | Resolved markets + smart money accuracy |
| 25 | GET | /api/v1/wallets/alpha-callers | heavy | Early movers on later-trending markets |
| 26 | GET | /api/v1/markets/dumb-money | medium | Low-score positions (contrarian indicator) |
| 27 | GET | /api/v1/wallets/insiders | heavy | Global insider candidates |
| 28 | GET | /api/v1/markets/insider-markets | medium | Markets grouped by insider activity |

## Scoring System (0-100)

| Tier | Score | Meaning |
|------|-------|---------|
| Elite | >= 80 | Top-tier historically profitable |
| Smart | 60-79 | Consistently profitable, signal-worthy |
| Average | 40-59 | Mixed track record |
| Novice | < 40 | New or losing traders |

Smart money threshold: score >= 60. Dumb money threshold: score < 30.

## Key Concepts

- **condition_id**: Uniquely identifies a market. One condition_id maps to multiple token_ids (one per outcome).
- **Price**: Implied probability (0 to 1). Price of 0.73 = 73% probability.
- **REDEEM trades**: Resolved market payouts. Filter by `side=BUY` or `side=SELL` to exclude. REDEEMs have `price=1.00, side=REDEEM`.

## Address Format

- 0x hex (Polygon). Must be lowercase for all API calls.

## Known Quirks

- `/markets/opportunities` frequently times out (504) under load. Fallback: `/markets/high-conviction`.
- `/wallets/top-performers?timeframe=7d` may 503. Try `timeframe=24h`.
- `/trades/whales` returns REDEEM trades alongside real trades.
- `/markets/insiders` has a 10s timeout on full two-phase scoring. Fallback: `/markets/trades` filtered to the market.
- `/markets/insider-markets` returns empty if global cache is cold (scheduler runs every 10 min).
