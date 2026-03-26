# MetEngine Data Agent Skill

LLM skill for MetEngine's pay-per-request analytics API across:
- Polymarket (38 endpoints)
- Hyperliquid (18 endpoints)
- Meteora (18 endpoints)

Supports three payment protocols: **MPP on Tempo** (EVM), **MPP on Solana** (`@solana/mpp`), and **x402** (Solana). Payment IS authentication -- no API keys, no accounts.

The repo is structured as a skill graph with YAML frontmatter routing. Agents load `index.md`, match triggers to a skill file, then load only the `requires` chain needed for the task. Max 5 files per task.

## Repo Layout

```
SKILL.md                          -- Compact orchestrator instructions
index.md                          -- Routing table (triggers -> skill files)
core/
  security.md                     -- Display rules, pricing, wallet security, limits
  trading-patterns.md             -- MPP + x402 payment flows, error handling, fallbacks
  connector-template.md           -- Guide for adding new platform connectors
polymarket/
  overview.md                     -- 38-endpoint summary, scoring, quirks
  markets.md                      -- Market discovery endpoints
  orders.md                       -- Intelligence and trade endpoints
  positions.md                    -- Wallet analysis endpoints
hyperliquid/
  overview.md                     -- 18-endpoint summary, smart score, quirks
  market-data.md                  -- Coin/market data endpoints
  trading.md                      -- Trade and pressure endpoints
  positions.md                    -- Trader profiling endpoints
meteora/
  overview.md                     -- 18-endpoint summary, pool types, quirks
  pools.md                        -- Pool discovery endpoints
  liquidity.md                    -- LP and position endpoints
  swaps.md                        -- Platform stats and DCA endpoints
wallet/
  overview.md                     -- Provider comparison (MPP vs x402 providers)
  mpp-signing-flows.md            -- MPP payment flow (Tempo EVM)
  signing-flows.md                -- Provider-agnostic x402 signing flow (Solana)
  phantom/setup-and-signing.md    -- Local keypair setup (x402)
  turnkey/setup-and-signing.md    -- HSM-backed key management (x402)
  privy/setup-and-signing.md      -- Embedded wallet / social login (x402)
public/metengine-mpp/SKILL.md    -- Public installable MPP skill for Claude Code
agents/openai.yaml                -- Listing metadata for skill directories
references/                       -- Retired (redirects to new structure)
```

## Design Goals

- Route queries to minimal context via trigger patterns
- Max 5 files loaded per task
- Every file has YAML frontmatter: `skill`, `requires`, `triggers`, `level`
- `core/security` required by all skills
- Max dependency depth: 4
- No file exceeds 300 lines

## Quick Start

1. Open `SKILL.md`
2. Load `index.md` -- match user intent to a skill via trigger patterns
3. Load `core/security.md` (always)
4. Load the matched skill's `requires` chain + the skill itself
