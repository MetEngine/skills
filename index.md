---
skill: router
level: routing
---

# MetEngine Skill Graph Router

Match user intent to the right skill file. Load the `requires` chain, then the matched file.

## Routing Table

| Trigger Pattern | Skill File | Requires |
|----------------|------------|----------|
| `polymarket.*market`, `condition_id`, `prediction`, `outcome` | polymarket/markets | core/security, polymarket/overview |
| `polymarket.*intel`, `insider.*poly`, `smart money.*poly`, `sentiment` | polymarket/orders | core/security, core/trading-patterns, polymarket/overview |
| `polymarket.*wallet`, `wallet.*profile`, `pnl.*poly`, `top.*performer` | polymarket/positions | core/security, polymarket/overview |
| `proxy.*wallet`, `polymarket.*trade`, `clob.*order`, `magic.*link` | polymarket/proxy-wallet | core/security, polymarket/overview |
| `hl.*coin`, `hyperliquid.*trending`, `perp.*data`, `funding` | hyperliquid/market-data | core/security, hyperliquid/overview |
| `hl.*trade`, `whale.*hl`, `long.*short`, `pressure` | hyperliquid/trading | core/security, core/trading-patterns, hyperliquid/overview |
| `hl.*trader`, `hl.*leaderboard`, `smart.*wallet.*hl`, `fresh.*whale` | hyperliquid/positions | core/security, hyperliquid/overview |
| `meteora.*pool`, `dlmm`, `damm`, `pool.*search`, `fee.*analysis` | meteora/pools | core/security, meteora/overview |
| `meteora.*lp`, `liquidity.*meteora`, `position.*meteora`, `lp.*profile` | meteora/liquidity | core/security, meteora/overview |
| `meteora.*stat`, `dca.*pressure`, `volume.*heatmap.*met`, `metengine.*share` | meteora/swaps | core/security, meteora/overview |
| `wallet.*setup`, `solana.*key`, `x402.*sign`, `payment.*setup` | wallet/signing-flows | core/security, wallet/overview |
| `mpp.*sign`, `mpp.*payment`, `tempo.*pay`, `machine.*payment` | wallet/mpp-signing-flows | core/security, wallet/overview |
| `phantom` | wallet/phantom/setup-and-signing | core/security, wallet/overview, wallet/signing-flows |
| `turnkey` | wallet/turnkey/setup-and-signing | core/security, wallet/overview, wallet/signing-flows |
| `privy` | wallet/privy/setup-and-signing | core/security, wallet/overview, wallet/signing-flows |
| `add.*connector`, `new.*platform`, `integrate.*platform` | core/connector-template | core/security |
| `compare.*platform`, `cross.*platform` | Load all three platform overviews | core/security |

## Rules

- Always load `core/security` first.
- Load at most 5 files per task (security + overview + up to 3 impl files).
- For cross-platform queries, load all three overviews but no impl files unless narrowed.
- `core/trading-patterns` is required only for intelligence, insider, and whale-trade flows.

## Base URL

```
https://agent.metengine.xyz
```

## Free Endpoints (no payment)

```
GET /health
GET /api/v1/pricing
GET /.well-known/mpp
```
