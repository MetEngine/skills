---
skill: core/connector-template
requires: [core/security]
triggers: ["add.*connector", "new.*platform", "integrate.*platform", "extend.*api"]
level: concept
---

# Adding a New Platform Connector

Step-by-step guide for extending the MetEngine skill graph with a new platform.

## Prerequisites

- Understanding of the target platform's on-chain data model
- Access to the platform's RPC or API for data indexing
- Familiarity with the MetEngine skill graph structure (see `skills/index.md`)

## Architecture Overview

Each platform connector consists of:

1. **Overview file** (`<platform>/overview.md`) -- concept-level summary, endpoint table, scoring, quirks
2. **Implementation files** (`<platform>/<domain>.md`) -- detailed endpoint contracts grouped by domain
3. **Routing entries** in `skills/index.md` -- trigger patterns pointing to the new files
4. **Pricing configuration** -- tier assignments for each endpoint

## Step 1: Define the Endpoint Manifest

List all endpoints the platform will expose. For each endpoint, determine:

| Field | Description |
|-------|-------------|
| Number | Sequential ID continuing from last endpoint (currently 63) |
| Method | GET or POST |
| Path | `/api/v1/<platform>/<resource>` |
| Tier | light, medium, heavy, or whale |
| Description | One-line summary |

### Tier Assignment Guide

| Tier | When to Use |
|------|-------------|
| light ($0.01) | Direct lookups, search, feeds. Single-table scan, <100ms query. |
| medium ($0.02) | Aggregated data, trending, time series. Multi-join or materialized view. |
| heavy ($0.05) | Computed intelligence, scoring, leaderboards. Complex aggregation or ML scoring. |
| whale ($0.08) | Multi-entity comparisons, cross-entity scans. N-way joins or full-table scans. |

## Step 2: Create the Overview File

Create `skills/<platform>/overview.md`:

```markdown
---
skill: <platform>/overview
requires: [core/security]
triggers: ["<platform>", "<platform-specific-keywords>"]
level: concept
---

# <Platform> Overview

<One paragraph describing the platform and what MetEngine indexes from it.>

## Endpoint Summary (N endpoints)

| # | Method | Path | Tier | Description |
|---|--------|------|------|-------------|
| ... | ... | ... | ... | ... |

## Scoring System

<How wallets/entities are scored on this platform. Include formula if available.>

| Tier | Score | Meaning |
|------|-------|---------|
| ... | ... | ... |

## Address Format

| Field | Format |
|-------|--------|
| Address type | <hex/base58/etc> |
| Case sensitivity | <sensitive/insensitive/must be lowercase> |

## Known Quirks

- <Document platform-specific gotchas, data anomalies, timeout-prone endpoints>
```

## Step 3: Create Implementation Files

Group endpoints by domain (typically 2-4 files). Each file follows this template:

```markdown
---
skill: <platform>/<domain>
requires: [core/security, <platform>/overview]
triggers: ["<domain-specific-patterns>"]
level: implementation
---

# <Platform> <Domain> Endpoints

#### N. METHOD /api/v1/<platform>/<resource>

<One-line description.>

**Parameters (<query string|request body>):**
| Param | Type | Default | Values | Required |
|-------|------|---------|--------|----------|
| ... | ... | ... | ... | ... |

**Response schema:**
\```json
{
  "data": { ... }
}
\```

<Optional: Known issues, pricing notes, or fallback info.>
```

### Domain Grouping Guidelines

| Domain | Contains |
|--------|----------|
| market-data | Discovery, search, trending, stats, time series |
| trading | Trade feeds, whale trades, pressure, signals |
| positions | Entity profiles, leaderboards, comparisons, PnL |
| pools | Pool discovery, detail, fees (DeFi-specific) |
| liquidity | LP profiles, position tracking (DeFi-specific) |

## Step 4: Add Routing Entries

Add rows to the routing table in `skills/index.md`:

```markdown
| `<platform>.*<keyword>` | <platform>/<domain> | core/security, <platform>/overview |
```

Choose trigger patterns that are:
- Specific enough to avoid false matches with existing platforms
- Broad enough to catch natural language variations
- Regex-compatible (used for pattern matching)

## Step 5: Configure Pricing

Each endpoint needs a tier assignment. Add to the pricing configuration:

- Base cost from tier
- Applicable timeframe multipliers
- Filter discounts (if the endpoint accepts narrowing filters)
- Compare-style scaling (if the endpoint accepts arrays of entities)
- Hard caps (for expensive endpoints, typically max $0.15-$0.20)

## Step 6: Document Known Quirks

For each endpoint that has non-obvious behavior:
- Timeout-prone endpoints: document fallback strategy
- Empty-result patterns: document alternative timeframes
- Data anomalies: document filtering recommendations
- Add these to the overview file AND to `core/trading-patterns.md` fallback table

## Step 7: Validation Checklist

Before merging:

- [ ] Every endpoint has a unique sequential number
- [ ] All paths follow `/api/v1/<platform>/<resource>` convention
- [ ] All parameter tables include Type, Default, Values, Required columns
- [ ] All response schemas show the `{ "data": ... }` wrapper
- [ ] Overview file has: endpoint summary, scoring system, address format, known quirks
- [ ] Frontmatter on every file has: skill, requires, triggers, level
- [ ] `core/security` is in the `requires` chain of every implementation file
- [ ] Routing entries in `index.md` cover all endpoint domains
- [ ] No file exceeds 300 lines
- [ ] Total dependency depth <= 4 (router -> overview -> impl -> provider)

## File Count Guidelines

| Platform Complexity | Endpoint Count | Suggested Files |
|-------------------|----------------|-----------------|
| Simple | 5-10 | overview + 1 impl |
| Medium | 10-20 | overview + 2-3 impl |
| Complex | 20-30 | overview + 3-4 impl |

## Example: Adding a Drift Protocol Connector

```
skills/
  drift/
    overview.md          -- 15 endpoints, perp scoring, Solana base58 addresses
    market-data.md       -- markets/trending, markets/search, stats
    trading.md           -- trades/feed, trades/whales, funding-rates
    positions.md         -- traders/profile, traders/leaderboard, traders/compare
```

Routing entries:
```
| `drift.*market`, `drift.*trending` | drift/market-data | core/security, drift/overview |
| `drift.*trade`, `drift.*whale` | drift/trading | core/security, core/trading-patterns, drift/overview |
| `drift.*trader`, `drift.*leaderboard` | drift/positions | core/security, drift/overview |
```
