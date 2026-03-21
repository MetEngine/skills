---
skill: wallet/turnkey/setup-and-signing
requires: [core/security, wallet/overview, wallet/signing-flows]
triggers: ["turnkey", "hsm.*wallet", "policy.*engine", "enterprise.*wallet"]
level: implementation
---

# Turnkey Setup and Signing

Use Turnkey for production deployments requiring HSM-backed key management, policy-controlled signing, and enterprise-grade security. Keys never leave Turnkey's secure enclave.

## Prerequisites

- Turnkey organization and API credentials
- Turnkey SDK packages
- Bun or Node.js runtime
- SOL + USDC in the Turnkey-managed wallet

## Setup

### 1. Install Dependencies

```bash
bun add @turnkey/http @turnkey/api-key-stamper @turnkey/svm @x402/core @x402/svm @solana/kit
```

### 2. Configure API Credentials

```bash
# Store Turnkey API credentials (NEVER commit to source control)
export TURNKEY_API_PUBLIC_KEY="your-api-public-key"
export TURNKEY_API_PRIVATE_KEY="your-api-private-key"
export TURNKEY_ORGANIZATION_ID="your-org-id"
export TURNKEY_WALLET_ID="your-wallet-id"  # Solana wallet created in Turnkey dashboard
```

### 3. Create Solana Wallet in Turnkey

If you haven't created a Solana wallet yet:

1. Log into Turnkey dashboard
2. Create a new wallet with Solana curve (Ed25519)
3. Note the wallet ID and Solana address
4. Fund the address with SOL and USDC on Mainnet

## Signer Creation

```typescript
import { TurnkeyClient } from "@turnkey/http";
import { ApiKeyStamper } from "@turnkey/api-key-stamper";

// Initialize Turnkey client
const stamper = new ApiKeyStamper({
  apiPublicKey: process.env.TURNKEY_API_PUBLIC_KEY!,
  apiPrivateKey: process.env.TURNKEY_API_PRIVATE_KEY!,
});

const turnkeyClient = new TurnkeyClient(
  { baseUrl: "https://api.turnkey.com" },
  stamper,
);

// Get the Solana address for the wallet
const walletAccounts = await turnkeyClient.getWalletAccounts({
  organizationId: process.env.TURNKEY_ORGANIZATION_ID!,
  walletId: process.env.TURNKEY_WALLET_ID!,
});

const solanaAccount = walletAccounts.accounts.find(
  (a) => a.curve === "CURVE_ED25519",
);

// Create a Turnkey-backed signer
// The signer signs via Turnkey API -- private key never leaves the HSM
const signer = {
  address: solanaAccount!.address,
  signTransaction: async (tx: Uint8Array) => {
    const result = await turnkeyClient.signRawPayload({
      organizationId: process.env.TURNKEY_ORGANIZATION_ID!,
      signWith: solanaAccount!.address,
      payload: Buffer.from(tx).toString("hex"),
      encoding: "PAYLOAD_ENCODING_HEXADECIMAL",
      hashFunction: "HASH_FUNCTION_NOT_APPLICABLE",
    });
    return Buffer.from(result.activity.result.signRawPayloadResult!.signature, "hex");
  },
};
```

## Full Working Example

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";
import { TurnkeyClient } from "@turnkey/http";
import { ApiKeyStamper } from "@turnkey/api-key-stamper";

// Initialize Turnkey client (see signer creation above)
const stamper = new ApiKeyStamper({
  apiPublicKey: process.env.TURNKEY_API_PUBLIC_KEY!,
  apiPrivateKey: process.env.TURNKEY_API_PRIVATE_KEY!,
});
const turnkeyClient = new TurnkeyClient({ baseUrl: "https://api.turnkey.com" }, stamper);

// Create Turnkey signer (see "Signer Creation" section above for full implementation)
const walletAccounts = await turnkeyClient.getWalletAccounts({
  organizationId: process.env.TURNKEY_ORGANIZATION_ID!,
  walletId: process.env.TURNKEY_WALLET_ID!,
});
const solanaAccount = walletAccounts.accounts.find((a) => a.curve === "CURVE_ED25519");
const turnkeySigner = {
  address: solanaAccount!.address,
  signTransaction: async (tx: Uint8Array) => {
    const result = await turnkeyClient.signRawPayload({
      organizationId: process.env.TURNKEY_ORGANIZATION_ID!,
      signWith: solanaAccount!.address,
      payload: Buffer.from(tx).toString("hex"),
      encoding: "PAYLOAD_ENCODING_HEXADECIMAL",
      hashFunction: "HASH_FUNCTION_NOT_APPLICABLE",
    });
    return Buffer.from(result.activity.result.signRawPayloadResult!.signature, "hex");
  },
};

// Initialize x402 client
const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(turnkeySigner) });
const httpClient = new x402HTTPClient(client);

// Make a paid request (see wallet/signing-flows for the full paidFetch function)
const response = await httpClient.fetch(
  "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5",
  { method: "GET" },
);
```

## Policy Engine

Turnkey supports policy rules to control signing:

| Policy | Description |
|--------|-------------|
| Spending limit | Max USDC per transaction or per time period |
| Allowlist | Only sign for specific destination addresses |
| Rate limit | Max transactions per minute/hour |
| Approval required | Multi-sig or approval workflow for large amounts |

Configure policies in the Turnkey dashboard or via API:

```typescript
await turnkeyClient.createPolicy({
  organizationId: process.env.TURNKEY_ORGANIZATION_ID!,
  policyName: "metengine-spending-limit",
  effect: "EFFECT_ALLOW",
  consensus: "approvers.any()",
  condition: "solana.amount <= 20000",  // 0.20 USDC in minor units
});
```

## Security Checklist

- [ ] API credentials stored in environment variables, never hardcoded
- [ ] Turnkey organization has appropriate access controls
- [ ] Spending policies configured to limit per-request and daily spend
- [ ] Wallet address allowlisted for MetEngine facilitator only
- [ ] Audit logs enabled in Turnkey dashboard
- [ ] `.env` file in `.gitignore`

## Session Memory Entry

```markdown
## Wallet
- provider: turnkey
- organization_id: <org-id>
- wallet_id: <wallet-id>
- public_address: <solana address>
- sol_balance_ok: true
- usdc_balance_ok: true
- policy: spending-limit-0.20-per-tx
- last_balance_check: YYYY-MM-DD
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `401 Unauthorized` | Check API key pair matches organization |
| `Policy denied` | Review Turnkey policies; increase spending limit if needed |
| Signing latency >500ms | Normal for first call (cold start); subsequent calls 100-300ms |
| `Wallet not found` | Verify wallet ID and organization ID match |
