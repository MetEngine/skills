---
skill: polymarket/markets
requires: [core/security, polymarket/overview]
triggers: ["polymarket.*market", "trending.*market", "search.*market", "capital.*flow", "volume.*heatmap", "resolution", "category"]
level: implementation
---

# Polymarket Market Endpoints

Market discovery, search, trending, capital flow, and resolution tracking.

#### 1. GET /api/v1/markets/trending

Trending markets with volume spikes.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| category | string | - | any valid category | no |
| sort_by | string | volume_spike | volume_spike, trade_count, smart_money_inflow | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "period_volume_usdc": "number", "period_trade_count": "number", "volume_spike_multiplier": "number", "smart_money_wallets_active": "number", "smart_money_net_direction": "string", "buy_sell_ratio": "number", "leader_price": "number" }] }
```

#### 2. GET /api/v1/markets/search

Search by keyword, category, status, or Polymarket URL.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| query | string | - | keyword or Polymarket URL | no |
| category | string | - | any valid category | no |
| status | string | active | active, closing_soon, resolved | no |
| has_smart_money_signal | boolean | - | true, false | no |
| sort_by | string | relevance | end_date, volume, relevance | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "end_date": "string", "status": "string", "total_volume_usdc": "number", "smart_money_outcome": "string | null", "smart_money_wallets": "number", "has_contrarian_signal": "boolean", "leader_price": "number" }] }
```

#### 3. GET /api/v1/markets/categories

All categories with activity stats.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| include_stats | boolean | true | true, false | no |
| timeframe | string | 24h | 24h, 7d | no |

```json
{ "data": [{ "name": "string", "active_markets": "number", "period_volume": "number", "period_trades": "number", "unique_traders": "number" }] }
```

#### 4. GET /api/v1/platform/stats

Platform-wide aggregate stats.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d, 30d | no |

```json
{ "data": { "timeframe": "string", "total_volume_usdc": "number", "total_trades": "number", "active_traders": "number", "active_markets": "number", "resolved_markets": "number", "smart_wallet_count": "number", "avg_trade_size_usdc": "number" } }
```

#### 6. GET /api/v1/markets/closing-soon

Markets closing within a specified time window. Useful for finding time-sensitive trading opportunities.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| hours | number | 24 | 1-72 | no |
| category | string | - | any valid category | no |
| limit | number | 20 | 1-100 | no |
| offset | number | 0 | - | no |
| sort_by | string | end_date | end_date, category | no |
| sort_order | string | asc | asc, desc | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "slug": "string", "end_date": "ISO 8601", "hours_until_close": "number", "category": "string" }] }
```

#### 7. GET /api/v1/markets/price-history

OHLCV price/probability time series.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| timeframe | string | 7d | 1h, 4h, 12h, 24h, 7d, 30d | no |
| bucket_size | string | 1h | 5m, 15m, 1h, 4h, 12h, 1d | no |

```json
{ "data": { "condition_id": "string", "question": "string", "category": "string", "candles_by_outcome": { "<outcome>": [{ "bucket": "string", "token_id": "string", "outcome": "string", "open": "number", "high": "number", "low": "number", "close": "number", "volume": "number", "trade_count": "number", "vwap": "number" }] } } }
```

#### 11. GET /api/v1/markets/similar

Related markets by wallet overlap.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| limit | integer | 10 | 1-50 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "shared_wallet_count": "number", "wallet_overlap_pct": "number", "total_volume_usdc": "number" }] }
```

#### 12. GET /api/v1/markets/opportunities

Markets where smart money disagrees with price. **Hard cap: $0.15.** Timeout-prone -- fallback to `/markets/high-conviction`.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_signal_strength | string | moderate | weak, moderate, strong | no |
| category | string | - | any valid category | no |
| closing_within_hours | integer | - | max 720 | no |
| min_smart_wallets | integer | 3 | >= 1 | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "smart_money_favors": "string", "smart_money_percentage": "number", "smart_wallet_count": "number", "avg_smart_score": "number", "price_signal_gap": "number", "signal_direction": "string", "signal_strength": "string", "leader_price": "number", "time_until_close": "string" }] }
```

#### 13. GET /api/v1/markets/high-conviction

High-conviction smart money bets.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | no |
| min_smart_wallets | integer | 5 | >= 1 | no |
| min_avg_score | integer | 65 | 0-100 | no |
| limit | integer | 20 | 1-100 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "smart_wallet_count": "number", "avg_smart_score": "number", "total_smart_usdc": "number", "conviction_score": "number", "favored_outcome": "string", "favored_outcome_percentage": "number", "entry_vs_current_gap": "number" }] }
```

#### 14. GET /api/v1/markets/capital-flow

Capital flow by category (sector rotation).

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 7d | 24h, 7d, 30d | no |
| smart_money_only | boolean | false | true, false | no |
| top_n_categories | integer | 20 | 1-50 | no |

```json
{ "data": { "timeframe": "string", "total_net_flow": "number", "categories": [{ "category": "string", "current_buy_volume": "number", "current_sell_volume": "number", "current_net_flow": "number", "previous_net_flow": "number", "net_flow_change": "number", "flow_trend": "string" }], "biggest_inflow": "string", "biggest_outflow": "string" } }
```

#### 16. GET /api/v1/markets/volume-heatmap

Volume distribution by category, hour, or day.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d, 30d | no |
| group_by | string | category | category, hour_of_day, day_of_week | no |
| smart_money_only | boolean | false | true, false | no |

```json
{ "data": { "total_volume": "number", "total_trades": "number", "breakdown": [{ "label": "string", "volume": "number", "trade_count": "number", "pct_of_total": "number", "volume_change_pct": "number", "trend": "string" }], "biggest_gainer": "string", "biggest_loser": "string" } }
```

#### 24. GET /api/v1/markets/resolutions

Resolved markets with smart money accuracy tracking.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | no |
| query | string | - | keyword | no |
| sort_by | string | resolved_recently | resolved_recently, volume, smart_money_accuracy | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "condition_id": "string", "question": "string", "category": "string", "winning_outcome": "string", "end_date": "string", "total_volume_usdc": "number", "smart_money_correct": "number", "smart_money_wrong": "number", "smart_money_accuracy": "number" }] }
```

#### 26. GET /api/v1/markets/dumb-money

Low-score trader positions (contrarian indicator).

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| max_score | integer | 30 | 0-100 | no |
| min_trades | integer | 5 | >= 1 | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": { "condition_id": "string", "question": "string", "category": "string", "summary": { "total_wallets": "number", "total_usdc": "number", "avg_score": "number", "consensus_outcome": "string" }, "positions": "array" } }
```
