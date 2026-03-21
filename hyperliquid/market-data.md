---
skill: hyperliquid/market-data
requires: [core/security, hyperliquid/overview]
triggers: ["hl.*coin", "hyperliquid.*trending", "coin.*list", "hl.*heatmap", "hl.*signal", "smart.*wallet.*signal"]
level: implementation
---

# Hyperliquid Market Data Endpoints

Coin discovery, trending, volume heatmaps, and smart wallet signals.

#### 28. GET /api/v1/hl/platform/stats

Platform aggregate stats. No parameters.

```json
{ "data": { "total_volume_usd": "number", "total_trades": "number", "active_traders": "number", "smart_wallet_count": "number", "unique_coins": "number" } }
```

#### 29. GET /api/v1/hl/coins/trending

Trending coins by activity. **Known issue:** `timeframe=24h` often returns empty -- use `timeframe=7d`.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "coin": "string", "volume_usd": "number", "trade_count": "number", "unique_traders": "number", "long_short_ratio": "number", "smart_wallet_count": "number", "volume_change_pct": "number" }] }
```

#### 30. GET /api/v1/hl/coins/list

All traded coins with 7d stats.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| limit | integer | 100 | 1-500 | no |

```json
{ "data": [{ "coin": "string", "volume_usd": "number", "trade_count": "number", "unique_traders": "number" }] }
```

#### 31. GET /api/v1/hl/coins/volume-heatmap

Volume by coin and hour.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d | no |
| limit | integer | 20 | 1-50 | no |

```json
{ "data": [{ "coin": "string", "bucket": "string", "volume_usd": "number", "trade_count": "number", "smart_volume_usd": "number" }] }
```

#### 43. GET /api/v1/hl/smart-wallets/signals

Aggregated directional signals by coin. **Known issue:** `timeframe=24h` often returns empty -- use `timeframe=7d`.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 6h, 12h, 24h, 7d | no |
| min_score | integer | 60 | 0-100 | no |
| limit | integer | 20 | 1-50 | no |

```json
{ "data": [{ "coin": "string", "direction": "string", "smart_wallet_count": "number", "total_volume": "number", "avg_score": "number", "top_traders": "array" }] }
```
