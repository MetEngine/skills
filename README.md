# MetEngine Data Agent Skill

LLM skill for MetEngine's x402 pay-per-request analytics API across:
- Polymarket
- Hyperliquid
- Meteora

The repo is structured for progressive disclosure so agents load minimal context first, then fetch only the docs needed for the active platform.

## Repo Layout

- `SKILL.md`: compact orchestrator instructions and loading rules
- `agents/openai.yaml`: listing metadata for skill directories/UIs
- `references/docs-index.json`: machine-readable routing map
- `references/docs-index.md`: human-readable routing guide
- `references/core-runtime.md`: minimal runtime payment flow
- `references/core-extended.md`: advanced/troubleshooting details
- `references/polymarket-endpoints.md`: Polymarket endpoint reference
- `references/hyperliquid-endpoints.md`: Hyperliquid endpoint reference
- `references/meteora-endpoints.md`: Meteora endpoint reference

## Design Goals

- Keep always-loaded skill context small
- Route queries to one platform doc by default
- Support remote raw-doc loading from GitHub when skill is installed elsewhere
- Preserve strict output rule: never truncate addresses/IDs

## Quick Local Check

1. Open `SKILL.md`.
2. Load `references/docs-index.json`.
3. Load `references/core-runtime.md`.
4. Load exactly one platform file based on user intent.

## Remote Doc Endpoints

- `https://raw.githubusercontent.com/MetEngine/skill/main/references/docs-index.json`
- `https://raw.githubusercontent.com/MetEngine/skill/main/references/core-runtime.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/references/core-extended.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/references/polymarket-endpoints.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/references/hyperliquid-endpoints.md`
- `https://raw.githubusercontent.com/MetEngine/skill/main/references/meteora-endpoints.md`
