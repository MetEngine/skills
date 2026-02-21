# Hyperliquid Endpoint Reference

## Scope

Detailed endpoint contracts for Hyperliquid routes.

## How To Use

- Search by path with `rg "GET /api/v1/hl" references/hyperliquid-endpoints.md`.
- Load only the endpoint section needed for the active query.

## TOC

- Endpoint list starts at the `####` headings below.

### HYPERLIQUID

#### 28. GET /api/v1/hl/platform/stats

Platform aggregate stats. No parameters.

**Response schema:**
```json
{
  "data": {
    "total_volume_usd": "number",
    "total_trades": "number",
    "active_traders": "number",
    "smart_wallet_count": "number",
    "unique_coins": "number"
  }
}
```

#### 29. GET /api/v1/hl/coins/trending

Trending coins by activity.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "coin": "string",
      "volume_usd": "number",
      "trade_count": "number",
      "unique_traders": "number",
      "long_short_ratio": "number",
      "smart_wallet_count": "number",
      "volume_change_pct": "number"
    }
  ]
}
```

**Known issue:** `timeframe=24h` often returns empty. Use `timeframe=7d` as fallback.

#### 30. GET /api/v1/hl/coins/list

All traded coins with 7d stats.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| limit | integer | 100 | 1-500 | no |

**Response schema:**
```json
{
  "data": [
    {
      "coin": "string",
      "volume_usd": "number",
      "trade_count": "number",
      "unique_traders": "number"
    }
  ]
}
```

#### 31. GET /api/v1/hl/coins/volume-heatmap

Volume by coin and hour.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d | no |
| limit | integer | 20 | 1-50 | no |

**Response schema:**
```json
{
  "data": [
    {
      "coin": "string",
      "bucket": "string",
      "volume_usd": "number",
      "trade_count": "number",
      "smart_volume_usd": "number"
    }
  ]
}
```

#### 32. GET /api/v1/hl/traders/leaderboard

Ranked trader leaderboard.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | all | 1d, 7d, 30d, all | no |
| sort_by | string | pnl | pnl, roi, win_rate, volume | no |
| sort_order | string | desc | desc, asc | no |
| min_trades | integer | 10 | >= 1 | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "trader": "string",
      "total_pnl": "number",
      "total_volume": "number",
      "total_trades": "number",
      "winning_trades": "number",
      "win_rate": "number",
      "roi_pct": "number",
      "smart_score": "number"
    }
  ]
}
```

#### 33. POST /api/v1/hl/traders/profile

Full trader dossier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 30 | 1-365 | no |
| trades_limit | integer | 50 | 1-500 | no |

**Response schema:**
```json
{
  "data": {
    "trader": "string",
    "stats": "object",
    "coin_breakdown": "array",
    "daily_pnl": "array",
    "recent_trades": "array"
  }
}
```

**Known issue:** Intermittent 500 errors. Fallback: use leaderboard + pnl-by-coin.

#### 34. POST /api/v1/hl/traders/compare

Compare 2-5 traders.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| traders | string[] | - | 2-5 0x addresses | yes |

**Response schema:**
```json
{
  "data": {
    "traders": "array of trader profiles",
    "shared_coins": "array",
    "comparison_winners": "object"
  }
}
```

**Pricing note:** Cost scales with trader count: `base * traders.length / 2`.

#### 35. GET /api/v1/hl/traders/daily-pnl

Daily PnL time series with streaks.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 30 | 1-365 | no |

**Response schema:**
```json
{
  "data": [
    {
      "date": "string",
      "pnl": "number",
      "volume": "number",
      "trades": "number",
      "cumulative_pnl": "number",
      "streak": "number"
    }
  ]
}
```

#### 36. POST /api/v1/hl/traders/pnl-by-coin

Per-coin PnL breakdown. Shows realized (closed) PnL only.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| trader | string | - | 0x address | yes |
| days | integer | 90 | 1-365 | no |

**Response schema:**
```json
{
  "data": [
    {
      "coin": "string",
      "pnl": "number",
      "volume": "number",
      "trades": "number",
      "winning_trades": "number",
      "win_rate": "number"
    }
  ]
}
```

**Note:** Only realized (closed) PnL. Unrealized PnL from open positions is not available.

#### 37. GET /api/v1/hl/traders/fresh-whales

New high-volume wallets (experienced traders creating new accounts, institutional desks, or insiders).

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| max_wallet_age_days | integer | 14 | 1-90 | no |
| min_volume | number | 100000 | >= 0 | no |
| min_pnl | number | 0 | any | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "trader": "string",
      "first_trade": "string",
      "wallet_age_days": "number",
      "total_pnl": "number",
      "total_volume": "number",
      "total_trades": "number",
      "winning_trades": "number"
    }
  ]
}
```

#### 38. GET /api/v1/hl/trades/whales

Large trades.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_usd | number | 50000 | >= 1 | no |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| coin | string | - | uppercase symbol (e.g. BTC) | no |
| side | string | - | Open Long, Open Short, Close Long, Close Short | no |
| direction | string | - | Long, Short | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": [
    {
      "trader": "string",
      "coin": "string",
      "side": "string",
      "direction": "string",
      "price": "number",
      "size": "number",
      "usd_value": "number",
      "closed_pnl": "number",
      "timestamp": "string",
      "smart_score": "number"
    }
  ]
}
```

#### 39. GET /api/v1/hl/trades/feed

Chronological trade feed for a coin.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coin | string | - | uppercase symbol | yes |
| limit | integer | 100 | 1-500 | no |

**Response schema:**
```json
{
  "data": [
    {
      "trader": "string",
      "coin": "string",
      "side": "string",
      "direction": "string",
      "price": "number",
      "size": "number",
      "usd_value": "number",
      "closed_pnl": "number",
      "timestamp": "string"
    }
  ]
}
```

#### 40. GET /api/v1/hl/trades/long-short-ratio

Long/short volume ratio time series.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coin | string | - | uppercase symbol | yes |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| bucket_size | string | 4h | 1h, 4h, 12h, 1d | no |

**Response schema:**
```json
{
  "data": [
    {
      "bucket": "string",
      "long_volume": "number",
      "short_volume": "number",
      "long_short_ratio": "number",
      "smart_long_volume": "number",
      "smart_short_volume": "number",
      "smart_long_short_ratio": "number"
    }
  ]
}
```

**Known issue:** May return all zeros. Fallback: reconstruct from `/hl/trades/whales` by counting long vs short volume.

#### 41. GET /api/v1/hl/smart-wallets/list

Smart wallet list.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| sort_by | string | score | score, pnl, volume | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": [
    {
      "trader": "string",
      "score": "number",
      "total_pnl": "number",
      "total_volume": "number",
      "total_trades": "number",
      "win_rate": "number"
    }
  ]
}
```

#### 42. GET /api/v1/hl/smart-wallets/activity

Smart wallet recent trades.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| min_score | integer | 60 | 0-100 | no |
| limit | integer | 100 | 1-500 | no |

**Response schema:**
```json
{
  "data": {
    "period_summary": "object",
    "trades": "array"
  }
}
```

#### 43. GET /api/v1/hl/smart-wallets/signals

Aggregated directional signals by coin.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 6h, 12h, 24h, 7d | no |
| min_score | integer | 60 | 0-100 | no |
| limit | integer | 20 | 1-50 | no |

**Response schema:**
```json
{
  "data": [
    {
      "coin": "string",
      "direction": "string",
      "smart_wallet_count": "number",
      "total_volume": "number",
      "avg_score": "number",
      "top_traders": "array"
    }
  ]
}
```

**Known issue:** `timeframe=24h` often returns empty. Use `timeframe=7d` as fallback.

#### 44. GET /api/v1/hl/pressure/pairs

Long/short pressure with smart positions.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coins | string | - | comma-separated (defaults to top 10) | no |
| trades_limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": {
    "coins": [
      {
        "coin": "string",
        "long_pressure": "number",
        "short_pressure": "number",
        "long_avg_entry": "number",
        "short_avg_entry": "number",
        "long_smart_count": "number",
        "short_smart_count": "number",
        "smart_positions": "array"
      }
    ]
  }
}
```

#### 45. GET /api/v1/hl/pressure/summary

Pressure summary across all coins. No parameters.

**Response schema:**
```json
{
  "data": {
    "coins": [
      {
        "coin": "string",
        "long_pressure": "number",
        "short_pressure": "number",
        "long_percent": "number",
        "short_percent": "number",
        "long_avg_entry": "number",
        "short_avg_entry": "number"
      }
    ]
  }
}
```

---

