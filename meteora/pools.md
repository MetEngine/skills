---
skill: meteora/pools
requires: [core/security, meteora/overview]
triggers: ["meteora.*pool", "pool.*search", "pool.*detail", "fee.*analysis", "pool.*trending", "smart.*wallet.*pool"]
level: implementation
---

# Meteora Pool Endpoints

Pool discovery, search, detail, volume history, events, fee analysis, and smart wallet activity.

#### 46. GET /api/v1/meteora/pools/trending

Trending pools by volume spike. **Known issue:** May contain duplicates -- deduplicate by `pool_address`.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 24h | 24h, 7d | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "pool_address": "string", "pool_type": "string", "token_x": "string", "token_y": "string", "volume_usd": "number", "event_count": "number", "unique_lps": "number", "volume_change_pct": "number" }] }
```

#### 47. GET /api/v1/meteora/pools/top

Top pools by volume, fees, or LP count.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| sort_by | string | volume | volume, lp_count, events, fees | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "pool_address": "string", "pool_type": "string", "token_x": "string", "token_y": "string", "volume_usd": "number", "event_count": "number", "unique_lps": "number", "total_fees_usd": "number" }] }
```

#### 48. GET /api/v1/meteora/pools/search

Search pools by address or token name.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| query | string | - | pool address or token name | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 10 | 1-50 | no |

```json
{ "data": [{ "pool_address": "string", "pool_type": "string", "token_x": "string", "token_y": "string", "volume_usd": "number", "event_count": "number", "unique_lps": "number" }] }
```

#### 49. GET /api/v1/meteora/pools/detail

Full pool detail.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |

```json
{ "data": { "pool_address": "string", "pool_type": "string", "token_x": "string", "token_y": "string", "total_volume_usd": "number", "total_events": "number", "unique_lps": "number", "volume_by_action": "object", "net_liquidity_usd": "number" } }
```

#### 50. GET /api/v1/meteora/pools/volume-history

Volume time series for a pool.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| bucket_size | string | 4h | 1h, 4h, 1d | no |

```json
{ "data": [{ "bucket": "string", "volume_usd": "number", "event_count": "number", "action_breakdown": "object" }] }
```

#### 51. GET /api/v1/meteora/pools/events

Chronological event feed for a pool.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| limit | integer | 100 | 1-500 | no |

```json
{ "data": [{ "owner": "string", "pool_address": "string", "event_type": "string", "usd_total": "number", "timestamp": "string", "tx_id": "string" }] }
```

#### 52. GET /api/v1/meteora/pools/fee-analysis

Fee claiming analysis for a pool.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

```json
{ "data": { "pool_address": "string", "total_fees_claimed": "number", "unique_claimers": "number", "time_series": "array", "top_claimers": "array" } }
```

#### 63. GET /api/v1/meteora/pools/smart-wallet

Pools with highest smart wallet LP activity.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 20 | 1-100 | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

```json
{ "data": { "pools": [{ "pool_address": "string", "pool_type": "string", "token_a": "string", "token_b": "string", "smart_lp_count": "number", "total_volume_usd": "number", "smart_volume_usd": "number", "smart_share_pct": "number" }] } }
```
