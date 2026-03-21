---
skill: polymarket/positions
requires: [core/security, polymarket/overview]
triggers: ["polymarket.*wallet", "wallet.*profile", "pnl.*poly", "top.*performer", "niche.*expert", "copy.*trader", "compare.*wallet"]
level: implementation
---

# Polymarket Wallet and Position Endpoints

Wallet profiling, PnL analysis, leaderboards, and copy-trader detection.

#### 17. POST /api/v1/wallets/profile

Full wallet dossier with score, positions, and trades. Wallet addresses MUST be lowercase.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| include_positions | boolean | true | true, false | no |
| include_trades | boolean | true | true, false | no |
| trades_limit | integer | 50 | 1-500 | no |

**Pricing note:** both includes=1.0x, one include=0.7x, neither=0.4x.

```json
{
  "data": {
    "wallet": "string",
    "profile": {
      "score": "number",
      "sharpe": "number",
      "win_rate": "number",
      "total_pnl": "number",
      "total_volume": "number",
      "resolved_positions": "number",
      "tags": ["string"],
      "tier": "string",
      "primary_category": "string",
      "is_specialist": "boolean"
    },
    "category_breakdown": "array",
    "active_positions": "array",
    "recent_trades": "array"
  }
}
```

#### 18. POST /api/v1/wallets/activity

Recent trading activity for a wallet.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| timeframe | string | 24h | 1h, 4h, 24h, 7d, 30d | no |
| category | string | - | any valid category | no |
| min_usdc | number | 0 | >= 0 | no |
| limit | integer | 100 | 1-500 | no |

```json
{ "data": { "wallet": "string", "wallet_score": "number", "wallet_tags": ["string"], "period_summary": "object", "trades": "array" } }
```

#### 19. POST /api/v1/wallets/pnl-breakdown

Per-market PnL breakdown.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| timeframe | string | 90d | 7d, 30d, 90d, all | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": { "wallet": "string", "total_realized_pnl": "number", "total_positions": "number", "winning_positions": "number", "losing_positions": "number", "best_trade": "object", "worst_trade": "object", "positions": "array" } }
```

#### 20. POST /api/v1/wallets/compare

Compare 2-5 wallets side-by-side. **Pricing: `base * wallets.length / 2`.**

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallets | string[] | - | 2-5 lowercase addresses | yes |
| include_shared_positions | boolean | true | true, false | no |

```json
{ "data": { "wallets": "array of wallet profiles", "comparison_winners": "object", "category_overlap": "object", "shared_positions": "array" } }
```

#### 21. POST /api/v1/wallets/copy-traders

Detect wallets copying a target wallet. **Hard cap: $0.12.**

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| max_lag_minutes | integer | 60 | 1-1440 | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| min_overlap_trades | integer | 3 | >= 1 | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "wallet": "string", "overlap_trades": "number", "avg_lag_seconds": "number", "copied_tokens": "array", "wallet_score": "number" }] }
```

#### 22. GET /api/v1/wallets/top-performers

Leaderboard. **Pricing: queries without `category` filter cost 2x.**

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 7d | today, 24h, 7d, 30d, 90d, 365d | no |
| category | string | - | any valid category | no |
| metric | string | pnl | pnl, roi, sharpe, win_rate, volume | no |
| min_trades | integer | 5 | >= 1 | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "rank": "number", "wallet": "string", "period_pnl_usdc": "number", "period_roi_percent": "number", "period_trades": "number", "period_win_rate": "number", "overall_score": "number", "sharpe_ratio": "number", "primary_category": "string", "tags": ["string"], "active_positions_count": "number" }] }
```

#### 23. GET /api/v1/wallets/niche-experts

Top wallets in a specific category.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | yes |
| min_category_trades | integer | 10 | >= 1 | no |
| sort_by | string | category_sharpe | category_sharpe, category_pnl, category_volume | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "wallet": "string", "category_sharpe": "number", "category_win_rate": "number", "category_volume": "number", "category_pnl": "number", "category_positions": "number", "is_specialist": "boolean", "volume_concentration": "number", "overall_score": "number", "tags": ["string"] }] }
```
