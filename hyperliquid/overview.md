---
skill: hyperliquid/overview
requires: [core/security]
triggers: ["hyperliquid", "perp", "funding", "open interest", "leverage", "hl"]
level: concept
---

# Hyperliquid Overview

Perpetual futures analytics on Hyperliquid L1. 18 endpoints covering coin trending, trader profiling, smart wallet signals, whale trade detection, and long/short pressure analysis. MetEngine indexes all trades with sub-minute latency and scores wallets using a formula-based smart score.

## Endpoint Summary (18 endpoints)

| # | Method | Path | Tier | Description |
|---|--------|------|------|-------------|
| 28 | GET | /api/v1/hl/platform/stats | light | Platform aggregate stats |
| 29 | GET | /api/v1/hl/coins/trending | medium | Trending coins by activity |
| 30 | GET | /api/v1/hl/coins/list | light | All traded coins with 7d stats |
| 31 | GET | /api/v1/hl/coins/volume-heatmap | medium | Volume by coin and hour |
| 32 | GET | /api/v1/hl/traders/leaderboard | heavy | Ranked trader leaderboard |
| 33 | POST | /api/v1/hl/traders/profile | heavy | Full trader dossier |
| 34 | POST | /api/v1/hl/traders/compare | whale | Compare 2-5 traders |
| 35 | GET | /api/v1/hl/traders/daily-pnl | medium | Daily PnL time series with streaks |
| 36 | POST | /api/v1/hl/traders/pnl-by-coin | medium | Per-coin PnL breakdown (closed only) |
| 37 | GET | /api/v1/hl/traders/fresh-whales | heavy | New high-volume wallets |
| 38 | GET | /api/v1/hl/trades/whales | medium | Large trades |
| 39 | GET | /api/v1/hl/trades/feed | light | Chronological trade feed for a coin |
| 40 | GET | /api/v1/hl/trades/long-short-ratio | medium | Long/short volume ratio time series |
| 41 | GET | /api/v1/hl/smart-wallets/list | light | Smart wallet list with scores |
| 42 | GET | /api/v1/hl/smart-wallets/activity | medium | Smart wallet recent trades |
| 43 | GET | /api/v1/hl/smart-wallets/signals | heavy | Aggregated directional signals by coin |
| 44 | GET | /api/v1/hl/pressure/pairs | heavy | Long/short pressure with smart positions |
| 45 | GET | /api/v1/hl/pressure/summary | medium | Pressure summary across all coins |

## Scoring System (0-100)

```
score = min(log2(trade_count + 1) * 4, 40)             // Activity (max 40)
      + (winning_trades / max(closing_trades, 1)) * 40  // Win rate (max 40)
      + min(if(pnl > 0, log10(pnl + 1) * 4, 0), 20)    // Profitability (max 20)
```

Smart wallet threshold: score >= 85 (stricter than Polymarket's 60).

## Key Concepts

- **Coin symbols**: Uppercase base only: `BTC`, not `BTC-USDC`.
- **PnL**: Only realized (closed) PnL available. No unrealized PnL from open positions.
- **`closed_pnl`**: Only non-zero on closing trades. Opening trades always have `closed_pnl = 0`.
- **Trade sides**: `Open Long`, `Open Short`, `Close Long`, `Close Short`.

## Address Format

- 0x hex (case-insensitive). Trader addresses are 0x-prefixed.

## Known Quirks

- `/hl/coins/trending?timeframe=24h` often returns empty. Default to `timeframe=7d`.
- `/hl/smart-wallets/signals?timeframe=24h` often returns empty. Default to `timeframe=7d`.
- `/hl/trades/long-short-ratio` may return all zeros. Reconstruct from `/hl/trades/whales`.
- `/hl/traders/profile` intermittently 500s. Fallback: leaderboard + pnl-by-coin.
