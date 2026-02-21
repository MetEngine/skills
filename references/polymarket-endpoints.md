# Polymarket Endpoint Reference

## Scope

Detailed endpoint contracts for Polymarket routes.

## How To Use

- Search by path with `rg "GET /api/v1/..." references/polymarket-endpoints.md`.
- Load only the endpoint section needed for the active query.

## TOC

- Endpoint list starts at the `####` headings below.

### POLYMARKET

#### 1. GET /api/v1/markets/trending

Trending markets with volume spikes.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d | no |
| category | string | - | any valid category | no |
| sort_by | string | volume_spike | volume_spike, trade_count, smart_money_inflow | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "period_volume_usdc": "number",
      "period_trade_count": "number",
      "volume_spike_multiplier": "number",
      "smart_money_wallets_active": "number",
      "smart_money_net_direction": "string",
      "buy_sell_ratio": "number",
      "leader_price": "number"
    }
  ]
}
```

#### 2. GET /api/v1/markets/search

Search markets by keyword, category, status. Accepts Polymarket URLs as the `query` param.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| query | string | - | keyword or Polymarket URL | no |
| category | string | - | any valid category | no |
| status | string | active | active, closing_soon, resolved | no |
| has_smart_money_signal | boolean | - | true, false | no |
| sort_by | string | relevance | end_date, volume, relevance | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "end_date": "string (ISO 8601)",
      "status": "string",
      "total_volume_usdc": "number",
      "smart_money_outcome": "string | null",
      "smart_money_wallets": "number",
      "has_contrarian_signal": "boolean",
      "leader_price": "number"
    }
  ]
}
```

#### 3. GET /api/v1/markets/categories

List all categories with activity stats.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| include_stats | boolean | true | true, false | no |
| timeframe | string | 24h | 24h, 7d | no |

**Response schema:**
```json
{
  "data": [
    {
      "name": "string",
      "active_markets": "number",
      "period_volume": "number",
      "period_trades": "number",
      "unique_traders": "number"
    }
  ]
}
```

#### 4. GET /api/v1/platform/stats

Platform-wide aggregate stats.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d, 30d | no |

**Response schema:**
```json
{
  "data": {
    "timeframe": "string",
    "total_volume_usdc": "number",
    "total_trades": "number",
    "active_traders": "number",
    "active_markets": "number",
    "resolved_markets": "number",
    "smart_wallet_count": "number",
    "avg_trade_size_usdc": "number"
  }
}
```

#### 5. POST /api/v1/markets/intelligence

Full smart money intelligence report on a specific market.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| top_n_wallets | integer | 10 | 1-50 | no |

**Response schema:**
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
        "<outcome_name>": {
          "wallet_count": "number",
          "total_usdc": "number",
          "percentage": "number",
          "top_wallets": [
            {
              "wallet": "string",
              "score": "number",
              "tags": ["string"],
              "usdc_invested": "number",
              "net_position": "number",
              "avg_buy_price": "number"
            }
          ]
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

#### 6. GET /api/v1/markets/price-history

OHLCV price/probability time series.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| timeframe | string | 7d | 1h, 4h, 12h, 24h, 7d, 30d | no |
| bucket_size | string | 1h | 5m, 15m, 1h, 4h, 12h, 1d | no |

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "candles_by_outcome": {
      "<outcome_name>": [
        {
          "bucket": "string (ISO 8601)",
          "token_id": "string",
          "outcome": "string",
          "open": "number",
          "high": "number",
          "low": "number",
          "close": "number",
          "volume": "number",
          "trade_count": "number",
          "vwap": "number"
        }
      ]
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

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "overall_sentiment": "object",
    "by_outcome": "object",
    "time_series": "array",
    "momentum": "object"
  }
}
```

#### 8. POST /api/v1/markets/participants

Participant summary by scoring tier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "total_wallets": "number",
    "total_usdc": "number",
    "by_outcome": "object",
    "tier_distribution": "object"
  }
}
```

#### 9. POST /api/v1/markets/insiders

Insider pattern detection using 7-signal behavioral scoring.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| limit | integer | 25 | 1-100 | no |
| min_score | integer | 20 | 0-100 | no |

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "insiders": [
      {
        "wallet": "string",
        "insider_score": "number",
        "signals": "object",
        "outcome": "string",
        "net_shares": "number",
        "buy_usdc": "number",
        "wallet_age_days": "number",
        "first_trade_timestamp": "string"
      }
    ],
    "summary": "object"
  }
}
```

#### 10. GET /api/v1/markets/trades

Chronological trade feed for a market.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d, 30d | no |
| side | string | - | BUY, SELL | no |
| min_usdc | number | 0 | >= 0 | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 100 | 1-500 | no |

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "trade_count": "number",
    "total_volume": "number",
    "trades": [
      {
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
      }
    ]
  }
}
```

#### 11. GET /api/v1/markets/similar

Related markets by wallet overlap.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| limit | integer | 10 | 1-50 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "shared_wallet_count": "number",
      "wallet_overlap_pct": "number",
      "total_volume_usdc": "number"
    }
  ]
}
```

#### 12. GET /api/v1/markets/opportunities

Markets where smart money disagrees with price.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_signal_strength | string | moderate | weak, moderate, strong | no |
| category | string | - | any valid category | no |
| closing_within_hours | integer | - | max 720 | no |
| min_smart_wallets | integer | 3 | >= 1 | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "smart_money_favors": "string",
      "smart_money_percentage": "number",
      "smart_wallet_count": "number",
      "avg_smart_score": "number",
      "price_signal_gap": "number",
      "signal_direction": "string",
      "signal_strength": "string",
      "leader_price": "number",
      "time_until_close": "string"
    }
  ]
}
```

#### 13. GET /api/v1/markets/high-conviction

High-conviction smart money bets.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | no |
| min_smart_wallets | integer | 5 | >= 1 | no |
| min_avg_score | integer | 65 | 0-100 | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "smart_wallet_count": "number",
      "avg_smart_score": "number",
      "total_smart_usdc": "number",
      "conviction_score": "number",
      "favored_outcome": "string",
      "favored_outcome_percentage": "number",
      "entry_vs_current_gap": "number"
    }
  ]
}
```

#### 14. GET /api/v1/markets/capital-flow

Capital flow by category (sector rotation).

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 7d | 24h, 7d, 30d | no |
| smart_money_only | boolean | false | true, false | no |
| top_n_categories | integer | 20 | 1-50 | no |

**Response schema:**
```json
{
  "data": {
    "timeframe": "string",
    "total_net_flow": "number",
    "categories": [
      {
        "category": "string",
        "current_buy_volume": "number",
        "current_sell_volume": "number",
        "current_net_flow": "number",
        "previous_buy_volume": "number",
        "previous_sell_volume": "number",
        "previous_net_flow": "number",
        "net_flow_change": "number",
        "flow_trend": "string"
      }
    ],
    "biggest_inflow": "string",
    "biggest_outflow": "string"
  }
}
```

#### 15. GET /api/v1/trades/whales

Large whale trades.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| min_usdc | number | 10000 | >= 1 | no |
| timeframe | string | 24h | 1h, 4h, 12h, 24h, 7d, 30d | no |
| condition_id | string | - | - | no |
| category | string | - | any valid category | no |
| side | string | - | BUY, SELL | no |
| smart_money_only | boolean | false | true, false | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": [
    {
      "tx_hash": "string",
      "wallet": "string",
      "condition_id": "string",
      "token_id": "string",
      "question": "string",
      "outcome": "string",
      "category": "string",
      "side": "string",
      "price": "number",
      "size": "number",
      "usdc_size": "number",
      "timestamp": "string",
      "wallet_score": "number",
      "win_rate": "number",
      "wallet_tags": ["string"]
    }
  ]
}
```

**Note:** This endpoint returns REDEEM trades (resolved market payouts) alongside real trades. Filter by `side=BUY` or `side=SELL` to exclude them. REDEEMs have `price=1.00` and `side=REDEEM`.

#### 16. GET /api/v1/markets/volume-heatmap

Volume distribution across categories, hours, or days.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 24h | 24h, 7d, 30d | no |
| group_by | string | category | category, hour_of_day, day_of_week | no |
| smart_money_only | boolean | false | true, false | no |

**Response schema:**
```json
{
  "data": {
    "total_volume": "number",
    "total_trades": "number",
    "breakdown": [
      {
        "label": "string",
        "volume": "number",
        "trade_count": "number",
        "pct_of_total": "number",
        "volume_change_pct": "number",
        "trend": "string"
      }
    ],
    "biggest_gainer": "string",
    "biggest_loser": "string"
  }
}
```

#### 17. POST /api/v1/wallets/profile

Full wallet dossier.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| include_positions | boolean | true | true, false | no |
| include_trades | boolean | true | true, false | no |
| trades_limit | integer | 50 | 1-500 | no |

**Response schema:**
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

**Note:** Wallet addresses MUST be lowercase.

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

**Response schema:**
```json
{
  "data": {
    "wallet": "string",
    "wallet_score": "number",
    "wallet_tags": ["string"],
    "period_summary": "object",
    "trades": "array"
  }
}
```

#### 19. POST /api/v1/wallets/pnl-breakdown

Per-market PnL breakdown.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| timeframe | string | 90d | 7d, 30d, 90d, all | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": {
    "wallet": "string",
    "total_realized_pnl": "number",
    "total_positions": "number",
    "winning_positions": "number",
    "losing_positions": "number",
    "best_trade": "object",
    "worst_trade": "object",
    "positions": "array"
  }
}
```

#### 20. POST /api/v1/wallets/compare

Compare 2-5 wallets side-by-side.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallets | string[] | - | 2-5 lowercase addresses | yes |
| include_shared_positions | boolean | true | true, false | no |

**Response schema:**
```json
{
  "data": {
    "wallets": "array of wallet profiles",
    "comparison_winners": "object",
    "category_overlap": "object",
    "shared_positions": "array"
  }
}
```

**Pricing note:** Cost scales with wallet count: `base * wallets.length / 2`.

#### 21. POST /api/v1/wallets/copy-traders

Detect wallets copying a target wallet.

**Request body (JSON):**
| Field | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| wallet | string | - | lowercase address | yes |
| max_lag_minutes | integer | 60 | 1-1440 | no |
| timeframe | string | 7d | 24h, 7d, 30d | no |
| min_overlap_trades | integer | 3 | >= 1 | no |
| limit | integer | 20 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "wallet": "string",
      "overlap_trades": "number",
      "avg_lag_seconds": "number",
      "copied_tokens": "array",
      "wallet_score": "number"
    }
  ]
}
```

#### 22. GET /api/v1/wallets/top-performers

Leaderboard.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| timeframe | string | 7d | today, 24h, 7d, 30d, 90d, 365d | no |
| category | string | - | any valid category | no |
| metric | string | pnl | pnl, roi, sharpe, win_rate, volume | no |
| min_trades | integer | 5 | >= 1 | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "rank": "number",
      "wallet": "string",
      "period_pnl_usdc": "number",
      "period_roi_percent": "number",
      "period_trades": "number",
      "period_win_rate": "number",
      "overall_score": "number",
      "sharpe_ratio": "number",
      "primary_category": "string",
      "tags": ["string"],
      "active_positions_count": "number"
    }
  ]
}
```

**Pricing note:** Queries without a `category` filter cost 2x due to full-table scan.

#### 23. GET /api/v1/wallets/niche-experts

Top wallets in a specific category.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | yes |
| min_category_trades | integer | 10 | >= 1 | no |
| sort_by | string | category_sharpe | category_sharpe, category_pnl, category_volume | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "wallet": "string",
      "category_sharpe": "number",
      "category_win_rate": "number",
      "category_volume": "number",
      "category_pnl": "number",
      "category_positions": "number",
      "is_specialist": "boolean",
      "volume_concentration": "number",
      "overall_score": "number",
      "tags": ["string"]
    }
  ]
}
```

#### 24. GET /api/v1/markets/resolutions

Resolved markets with smart money accuracy.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| category | string | - | any valid category | no |
| query | string | - | keyword | no |
| sort_by | string | resolved_recently | resolved_recently, volume, smart_money_accuracy | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "condition_id": "string",
      "question": "string",
      "category": "string",
      "winning_outcome": "string",
      "end_date": "string",
      "total_volume_usdc": "number",
      "smart_money_correct": "number",
      "smart_money_wrong": "number",
      "smart_money_accuracy": "number"
    }
  ]
}
```

#### 25. GET /api/v1/wallets/alpha-callers

Wallets that trade early on later-trending markets.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| days_back | integer | 30 | 1-90 | no |
| min_days_early | integer | 7 | 1-60 | no |
| min_bet_usdc | number | 100 | >= 0 | no |
| limit | integer | 25 | 1-100 | no |

**Response schema:**
```json
{
  "data": [
    {
      "wallet": "string",
      "market_question": "string",
      "condition_id": "string",
      "entry_date": "string",
      "days_before_resolution": "number",
      "bet_size_usdc": "number",
      "winning_outcome": "string",
      "wallet_score": "number",
      "win_rate": "number"
    }
  ]
}
```

#### 26. GET /api/v1/markets/dumb-money

Low-score trader positions on a market (contrarian indicator).

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| condition_id | string | - | - | yes |
| max_score | integer | 30 | 0-100 | no |
| min_trades | integer | 5 | >= 1 | no |
| limit | integer | 50 | 1-200 | no |

**Response schema:**
```json
{
  "data": {
    "condition_id": "string",
    "question": "string",
    "category": "string",
    "summary": {
      "total_wallets": "number",
      "total_usdc": "number",
      "avg_score": "number",
      "consensus_outcome": "string"
    },
    "positions": "array"
  }
}
```

#### 27. GET /api/v1/wallets/insiders

Global insider candidates by behavioral score.

**Parameters (query string):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| limit | integer | 50 | 1-200 | no |
| min_score | integer | 50 | 0-100 | no |
| max_wallet_age_days | integer | 60 | 1-90 | no |

**Response schema:**
```json
{
  "data": {
    "candidates": [
      {
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
      }
    ],
    "total_candidates": "number"
  }
}
```

---

