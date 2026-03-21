---
skill: meteora/overview
requires: [core/security]
triggers: ["meteora", "pool", "dlmm", "damm", "lp", "apr", "impermanent loss"]
level: concept
---

# Meteora Overview

Solana LP/AMM analytics for Meteora DLMM and DAMM v2 pools. 18 endpoints covering pool discovery, LP profiling, whale event tracking, fee analysis, position history, and DCA pressure. MetEngine indexes all on-chain events with sub-minute latency.

## Endpoint Summary (18 endpoints)

| # | Method | Path | Tier | Description |
|---|--------|------|------|-------------|
| 46 | GET | /api/v1/meteora/pools/trending | medium | Trending pools by volume spike |
| 47 | GET | /api/v1/meteora/pools/top | medium | Top pools by volume/fees/LP count |
| 48 | GET | /api/v1/meteora/pools/search | light | Search by address or token name |
| 49 | GET | /api/v1/meteora/pools/detail | medium | Full pool detail |
| 50 | GET | /api/v1/meteora/pools/volume-history | light | Volume time series |
| 51 | GET | /api/v1/meteora/pools/events | light | Chronological event feed |
| 52 | GET | /api/v1/meteora/pools/fee-analysis | medium | Fee claiming analysis |
| 53 | GET | /api/v1/meteora/lps/top | heavy | Top LPs leaderboard |
| 54 | POST | /api/v1/meteora/lps/profile | heavy | Full LP dossier |
| 55 | GET | /api/v1/meteora/lps/whales | medium | Large LP events |
| 56 | POST | /api/v1/meteora/lps/compare | whale | Compare 2-5 LPs |
| 57 | GET | /api/v1/meteora/positions/active | medium | Active LP positions |
| 58 | GET | /api/v1/meteora/positions/history | light | Position event history (DLMM only) |
| 59 | GET | /api/v1/meteora/platform/stats | light | Platform-wide stats |
| 60 | GET | /api/v1/meteora/platform/volume-heatmap | medium | Volume by action/hour |
| 61 | GET | /api/v1/meteora/platform/metengine-share | light | MetEngine routing share |
| 62 | GET | /api/v1/meteora/dca/pressure | medium | DCA accumulation pressure by token |
| 63 | GET | /api/v1/meteora/pools/smart-wallet | heavy | Pools with smart wallet LP activity |

## Pool Types

| Type | Token Fields | Event Type Format | Position Addresses |
|------|-------------|-------------------|-------------------|
| DLMM | `token_x` / `token_y` | PascalCase: `AddLiquidity`, `RemoveLiquidity`, `Swap`, `ClaimFee` | Yes |
| DAMM v2 | `token_a` / `token_b` | snake_case: `add_liquidity`, `remove_liquidity`, `swap`, `claim_fee` | No |

## Scoring System (LP Smart Score 0-100)

```
win_ratio (0-35 pts) + volume (0-25 pts, log10-scaled) + avg_pnl% (0-30 pts) + experience (0-10 pts)
```

Minimum 3 closed positions required for scoring.

## Address Format

- Solana pubkeys (base58, case-sensitive).

## Known Quirks

- DAMM v2 pools frequently show implausible fee rates (30-50% of volume) due to anti-sniper fee scheduler on new token launches. Separate DAMM v2 from DLMM results and flag anomalies.
- `/meteora/pools/trending` may have duplicate entries. Deduplicate by `pool_address`.
- `/meteora/lps/top?sort_by=fees` may 500. Fallback: `sort_by=volume`.
- Some token mints show as "???" (unresolved). Check pair context for identification.
- Position addresses only exist for DLMM, not DAMM v2.
- Net value = deposits minus withdrawals (not current market value).
