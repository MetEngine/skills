---
skill: hyperliquid/positions
requires: [core/security, hyperliquid/overview]
triggers: ["hl.*trader", "hl.*leaderboard", "hl.*profile", "smart.*wallet.*hl", "fresh.*whale", "hl.*pnl", "compare.*trader"]
level: implementation
---

# Hyperliquid Trader and Position Endpoints

Trader profiling, leaderboards, PnL analysis, smart wallet tracking, and fresh whale detection.

#### 32. GET /api/v1/hl/traders/leaderboard

Ranked trader leaderboard.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | all | 1d, 7d, 30d, all | no |
| sort_by | string | pnl | pnl, roi, win_rate, volume | no |
| sort_order | string | desc | desc, asc | no |
| min_trades | integer | 10 | >= 1 | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "trader": "string", "total_pnl": "number", "total_volume": "number", "total_trades": "number", "winning_trades": "number", "win_rate": "number", "roi_pct": "number", "smart_score": "number" }] }
```

#### 33. POST /api/v1/hl/traders/profile

Full trader dossier. **Known issue:** Intermittent 500s. Fallback: leaderboard + pnl-by-coin.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 30 | 1-365 | no |
| trades_limit | integer | 50 | 1-500 | no |

```json
{ "data": { "trader": "string", "stats": "object", "coin_breakdown": "array", "daily_pnl": "array", "recent_trades": "array" } }
```

#### 34. POST /api/v1/hl/traders/compare

Compare 2-5 traders. **Pricing: `base * traders.length / 2`.**

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| traders | string[] | - | 2-5 0x addresses | yes |

```json
{ "data": { "traders": "array of trader profiles", "shared_coins": "array", "comparison_winners": "object" } }
```

#### 35. GET /api/v1/hl/traders/daily-pnl

Daily PnL time series with streaks.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 30 | 1-365 | no |

```json
{ "data": [{ "date": "string", "pnl": "number", "volume": "number", "trades": "number", "cumulative_pnl": "number", "streak": "number" }] }
```

#### 36. POST /api/v1/hl/traders/pnl-by-coin

Per-coin PnL breakdown. Shows realized (closed) PnL only -- unrealized PnL from open positions is not available.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 90 | 1-365 | no |

```json
{ "data": [{ "coin": "string", "pnl": "number", "volume": "number", "trades": "number", "winning_trades": "number", "win_rate": "number" }] }
```

#### 37. GET /api/v1/hl/traders/fresh-whales

New high-volume wallets (experienced traders creating new accounts, institutional desks, or insiders).

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| max_wallet_age_days | integer | 14 | 1-90 | no |
| min_volume | number | 100000 | >= 0 | no |
| min_pnl | number | 0 | any | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "trader": "string", "first_trade": "string", "wallet_age_days": "number", "total_pnl": "number", "total_volume": "number", "total_trades": "number", "winning_trades": "number" }] }
```

#### 41. GET /api/v1/hl/smart-wallets/list

Smart wallet list with scores.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| sort_by | string | score | score, pnl, volume | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": [{ "trader": "string", "score": "number", "total_pnl": "number", "total_volume": "number", "total_trades": "number", "win_rate": "number" }] }
```

#### 42. GET /api/v1/hl/smart-wallets/activity

Smart wallet recent trades.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| min_score | integer | 60 | 0-100 | no |
| limit | integer | 100 | 1-500 | no |

```json
{ "data": { "period_summary": "object", "trades": "array" } }
```
