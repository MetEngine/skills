---
skill: wallet/signing-flows
requires: [core/security, wallet/overview]
triggers: ["x402.*sign", "payment.*sign", "sign.*transaction", "payment.*setup"]
level: implementation
---

# Provider-Agnostic Signing Flows

All MetEngine x402 payments follow the same flow regardless of wallet provider. The only difference is HOW the `signer` object is created. This file documents the shared flow; see provider-specific files for signer creation.

## Signing Flow

```
1. Create signer          --> Provider-specific (see SIGN_CALL below)
2. Initialize x402 client --> Shared (same for all providers)
3. Make paid request       --> Shared (same for all providers)
4. Process response        --> Shared (same for all providers)
```

## Shared Setup (All Providers)

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";
import type { PaymentRequired, SettleResponse } from "@x402/core/types";

// SIGN_CALL: Replace with provider-specific signer creation
// See: wallet/phantom, wallet/turnkey, or wallet/privy
const signer = SIGN_CALL();

// Shared x402 client initialization
const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(signer) });
const httpClient = new x402HTTPClient(client);
```

## Shared Paid Fetch Function

```typescript
const BASE_URL = "https://agent.metengine.xyz";

async function paidFetch(
  path: string,
  options?: { method?: string; body?: Record<string, unknown> },
): Promise<{ data: unknown; settlement: SettleResponse; price: number }> {
  const method = options?.method ?? "GET";
  const url = `${BASE_URL}${path}`;
  const fetchOpts: RequestInit = { method };
  if (options?.body) {
    fetchOpts.headers = { "Content-Type": "application/json" };
    fetchOpts.body = JSON.stringify(options.body);
  }

  // Step 1: Get 402 with price
  const initial = await fetch(url, fetchOpts);
  if (initial.status !== 402) throw new Error(`Expected 402, got ${initial.status}`);
  const body = await initial.json();

  // Step 2: Parse payment requirements
  const paymentRequired: PaymentRequired = httpClient.getPaymentRequiredResponse(
    (name) => initial.headers.get(name), body,
  );
  const price = Number(paymentRequired.accepts[0]!.amount);

  // Step 3: Sign payment
  const paymentPayload = await httpClient.createPaymentPayload(paymentRequired);
  const paymentHeaders = httpClient.encodePaymentSignatureHeader(paymentPayload);

  // Step 4: Re-send with payment
  const paid = await fetch(url, {
    ...fetchOpts,
    headers: { ...fetchOpts.headers as Record<string, string>, ...paymentHeaders },
  });
  if (paid.status !== 200) {
    const err = await paid.json();
    throw new Error(`Payment failed (${paid.status}): ${JSON.stringify(err)}`);
  }
  const paidBody = (await paid.json()) as { data: unknown };

  // Step 5: Extract settlement proof
  const settlement = httpClient.getPaymentSettleResponse(
    (name) => paid.headers.get(name),
  );

  return { data: paidBody.data, settlement, price };
}
```

## Usage Examples

```typescript
// GET endpoint
const { data, price } = await paidFetch("/api/v1/markets/trending?timeframe=24h&limit=5");
console.log(`Paid $${price} USDC. Got ${(data as any[]).length} markets.`);

// POST endpoint
const { data: intel } = await paidFetch("/api/v1/markets/intelligence", {
  method: "POST",
  body: { condition_id: "0xabc123...", top_n_wallets: 10 },
});
```

## NPM Dependencies

```bash
bun add @x402/core @x402/svm @solana/kit
```

## Provider-Specific Signer Creation

Replace `SIGN_CALL()` above with one of:

### Phantom / Local Keypair
```typescript
import { getBase58Encoder, createKeyPairSignerFromBytes } from "@solana/kit";
const bytes = getBase58Encoder().encode(process.env.SOLANA_PRIVATE_KEY!);
const signer = await createKeyPairSignerFromBytes(bytes);
```
See: `wallet/phantom/setup-and-signing`

### Turnkey
```typescript
import { TurnkeyClient } from "@turnkey/http";
import { createTurnkeySvmSigner } from "@turnkey/svm";
const turnkeyClient = new TurnkeyClient({ baseUrl: "https://api.turnkey.com" }, stamper);
const signer = await createTurnkeySvmSigner(turnkeyClient, organizationId, walletId);
```
See: `wallet/turnkey/setup-and-signing`

### Privy
```typescript
import { PrivyClient } from "@privy-io/server-auth";
const privy = new PrivyClient(appId, appSecret);
const wallet = await privy.walletApi.getWallet(walletId);
const signer = {
  address: wallet.address,
  signTransaction: async (tx: Uint8Array) => {
    const result = await privy.walletApi.solana.signTransaction({
      walletId: wallet.id,
      transaction: Buffer.from(tx).toString("base64"),
    });
    return Buffer.from(result.signedTransaction, "base64");
  },
};
```
See: `wallet/privy/setup-and-signing`

## Health Check (No Signing Required)

```bash
curl -sS https://agent.metengine.xyz/health
curl -sS https://agent.metengine.xyz/api/v1/pricing
```

These endpoints are free and require no wallet or payment.

## Onboarding Path

1. Verify service is live: `GET /health`
2. Make a paid request: `GET /api/v1/markets/trending?timeframe=24h&limit=5`
3. First call returns `402` with price. Sign payment. Re-send with `PAYMENT-SIGNATURE` header.
4. Receive `200` with data + settlement proof.
