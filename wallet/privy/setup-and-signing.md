---
skill: wallet/privy/setup-and-signing
requires: [core/security, wallet/overview, wallet/signing-flows]
triggers: ["privy", "embedded.*wallet", "social.*login.*wallet", "server.*side.*sign"]
level: implementation
---

# Privy Setup and Signing

Use Privy for consumer-facing applications where users authenticate via social login (Google, email, etc.) and get embedded Solana wallets without managing keys directly. Supports server-side signing for automated agent flows.

## Prerequisites

- Privy app ID and app secret (from Privy dashboard)
- Privy SDK packages
- Bun or Node.js runtime
- SOL + USDC in the Privy-managed wallet

## Setup

### 1. Install Dependencies

```bash
bun add @privy-io/server-auth @privy-io/js-sdk-core @x402/core @x402/svm @solana/kit
```

### 2. Configure Credentials

```bash
# Store Privy credentials (NEVER commit to source control)
export PRIVY_APP_ID="your-app-id"
export PRIVY_APP_SECRET="your-app-secret"
export PRIVY_WALLET_ID="your-wallet-id"  # Created via Privy Wallet API
```

### 3. Create an Embedded Wallet

Privy wallets can be created programmatically via the Wallet API:

```typescript
import { PrivyClient } from "@privy-io/server-auth";

const privy = new PrivyClient(
  process.env.PRIVY_APP_ID!,
  process.env.PRIVY_APP_SECRET!,
);

// Create a new Solana wallet
const wallet = await privy.walletApi.create({
  chainType: "solana",
});

console.log("Wallet ID:", wallet.id);
console.log("Address:", wallet.address);
// Fund this address with SOL and USDC
```

## Signer Creation

```typescript
import { PrivyClient } from "@privy-io/server-auth";

const privy = new PrivyClient(
  process.env.PRIVY_APP_ID!,
  process.env.PRIVY_APP_SECRET!,
);

const walletId = process.env.PRIVY_WALLET_ID!;

// Get wallet address
const wallet = await privy.walletApi.getWallet(walletId);

// Create a Privy-backed signer
const signer = {
  address: wallet.address,
  signTransaction: async (tx: Uint8Array) => {
    const result = await privy.walletApi.solana.signTransaction({
      walletId,
      transaction: Buffer.from(tx).toString("base64"),
    });
    return Buffer.from(result.signedTransaction, "base64");
  },
};
```

## Full Working Example

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";
import { PrivyClient } from "@privy-io/server-auth";

// Initialize Privy
const privy = new PrivyClient(
  process.env.PRIVY_APP_ID!,
  process.env.PRIVY_APP_SECRET!,
);

// Create Privy signer (see "Signer Creation" section above for full implementation)
const walletId = process.env.PRIVY_WALLET_ID!;
const wallet = await privy.walletApi.getWallet(walletId);
const privySigner = {
  address: wallet.address,
  signTransaction: async (tx: Uint8Array) => {
    const result = await privy.walletApi.solana.signTransaction({
      walletId,
      transaction: Buffer.from(tx).toString("base64"),
    });
    return Buffer.from(result.signedTransaction, "base64");
  },
};

// Initialize x402 client
const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(privySigner) });
const httpClient = new x402HTTPClient(client);

// Make a paid request (see wallet/signing-flows for the full paidFetch function)
const response = await httpClient.fetch(
  "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5",
  { method: "GET" },
);
```

## MCP Tools Integration

Privy wallets integrate with Claude's MCP (Model Context Protocol) tools for browser-based agent workflows:

| MCP Tool | Usage |
|----------|-------|
| `privy_create_wallet` | Create a new Solana embedded wallet |
| `privy_get_wallet` | Get wallet address and balance |
| `privy_sign_transaction` | Sign a Solana transaction |
| `privy_send_transaction` | Sign and submit a transaction |

These tools allow Claude to manage Privy wallets directly in a browser automation session without exposing private keys.

## User Authentication Flow (Consumer App)

For consumer-facing apps where end users authenticate:

```typescript
// Client-side: User logs in via Privy
import { usePrivy } from "@privy-io/react-auth";

const { login, authenticated, user } = usePrivy();

// After login, user has an embedded Solana wallet
const solanaWallet = user?.linkedAccounts.find(
  (a) => a.type === "wallet" && a.chainType === "solana",
);
```

## Security Checklist

- [ ] App secret stored in environment variable, never client-side
- [ ] Server-side signing only (app secret never exposed to browser)
- [ ] Wallet API access restricted to authorized server processes
- [ ] `.env` file in `.gitignore`
- [ ] Rate limiting configured on Privy dashboard
- [ ] Webhook configured for transaction notifications (optional)

## Session Memory Entry

```markdown
## Wallet
- provider: privy
- app_id: <privy-app-id>
- wallet_id: <privy-wallet-id>
- public_address: <solana address>
- sol_balance_ok: true
- usdc_balance_ok: true
- last_balance_check: YYYY-MM-DD
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `401 Unauthorized` | Check app ID and app secret match |
| `Wallet not found` | Verify wallet ID belongs to this Privy app |
| `Insufficient funds` | Fund the wallet address shown in Privy dashboard |
| Signing latency >300ms | Normal for server-side signing; subsequent calls faster |
| `Rate limit exceeded` | Check Privy dashboard rate limits; upgrade plan if needed |
