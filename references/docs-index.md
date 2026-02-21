# MetEngine Docs Index

Use this index to load only the minimum required context.

## Load Sequence

1. Read `references/docs-index.json` for machine-readable routing.
2. Read `references/core-runtime.md` for x402 payment, pricing, error handling, and scoring semantics.
3. Read one platform file:
- `references/polymarket-endpoints.md`
- `references/hyperliquid-endpoints.md`
- `references/meteora-endpoints.md`
4. Only if needed, read `references/core-extended.md` for advanced troubleshooting and legacy detail.

## Remote Fetch Endpoints

- https://raw.githubusercontent.com/MetEngine/skill/main/references/docs-index.json
- https://raw.githubusercontent.com/MetEngine/skill/main/references/core-runtime.md
- https://raw.githubusercontent.com/MetEngine/skill/main/references/core-extended.md
- https://raw.githubusercontent.com/MetEngine/skill/main/references/polymarket-endpoints.md
- https://raw.githubusercontent.com/MetEngine/skill/main/references/hyperliquid-endpoints.md
- https://raw.githubusercontent.com/MetEngine/skill/main/references/meteora-endpoints.md

## Fast Routing Heuristics

- If user asks about `condition_id`, `markets`, `outcomes`, `holders`: load Polymarket docs.
- If user asks about `perps`, `funding`, `OI`, `leverage`, `coins`: load Hyperliquid docs.
- If user asks about `pools`, `DLMM`, `DAMM`, `LP`, `fees`, `APR`: load Meteora docs.

## Context Budget Rule

Default context budget: `core + exactly one platform`.
Only load additional platform docs if the user explicitly requests cross-platform analysis.
