---
skill: meteora/liquidity
requires: [core/security, meteora/overview]
triggers: ["meteora.*lp", "lp.*profile", "lp.*whale", "compare.*lp", "meteora.*position", "position.*history", "active.*position"]
level: implementation
---

# Meteora LP and Position Endpoints

LP profiling, leaderboards, whale detection, comparisons, and position tracking.

#### 53. GET /api/v1/meteora/lps/top

Top LPs leaderboard. **Known issue:** `sort_by=fees` may 500 -- use `sort_by=volume` as fallback.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| sort_by | string | volume | volume, pool_count, events | no |
| timeframe | string | 30d | 7d, 30d | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "owner": "string", "total_volume_usd": "number", "pool_count": "number", "event_count": "number", "pool_types": ["string"] }] }
```

#### 54. POST /api/v1/meteora/lps/profile

Full LP dossier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| owner | string | - | Solana pubkey | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |
| events_limit | integer | 50 | 1-500 | no |

```json
{ "data": { "owner": "string", "summary": "object", "pool_breakdown": "array", "recent_events": "array" } }
```

#### 55. GET /api/v1/meteora/lps/whales

Large LP events.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_type | string | all | dlmm, damm_v2, all | no |
| min_usd | number | 10000 | >= 1 | no |
| timeframe | string | 24h | 24h, 7d, 30d | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": [{ "owner": "string", "pool_address": "string", "pool_type": "string", "event_type": "string", "usd_total": "number", "timestamp": "string" }] }
```

#### 56. POST /api/v1/meteora/lps/compare

Compare 2-5 LPs. **Pricing: `base * owners.length / 2`.**

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| owners | string[] | - | 2-5 Solana pubkeys | yes |
| pool_type | string | all | dlmm, damm_v2, all | no |

```json
{ "data": { "lps": "array of LP profiles", "shared_pools": "array", "comparison_winners": "object" } }
```

#### 57. GET /api/v1/meteora/positions/active

Active LP positions. Provide `pool_address` or `owner` (or both) to filter.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| pool_address | string | - | Solana pubkey | no |
| owner | string | - | Solana pubkey | no |
| pool_type | string | all | dlmm, damm_v2, all | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": [{ "owner": "string", "pool_address": "string", "pool_type": "string", "position_address": "string", "deposited_usd": "number", "withdrawn_usd": "number", "net_value_usd": "number", "event_count": "number" }] }
```

Note: `net_value_usd` = deposits minus withdrawals, NOT current market value. Position addresses only exist for DLMM pools.

#### 58. GET /api/v1/meteora/positions/history

Position event history. DLMM only -- DAMM v2 positions do not have position addresses.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| position_address | string | - | - | yes |

```json
{ "data": { "position_address": "string", "events": [{ "event_type": "string", "usd_total": "number", "timestamp": "string", "tx_id": "string" }] } }
```
