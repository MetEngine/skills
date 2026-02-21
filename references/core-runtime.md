# Core Runtime (Minimal)

Use this file for normal paid requests. Load `core-extended.md` only for troubleshooting or deep policy details.

## Preconditions

- Wallet available in local Solana CLI keypair (default: `~/.config/solana/id.json`).
- Wallet funded with:
  - SOL for transaction fees.
  - USDC for API payments.
- Dependencies: `@x402/core`, `@x402/svm`, `@solana/kit`.

## Free Checks

```bash
curl -sS https://agent.metengine.xyz/health
curl -sS https://agent.metengine.xyz/api/v1/pricing
```

## x402 Payment Flow

1. Call a paid endpoint without payment header.
2. Receive `402` with payment requirements (`PAYMENT-REQUIRED` / `X-PAYMENT-REQUIRED`).
3. Sign payment locally with wallet.
4. Retry same request with `PAYMENT-SIGNATURE` (or `X-PAYMENT`).
5. Receive `200` and settlement proof (`PAYMENT-RESPONSE` / `X-PAYMENT-RESPONSE`).

Settlement occurs only on successful requests.

## Minimal TypeScript Pattern

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";

const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(signer) });
const httpClient = new x402HTTPClient(client);

const response = await httpClient.fetch("https://agent.metengine.xyz/api/v1/...", {
  method: "GET"
});
```

## Error Handling

- `402`: sign payment and retry.
- `400`: invalid query params; fix input.
- `404`: endpoint/resource missing.
- `429`: backoff and retry.
- `5xx`: retry with jitter; if persistent, switch to fallback endpoint.

## Output Rules

- Never truncate addresses or IDs.
- Include endpoint path and key parameters used in the response narrative.
