---
skill: core/trading-patterns
requires: [core/security]
triggers: ["error.*handling", "fallback", "retry", "payment.*flow", "x402", "mpp"]
level: concept
---

# Trading Patterns and Error Handling

Shared patterns for intelligence, insider detection, and whale-trade flows across all platforms.

MetEngine supports three payment protocols. All use the same settle-after-execute guarantee: failed queries = no charge.

## MPP Payment Flow (Tempo EVM)

```
1. GET /api/v1/<endpoint>              --> 402 + WWW-Authenticate: Payment header
2. Sign TIP-20 USDC transfer (do NOT broadcast)
3. GET /api/v1/<endpoint> + Authorization: Payment <credential>
4. Server executes query -> if data returned -> broadcasts tx -> 200 + Payment-Receipt header
```

### MPP Setup

Tempo chain: ID 4217 (mainnet), RPC `https://rpc.presto.tempo.xyz`, native gas is USD, USDC.e contract `0x20C000000000000000000000b9537d11c60E8b50`. Bridge at `https://bridge.tempo.xyz`.

```bash
# 1. Create a wallet (one-time, stored in ~/.mppx/)
npx mppx account create

# 2. View address, then fund with USDC.e on Tempo chain via bridge
npx mppx account view

# 3. Make requests (automatic 402 handling)
npx mppx https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5

# Free endpoints (no wallet needed)
npx mppx https://agent.metengine.xyz/health
npx mppx https://agent.metengine.xyz/.well-known/mpp
```

### MPP Client Bootstrap (TypeScript/Bun)

```typescript
import { createClient } from "mppx/client";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

// Create viem wallet
const account = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
const wallet = createWalletClient({
  account,
  transport: http("https://rpc.presto.tempo.xyz"), // Tempo mainnet (chain ID 4217)
});

// Create MPP client with automatic 402 handling
const client = createClient({ wallet });
```

### MPP Reusable Paid Fetch

```typescript
const BASE_URL = "https://agent.metengine.xyz";

async function paidFetch(path: string, opts?: { method?: string; body?: Record<string, unknown> }) {
  const method = opts?.method ?? "GET";
  const url = `${BASE_URL}${path}`;
  const fetchOpts: RequestInit = { method };
  if (opts?.body) {
    fetchOpts.headers = { "Content-Type": "application/json" };
    fetchOpts.body = JSON.stringify(opts.body);
  }
  const response = await client.fetch(url, fetchOpts);
  if (!response.ok) throw new Error(`Request failed (${response.status}): ${await response.text()}`);
  return { data: (await response.json()).data };
}
```

### MPP NPM Dependencies

```bash
bun add mppx viem
```

## MPP Solana Payment Flow

```
1. GET /api/v1/<endpoint>                          --> 402 + WWW-Authenticate: Payment method="solana"
2. @solana/mpp client parses challenge, signs USDC SPL transfer
3. GET /api/v1/<endpoint> + Authorization: Payment  --> 200 + data + Payment-Receipt header
```

### MPP Solana Client Bootstrap (TypeScript/Bun)

```typescript
import { Mppx, solana } from "@solana/mpp/client";
import { createKeyPairSignerFromBytes, getBase58Encoder } from "@solana/kit";

const bytes = getBase58Encoder().encode(process.env.SOLANA_PRIVATE_KEY!);
const signer = await createKeyPairSignerFromBytes(bytes);

const mppx = Mppx.create({
  methods: [solana.charge({ signer })],
});

// Automatic 402 handling -- signs and retries with credential
const response = await mppx.fetch("https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h");
const { data } = await response.json();
```

### Dependencies

```bash
bun add @solana/mpp mppx @solana/kit
```

## x402 Payment Flow (Solana)

```
1. GET /api/v1/<endpoint>              --> 402 + PAYMENT-REQUIRED header
2. Sign payment locally with wallet    --> PAYMENT-SIGNATURE header
3. GET /api/v1/<endpoint> + signature  --> 200 + data + PAYMENT-RESPONSE header
```

### x402 Client Bootstrap (TypeScript/Bun)

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";
import { getBase58Encoder, createKeyPairSignerFromBytes } from "@solana/kit";

const bytes = getBase58Encoder().encode(process.env.SOLANA_PRIVATE_KEY!);
const signer = await createKeyPairSignerFromBytes(bytes);
const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(signer) });
const httpClient = new x402HTTPClient(client);
```

### x402 Reusable Paid Fetch

```typescript
const BASE_URL = "https://agent.metengine.xyz";

async function paidFetch(path: string, opts?: { method?: string; body?: Record<string, unknown> }) {
  const method = opts?.method ?? "GET";
  const url = `${BASE_URL}${path}`;
  const fetchOpts: RequestInit = { method };
  if (opts?.body) {
    fetchOpts.headers = { "Content-Type": "application/json" };
    fetchOpts.body = JSON.stringify(opts.body);
  }
  const initial = await fetch(url, fetchOpts);
  if (initial.status !== 402) throw new Error(`Expected 402, got ${initial.status}`);
  const body = await initial.json();
  const paymentRequired = httpClient.getPaymentRequiredResponse((n) => initial.headers.get(n), body);
  const paymentPayload = await httpClient.createPaymentPayload(paymentRequired);
  const paymentHeaders = httpClient.encodePaymentSignatureHeader(paymentPayload);
  const paid = await fetch(url, {
    ...fetchOpts,
    headers: { ...fetchOpts.headers as Record<string, string>, ...paymentHeaders },
  });
  if (paid.status !== 200) throw new Error(`Payment failed (${paid.status}): ${await paid.text()}`);
  const settlement = httpClient.getPaymentSettleResponse((n) => paid.headers.get(n));
  return { data: (await paid.json()).data, settlement };
}
```

### x402 NPM Dependencies

```bash
bun add @x402/core @x402/svm @solana/kit
```

## Error Handling

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process `{ "data": ... }`. Settlement proof in `PAYMENT-RESPONSE`. |
| 400 | Bad request | Fix params. Check required fields and value ranges. |
| 402 | Payment required | Sign and re-send. Or check payment validity if already signed. |
| 404 | Not found | Verify path against skill graph. |
| 429 | Rate limited | Backoff with jitter, retry. |
| 500 | Server error | Retry once, then try fallback endpoint. |
| 502 | Facilitator error | Retry payment verification. |
| 503 | Capacity/unavailable | Check `Retry-After` header, wait, retry. |
| 504 | Query timeout | No charge. Retry with narrower params or use fallback. |

## Fallback Strategies

| Failed Endpoint | Fallback |
|----------------|----------|
| `/markets/opportunities` (504) | `/markets/high-conviction` |
| `/wallets/top-performers` (503 on 7d) | Try `timeframe=24h` |
| `/markets/insiders` (timeout) | `/markets/trades` with market filter |
| `/hl/coins/trending` (empty on 24h) | Use `timeframe=7d` |
| `/hl/smart-wallets/signals` (empty on 24h) | Use `timeframe=7d` |
| `/hl/trades/long-short-ratio` (all zeros) | Reconstruct from `/hl/trades/whales` by side |
| `/hl/traders/profile` (500) | `/hl/traders/leaderboard` + `/hl/traders/pnl-by-coin` |
| `/meteora/lps/top?sort_by=fees` (500) | Use `sort_by=volume` |
| `/meteora/lps/profile` (500) | `/meteora/pools/fee-analysis` for claimer data |

## Retry Pattern

```
attempt 1: immediate
attempt 2: 1s + random(0-500ms)
attempt 3: 3s + random(0-1000ms)
give up: switch to fallback endpoint
```

Never retry more than 3 times. On persistent failure, switch to the fallback endpoint or inform the user.

## Price Modifiers Reference

**Timeframe multipliers**: 1h=0.5x, 4h=0.7x, 12h=0.9x, 24h=1.0x, 7d=2.0x, 30d=3.0x, 90d=4.0x, 365d=5.0x

**Filter discounts**: category=0.7x, condition_id/pool_address=0.5x, smart_money_only=0.7x, coin/coins=0.7x, pool_type(not "all")=0.7x

**Special rules**:
- `wallets/compare`: `price *= wallets.length / 2`
- `hl/traders/compare`: `price *= traders.length / 2`
- `meteora/lps/compare`: `price *= owners.length / 2`
- `wallets/profile`: both includes=1.0x, one=0.7x, neither=0.4x
- `wallets/top-performers` without category: 2.0x penalty

**Hard caps**: `/markets/opportunities` max $0.15, `/wallets/copy-traders` max $0.12
