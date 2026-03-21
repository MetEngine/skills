---
skill: hyperliquid/trading
requires: [core/security, core/trading-patterns, hyperliquid/overview]
triggers: ["hl.*trade", "whale.*hl", "long.*short", "pressure.*hl", "hl.*feed"]
level: implementation
---

# Hyperliquid Trading Endpoints

Whale trades, trade feeds, long/short ratio, and pressure analysis.

#### 38. GET /api/v1/hl/trades/whales

Large trades.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_usd | number | 50000 | >= 1 | no |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| coin | string | - | uppercase symbol (e.g. BTC) | no |
| side | string | - | Open Long, Open Short, Close Long, Close Short | no |
| direction | string | - | Long, Short | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": [{ "trader": "string", "coin": "string", "side": "string", "direction": "string", "price": "number", "size": "number", "usd_value": "number", "closed_pnl": "number", "timestamp": "string", "smart_score": "number" }] }
```

#### 39. GET /api/v1/hl/trades/feed

Chronological trade feed for a coin.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coin | string | - | uppercase symbol | yes |
| limit | integer | 100 | 1-500 | no |

```json
{ "data": [{ "trader": "string", "coin": "string", "side": "string", "direction": "string", "price": "number", "size": "number", "usd_value": "number", "closed_pnl": "number", "timestamp": "string" }] }
```

#### 40. GET /api/v1/hl/trades/long-short-ratio

Long/short volume ratio time series. **Known issue:** May return all zeros. Reconstruct from `/hl/trades/whales` by counting long vs short volume.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coin | string | - | uppercase symbol | yes |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| bucket_size | string | 4h | 1h, 4h, 12h, 1d | no |

```json
{ "data": [{ "bucket": "string", "long_volume": "number", "short_volume": "number", "long_short_ratio": "number", "smart_long_volume": "number", "smart_short_volume": "number", "smart_long_short_ratio": "number" }] }
```

#### 44. GET /api/v1/hl/pressure/pairs

Long/short pressure with smart positions for specific coins.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| coins | string | - | comma-separated (defaults to top 10) | no |
| trades_limit | integer | 20 | 1-100 | no |

```json
{ "data": { "coins": [{ "coin": "string", "long_pressure": "number", "short_pressure": "number", "long_avg_entry": "number", "short_avg_entry": "number", "long_smart_count": "number", "short_smart_count": "number", "smart_positions": "array" }] } }
```

#### 45. GET /api/v1/hl/pressure/summary

Pressure summary across all coins. No parameters.

```json
{ "data": { "coins": [{ "coin": "string", "long_pressure": "number", "short_pressure": "number", "long_percent": "number", "short_percent": "number", "long_avg_entry": "number", "short_avg_entry": "number" }] } }
```
