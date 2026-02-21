# Meteora Endpoint Reference

## Scope

Detailed endpoint contracts for Meteora routes.

## How To Use

- Search by path with `rg "GET /api/v1/meteora" references/meteora-endpoints.md`.
- Load only the endpoint section needed for the active query.

## TOC

- Endpoint list starts at the `####` headings below.

### METEORA

#### 46. GET /api/v1/meteora/pools/trending

Trending pools by volume spike.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 24h | 24h, 7d | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "pool_address": "string",
      "pool_type": "string",
      "token_x": "string",
      "token_y": "string",
      "volume_usd": "number",
      "event_count": "number",
      "unique_lps": "number",
      "volume_change_pct": "number"
    }
  ]
}
```

**Known issue:** May contain duplicate entries. Deduplicate by `pool_address`.

#### 47. GET /api/v1/meteora/pools/top

Top pools by volume, fees, or LP count.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| sort_by | string | volume | volume, lp_count, events, fees | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "pool_address": "string",
      "pool_type": "string",
      "token_x": "string",
      "token_y": "string",
      "volume_usd": "number",
      "event_count": "number",
      "unique_lps": "number",
      "total_fees_usd": "number"
    }
  ]
}
```

#### 48. GET /api/v1/meteora/pools/search

Search pools by address or token name.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| query | string | - | pool address or token name | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 10 | 1-50 | no |

**Response schema:**
```json
{
  "data": [
    {
      "pool_address": "string",
      "pool_type": "string",
      "token_x": "string",
      "token_y": "string",
      "volume_usd": "number",
      "event_count": "number",
      "unique_lps": "number"
    }
  ]
}
```

#### 49. GET /api/v1/meteora/pools/detail

Full pool detail.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |

**Response schema:**
```json
{
  "data": {
    "pool_address": "string",
    "pool_type": "string",
    "token_x": "string",
    "token_y": "string",
    "total_volume_usd": "number",
    "total_events": "number",
    "unique_lps": "number",
    "volume_by_action": "object",
    "net_liquidity_usd": "number"
  }
}
```

#### 50. GET /api/v1/meteora/pools/volume-history

Volume time series.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| bucket_size | string | 4h | 1h, 4h, 1d | no |

**Response schema:**
```json
{
  "data": [
    {
      "bucket": "string",
      "volume_usd": "number",
      "event_count": "number",
      "action_breakdown": "object"
    }
  ]
}
```

#### 51. GET /api/v1/meteora/pools/events

Chronological event feed.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| limit | integer | 100 | 1-500 | no |

**Response schema:**
```json
{
  "data": [
    {
      "owner": "string",
      "pool_address": "string",
      "event_type": "string",
      "usd_total": "number",
      "timestamp": "string",
      "tx_id": "string"
    }
  ]
}
```

#### 52. GET /api/v1/meteora/pools/fee-analysis

Fee claiming analysis.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | yes |
| pool_type | string | - | dlmm, damm_v2 (auto-detected if omitted) | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

**Response schema:**
```json
{
  "data": {
    "pool_address": "string",
    "total_fees_claimed": "number",
    "unique_claimers": "number",
    "time_series": "array",
    "top_claimers": "array"
  }
}
```

#### 53. GET /api/v1/meteora/lps/top

Top LPs leaderboard.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| sort_by | string | volume | volume, pool_count, events | no |
| timeframe | string | 30d | 7d, 30d | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "owner": "string",
      "total_volume_usd": "number",
      "pool_count": "number",
      "event_count": "number",
      "pool_types": ["string"]
    }
  ]
}
```

**Known issue:** `sort_by=fees` may return 500. Use `sort_by=volume` as fallback.

#### 54. POST /api/v1/meteora/lps/profile

Full LP dossier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| owner | string | - | Solana pubkey | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |
| events_limit | integer | 50 | 1-500 | no |

**Response schema:**
```json
{
  "data": {
    "owner": "string",
    "summary": "object",
    "pool_breakdown": "array",
    "recent_events": "array"
  }
}
```

#### 55. GET /api/v1/meteora/lps/whales

Large LP events.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| min_usd | number | 10000 | >= 1 | no |
| timeframe | string | 24h | 24h, 7d, 30d | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": [
    {
      "owner": "string",
      "pool_address": "string",
      "pool_type": "string",
      "event_type": "string",
      "usd_total": "number",
      "timestamp": "string"
    }
  ]
}
```

#### 56. POST /api/v1/meteora/lps/compare

Compare 2-5 LPs.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| owners | string[] | - | 2-5 Solana pubkeys | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |

**Response schema:**
```json
{
  "data": {
    "lps": "array of LP profiles",
    "shared_pools": "array",
    "comparison_winners": "object"
  }
}
```

**Pricing note:** Cost scales with LP count: `base * owners.length / 2`.

#### 57. GET /api/v1/meteora/positions/active

Active LP positions.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | no |
| owner | string | - | Solana pubkey | no |
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": [
    {
      "owner": "string",
      "pool_address": "string",
      "pool_type": "string",
      "position_address": "string",
      "deposited_usd": "number",
      "withdrawn_usd": "number",
      "net_value_usd": "number",
      "event_count": "number"
    }
  ]
}
```

#### 58. GET /api/v1/meteora/positions/history

Position event history (DLMM only).

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| position_address | string | - | - | yes |

**Response schema:**
```json
{
  "data": {
    "position_address": "string",
    "events": [
      {
        "event_type": "string",
        "usd_total": "number",
        "timestamp": "string",
        "tx_id": "string"
      }
    ]
  }
}
```

#### 59. GET /api/v1/meteora/platform/stats

Platform-wide stats.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |

**Response schema:**
```json
{
  "data": {
    "total_volume_usd": "number",
    "total_events": "number",
    "active_lps": "number",
    "active_pools": "number",
    "by_pool_type": "object"
  }
}
```

#### 60. GET /api/v1/meteora/platform/volume-heatmap

Volume by action/hour.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 24h | 24h, 7d | no |

**Response schema:**
```json
{
  "data": [
    {
      "action": "string",
      "bucket": "string",
      "pool_type": "string",
      "volume_usd": "number",
      "event_count": "number"
    }
  ]
}
```

#### 61. GET /api/v1/meteora/platform/metengine-share

MetEngine routing share.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

**Response schema:**
```json
{
  "data": {
    "total_events": "number",
    "metengine_events": "number",
    "metengine_share_pct": "number",
    "total_volume_usd": "number",
    "metengine_volume_usd": "number"
  }
}
```

#### 62. GET /api/v1/meteora/dca/pressure

DCA accumulation pressure by token.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| token | string | - | specific mint address | no |
| timeframe | string | 1h | 1m, 5m, 30m, 1h, 6h, 24h | no |

**Response schema (single token):**
```json
{
  "data": {
    "tokenMint": "string",
    "timeframe": "string",
    "buyVolume": "number",
    "sellVolume": "number",
    "netPressure": "number",
    "score": "number"
  }
}
```

**Response schema (all tokens, when `token` omitted):**
```json
{
  "data": {
    "timeframe": "string",
    "tokens": "array",
    "lastUpdated": "string"
  }
}
```

#### 63. GET /api/v1/meteora/pools/smart-wallet

Pools with highest smart wallet LP activity.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 20 | 1-100 | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |

**Response schema:**
```json
{
  "data": {
    "pools": [
      {
        "pool_address": "string",
        "pool_type": "string",
        "token_a": "string",
        "token_b": "string",
        "smart_lp_count": "number",
        "total_volume_usd": "number",
        "smart_volume_usd": "number",
        "smart_share_pct": "number"
      }
    ]
  }
}
```

---

