---
name: metengine-data-agent
description: Real-time smart money analytics for Polymarket, Hyperliquid, and Meteora via MetEngine pay-per-request API. Supports MPP (Tempo EVM) and x402 (Solana). Use when tasks require wallet scoring, insider detection, position analysis, market flow, or LP/AMM analytics across these platforms. Payment IS authentication -- no API keys, no accounts.
---

# MetEngine Data Agent

Keep this file small. Load detailed docs only when needed.

## Non-Negotiable Rule

- Never truncate addresses or IDs. Always return full values (EVM hex, Solana base58, condition IDs, token IDs, pool addresses, tx hashes).

## Runtime Workflow

1. Load `index.md` and match user intent to a skill file via the routing table.
2. Load the matched skill's `requires` chain (always starts with `core/security`).
3. Load the matched skill file itself.
4. If payment flow details are needed, load `core/trading-patterns`.
5. Execute payment handshake (either protocol):
   - **MPP:** EVM wallet on Tempo chain. `npx mppx <url>` or `mppx/client` SDK.
   - **x402:** Solana wallet with USDC + SOL. `@x402/core` + `@x402/svm`.
6. Return results with full IDs and endpoint provenance.

## Progressive Loading Map

Load order (stop as soon as enough context exists):

1. `index.md` (routing table -- match triggers to skill files)
2. `core/security.md` (always loaded -- display rules, pricing, limits)
3. One platform overview:
   - `polymarket/overview.md`
   - `hyperliquid/overview.md`
   - `meteora/overview.md`
4. One or more implementation files as needed:
   - `polymarket/markets.md`, `polymarket/orders.md`, `polymarket/positions.md`
   - `hyperliquid/market-data.md`, `hyperliquid/trading.md`, `hyperliquid/positions.md`
   - `meteora/pools.md`, `meteora/liquidity.md`, `meteora/swaps.md`
5. Wallet setup (if needed):
   - **MPP:** `wallet/overview.md` -> `wallet/mpp-signing-flows.md`
   - **x402:** `wallet/overview.md` -> `wallet/signing-flows.md` -> provider file

Do not load all platform files unless user explicitly asks for cross-platform aggregation.
Max 5 files per task.

## Remote LLM-Friendly Docs (for hosted usage)

Use raw GitHub URLs when the skill is installed remotely:

- `https://raw.githubusercontent.com/MetEngine/skill/main/index.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/core/security.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/core/trading-patterns.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/polymarket/overview.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/hyperliquid/overview.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/meteora/overview.md`

## Token Discipline

- Load `index.md` first, match triggers, then load only the required chain.
- Keep working context to: `core/security` + one platform overview + one implementation file.
- Only pull multi-platform docs if user asks for comparisons, arb, or blended flows.
- Prefer `rg`/targeted section reads over loading full files.
