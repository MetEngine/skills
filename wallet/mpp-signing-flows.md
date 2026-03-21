---
skill: wallet/mpp-signing-flows
requires: [core/security, wallet/overview]
triggers: ["mpp.*sign", "mpp.*payment", "tempo.*pay", "machine.*payment"]
level: implementation
---

# MPP (Machine Payments Protocol) Signing Flows

MPP uses standard HTTP authentication headers for pay-per-request API access using EVM stablecoins (USDC.e on Tempo chain). The flow is: challenge -> credential -> receipt, using `WWW-Authenticate: Payment`, `Authorization: Payment`, and `Payment-Receipt` headers.

**Before any `mppx` CLI or SDK usage, set the mainnet RPC.** The `mppx` CLI defaults to Tempo testnet (chain 42431), which will fail with `TIP20 token error: Uninitialized`. MetEngine runs on mainnet only.

```bash
export MPPX_RPC_URL=https://rpc.presto.tempo.xyz
```

## Tempo Chain Details

| Detail | Value |
|--------|-------|
| Chain ID | 4217 (mainnet) |
| RPC | `https://rpc.presto.tempo.xyz` |
| Native gas | USD (not ETH) |
| USDC.e contract | `0x20C000000000000000000000b9537d11c60E8b50` |
| Explorer | `https://explore.tempo.xyz` |
| Bridge | `https://bridge.tempo.xyz` (from Ethereum/Arbitrum/Base) |

## Protocol Flow

```
1. Client: GET /api/v1/markets/trending
2. Server: 402 + WWW-Authenticate: Payment (challenge with amount, currency, recipient)
3. Client: Sign a TIP-20 USDC transfer transaction (do NOT broadcast)
4. Client: Re-send with Authorization: Payment <credential>
5. Server: Execute query -> if success with data -> broadcast tx -> 200 + Payment-Receipt header
```

Key difference from x402: Server only broadcasts the transaction AFTER confirming the query returned data. If the query fails, times out, or returns empty, no payment is settled.

## Automatic 402 Handling (Recommended)

```typescript
import { createClient } from "mppx/client";
import { createWalletClient, http } from "viem";
import { tempo } from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";

// Create viem wallet -- MUST use Tempo mainnet RPC and chain
const account = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
const wallet = createWalletClient({
  account,
  chain: tempo, // Tempo mainnet (chain ID 4217)
  transport: http("https://rpc.presto.tempo.xyz"),
});

// Create an MPP client with automatic 402 retry
const client = createClient({ wallet });

// Automatic: fetches, gets 402, signs, re-sends
const response = await client.fetch(
  "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5",
);
const { data } = await response.json();
```

## Manual Flow (Provider-Agnostic)

```typescript
import { Challenge, Credential, Receipt } from "mppx";
import { createWalletClient, http, encodeFunctionData } from "viem";

const BASE_URL = "https://agent.metengine.xyz";

async function paidFetch(
  path: string,
  wallet: any, // viem WalletClient with account
  options?: { method?: string; body?: Record<string, unknown> },
): Promise<{ data: unknown; receipt: Receipt.Receipt; price: number }> {
  const method = options?.method ?? "GET";
  const url = `${BASE_URL}${path}`;
  const fetchOpts: RequestInit = { method };
  if (options?.body) {
    fetchOpts.headers = { "Content-Type": "application/json" };
    fetchOpts.body = JSON.stringify(options.body);
  }

  // Step 1: Get 402 with challenge
  const initial = await fetch(url, fetchOpts);
  if (initial.status !== 402) throw new Error(`Expected 402, got ${initial.status}`);

  // Step 2: Parse challenge from WWW-Authenticate header
  const challenge = Challenge.fromResponse(initial);
  const price = Number(challenge.request.amount) / 1_000_000; // Convert from smallest unit

  // Step 3: Sign a TIP-20 transfer transaction (DO NOT broadcast)
  // The transaction transfers `amount` of `currency` token to `recipient`
  const signedTx = await wallet.signTransaction({
    to: challenge.request.currency,        // TIP-20 token contract
    data: encodeFunctionData({             // transfer(recipient, amount)
      abi: [{
        name: "transfer",
        type: "function",
        inputs: [{ name: "to", type: "address" }, { name: "amount", type: "uint256" }],
        outputs: [{ type: "bool" }],
      }],
      functionName: "transfer",
      args: [challenge.request.recipient, BigInt(challenge.request.amount)],
    }),
  });

  // Step 4: Build credential and re-send
  const credential = Credential.from({
    challenge,
    payload: { type: "transaction", signature: signedTx },
    source: `did:pkh:eip155:${challenge.request.chainId}:${wallet.account.address}`,
  });

  const paid = await fetch(url, {
    ...fetchOpts,
    headers: {
      ...(fetchOpts.headers as Record<string, string>),
      Authorization: Credential.serialize(credential),
    },
  });

  if (!paid.ok) {
    const err = await paid.json();
    throw new Error(`Payment failed (${paid.status}): ${JSON.stringify(err)}`);
  }

  // Step 5: Extract receipt
  const receipt = Receipt.fromResponse(paid);
  const { data } = (await paid.json()) as { data: unknown };

  return { data, receipt, price };
}
```

## NPM Dependencies

```bash
bun add mppx viem
```

## Wallet Setup (viem)

```typescript
import { createWalletClient, http } from "viem";
import { tempo } from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";

// From private key (development/CLI)
const account = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
const wallet = createWalletClient({
  account,
  chain: tempo, // Tempo mainnet (chain ID 4217) -- do NOT omit or use tempoTestnet
  transport: http("https://rpc.presto.tempo.xyz"),
});

// Or use browser wallet (MetaMask, etc.)
// const [account] = await window.ethereum.request({ method: "eth_requestAccounts" });
```

## Wallet Setup (mppx CLI)

```bash
# 1. Set mainnet RPC FIRST (mppx defaults to testnet without this!)
export MPPX_RPC_URL=https://rpc.presto.tempo.xyz

# 2. Create a new wallet (stored locally in ~/.mppx/)
npx mppx account create

# 3. View wallet address
npx mppx account view

# 4. Fund with USDC.e on Tempo mainnet:
#    - Bridge from Ethereum/Arbitrum/Base via https://bridge.tempo.xyz
#    - Send USDC.e to the address shown by `account view`
#    - USDC.e contract: 0x20C000000000000000000000b9537d11c60E8b50

# 5. Make a request (automatic 402 handling)
npx mppx https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5

# Alternative: pass -r per request instead of env var
npx mppx -r https://rpc.presto.tempo.xyz https://agent.metengine.xyz/api/v1/markets/trending
```

## Health Check (No Signing Required)

```bash
curl -sS https://agent.metengine.xyz/health
curl -sS https://agent.metengine.xyz/api/v1/pricing
curl -sS https://agent.metengine.xyz/.well-known/mpp
```

These endpoints are free and require no wallet or payment.

## Onboarding Path

1. Set mainnet RPC: `export MPPX_RPC_URL=https://rpc.presto.tempo.xyz`
2. Verify service is live: `GET /health`
3. Check discovery: `GET /.well-known/mpp` (endpoint catalog, pricing)
4. Make a paid request: `GET /api/v1/markets/trending?timeframe=24h&limit=5`
5. First call returns `402` with `WWW-Authenticate: Payment` header.
6. Sign a TIP-20 USDC transfer (pull mode -- do NOT broadcast yourself).
7. Re-send with `Authorization: Payment <credential>`.
8. Receive `200` with data + `Payment-Receipt` header containing tx hash.

## Important: Pull Mode Only

The server only accepts **pull mode** credentials (`type: "transaction"` with a signed but unbroadcast transaction). Push mode (`type: "hash"` where the client already broadcast) is rejected because the server must control settlement timing to implement the "no charge on failure" guarantee.
