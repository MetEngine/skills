---
skill: meteora/swaps
requires: [core/security, meteora/overview]
triggers: ["meteora.*stat", "dca.*pressure", "volume.*heatmap.*met", "metengine.*share", "meteora.*platform"]
level: implementation
---

# Meteora Platform and DCA Endpoints

Platform-wide stats, volume heatmaps, MetEngine routing share, and DCA pressure analysis.

#### 59. GET /api/v1/meteora/platform/stats

Platform-wide stats.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |

```json
{ "data": { "total_volume_usd": "number", "total_events": "number", "active_lps": "number", "active_pools": "number", "by_pool_type": "object" } }
```

#### 60. GET /api/v1/meteora/platform/volume-heatmap

Volume by action and hour.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 24h | 24h, 7d | no |

```json
{ "data": [{ "action": "string", "bucket": "string", "pool_type": "string", "volume_usd": "number", "event_count": "number" }] }
```

#### 61. GET /api/v1/meteora/platform/metengine-share

MetEngine routing share -- what percentage of Meteora activity is routed through MetEngine.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

```json
{ "data": { "total_events": "number", "metengine_events": "number", "metengine_share_pct": "number", "total_volume_usd": "number", "metengine_volume_usd": "number" } }
```

#### 62. GET /api/v1/meteora/dca/pressure

DCA accumulation pressure by token. When called without a `token` parameter, returns pressure across all tokens.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| token | string | - | specific mint address | no |
| timeframe | string | 1h | 1m, 5m, 30m, 1h, 6h, 24h | no |

**Single token response:**
```json
{ "data": { "tokenMint": "string", "timeframe": "string", "buyVolume": "number", "sellVolume": "number", "netPressure": "number", "score": "number" } }
```

**All tokens response (when `token` omitted):**
```json
{ "data": { "timeframe": "string", "tokens": "array", "lastUpdated": "string" } }
```
