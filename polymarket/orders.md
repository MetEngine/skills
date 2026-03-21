---
skill: polymarket/orders
requires: [core/security, core/trading-patterns, polymarket/overview]
triggers: ["polymarket.*intel", "insider.*poly", "smart money.*poly", "sentiment.*poly", "whale.*trade.*poly", "alpha.*caller"]
level: implementation
---

# Polymarket Intelligence and Trade Endpoints

Smart money analysis, insider detection, sentiment tracking, and whale trade monitoring.

#### 5. POST /api/v1/markets/intelligence

Full smart money intelligence report on a specific market.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| top_n_wallets | integer | 10 | 1-50 | no |

```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "end_date": "string",
    "outcomes": "object",
    "smart_money": {
      "consensus_outcome": "string",
      "consensus_strength": "number",
      "by_outcome": {
        "<outcome>": {
          "wallet_count": "number",
          "total_usdc": "number",
          "percentage": "number",
          "top_wallets": [{
            "wallet": "string",
            "score": "number",
            "tags": ["string"],
            "usdc_invested": "number",
            "net_position": "number",
            "avg_buy_price": "number"
          }]
        }
      }
    },
    "dumb_money": {
      "consensus_outcome": "string",
      "contrarian_to_smart": "boolean",
      "by_outcome": "object"
    },
    "signal_analysis": {
      "smart_vs_price_aligned": "boolean",
      "contrarian_signal": "boolean",
      "signal_summary": "string"
    },
    "recent_activity": {
      "volume_24h": "number",
      "trade_count_24h": "number",
      "buy_sell_ratio": "number",
      "volume_trend": "string"
    }
  }
}
```

#### 7. POST /api/v1/markets/sentiment

Sentiment time series with smart money overlay.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| bucket_size | string | 4h | 1h, 4h, 12h, 1d | no |

```json
{ "data": { "condition_id": "string", "question": "string", "overall_sentiment": "object", "by_outcome": "object", "time_series": "array", "momentum": "object" } }
```

#### 8. POST /api/v1/markets/participants

Participant summary by scoring tier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |

```json
{ "data": { "condition_id": "string", "question": "string", "total_wallets": "number", "total_usdc": "number", "by_outcome": "object", "tier_distribution": "object" } }
```

#### 9. POST /api/v1/markets/insiders

Per-market insider detection with full two-phase 7-signal scoring. Phase 1 uses aggregated data (wallet age, market selection, category concentration) to prune candidates. Phase 2 queries raw trades for survivors (first bet size, bet timing, withdrawal speed, short-bet win rate). Includes summary stats. 60s Redis cache, 10s timeout -- fallback to `/markets/trades` filtered to the market.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| limit | integer | 25 | 1-100 | no |
| min_score | integer | 20 | 0-100 | no |

```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "end_date": "string",
    "insiders": [{
      "wallet": "string",
      "insider_score": "number",
      "signals": {
        "wallet_age": { "score": "number", "days": "number" },
        "first_bet_size": { "score": "number", "usdc": "number" },
        "bet_timing": { "score": "number", "hours_before_end": "number|null" },
        "withdrawal_speed": { "score": "number", "minutes_after_resolution": "number|null" },
        "market_selection": { "score": "number", "is_binary": "boolean", "is_high_volume": "boolean" },
        "category_concentration": { "score": "number", "distinct_categories": "number", "primary_category": "string|null" },
        "short_bet_win_rate": { "score": "number", "win_rate": "number|null", "sample_size": "number" }
      },
      "outcome": "string",
      "net_shares": "number",
      "buy_usdc": "number",
      "wallet_age_days": "number",
      "first_trade_timestamp": "string"
    }],
    "summary": {
      "total_candidates_scanned": "number",
      "phase1_passed": "number",
      "avg_insider_score": "number",
      "highest_score": "number",
      "dominant_outcome": "string|null"
    }
  }
}
```

#### 28. GET /api/v1/markets/insider-markets

Markets grouped by insider activity. Aggregates the global insider cache by condition_id, showing how many insiders are active per market, their total volume, scores, and dominant outcome. Includes top 3 insiders per market as preview. Data from the 10-minute global cache.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| limit | integer | 50 | 1-200 | no |
| offset | integer | 0 | >= 0 | no |
| sort_by | string | insider_count | insider_count, total_insider_volume, highest_score, avg_score | no |
| min_insiders | integer | 2 | 1-100 | no |

```json
{
  "data": {
    "generated_at": "string|null",
    "markets": [{
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "insider_count": "number",
      "total_insider_volume": "number",
      "highest_score": "number",
      "avg_score": "number",
      "dominant_outcome": "string|null",
      "top_insiders": [{
        "wallet": "string",
        "insider_score": "number",
        "outcome": "string",
        "buy_usdc": "number"
      }]
    }],
    "total": "number"
  }
}
```

#### 10. GET /api/v1/markets/trades

Chronological trade feed for a market.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d, 30d | no |
| side | string | - | BUY, SELL | no |
| min_usdc | number | 0 | >= 0 | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 100 | 1-500 | no |

```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "trade_count": "number",
    "total_volume": "number",
    "trades": [{
      "tx_hash": "string",
      "wallet": "string",
      "token_id": "string",
      "outcome": "string",
      "side": "string",
      "price": "number",
      "size": "number",
      "usdc_size": "number",
      "timestamp": "string",
      "wallet_score": "number",
      "wallet_tags": ["string"]
    }]
  }
}
```

#### 15. GET /api/v1/trades/whales

Large whale trades. Returns REDEEM trades alongside real trades -- filter by `side=BUY` or `side=SELL` to exclude.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_usdc | number | 10000 | >= 1 | no |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d, 30d | no |
| condition_id | string | - | - | no |
| category | string | - | any valid category | no |
| side | string | - | BUY, SELL | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 50 | 1-200 | no |

```json
{ "data": [{ "tx_hash": "string", "wallet": "string", "condition_id": "string", "token_id": "string", "question": "string", "outcome": "string", "category": "string", "side": "string", "price": "number", "size": "number", "usdc_size": "number", "timestamp": "string", "wallet_score": "number", "win_rate": "number", "wallet_tags": ["string"] }] }
```

#### 25. GET /api/v1/wallets/alpha-callers

Wallets that trade early on later-trending markets.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| days_back | integer | 30 | 1-90 | no |
| min_days_early | integer | 7 | 1-60 | no |
| min_bet_usdc | number | 100 | >= 0 | no |
| limit | integer | 25 | 1-100 | no |

```json
{ "data": [{ "wallet": "string", "market_question": "string", "condition_id": "string", "entry_date": "string", "days_before_resolution": "number", "bet_size_usdc": "number", "winning_outcome": "string", "wallet_score": "number", "win_rate": "number" }] }
```

#### 27. GET /api/v1/wallets/insiders

Global insider candidates by behavioral score.

| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| limit | integer | 50 | 1-200 | no |
| min_score | integer | 50 | 0-100 | no |
| max_wallet_age_days | integer | 60 | 1-90 | no |

```json
{
  "data": {
    "candidates": [{
      "wallet": "string",
      "insider_score": "number",
      "wallet_age_days": "number",
      "first_trade": "string",
      "markets_traded": "number",
      "total_buy_usdc": "number",
      "category_count": "number",
      "categories": ["string"],
      "win_count": "number",
      "total_resolved": "number",
      "win_rate": "number",
      "signals": "object"
    }],
    "total_candidates": "number"
  }
}
```
