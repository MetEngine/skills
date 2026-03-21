---
skill: wallet/phantom/setup-and-signing
requires: [core/security, wallet/overview, wallet/signing-flows]
triggers: ["phantom", "local.*keypair", "solana.*cli", "file.*wallet"]
level: implementation
---

# Phantom / Local Keypair Setup and Signing

Use this provider for CLI agents, local development, and single-user setups. The keypair is stored as a local file (Solana CLI format or base58-encoded private key).

## Prerequisites

- Solana CLI installed (`solana-keygen` for key generation)
- Or an exported Phantom wallet keypair
- Bun or Node.js runtime
- SOL + USDC in the wallet

## Setup

### Option A: Solana CLI Keypair (Recommended for CLI agents)

```bash
# Generate a new keypair (if needed)
solana-keygen new --outfile ~/.config/solana/id.json

# Check the public address
solana-keygen pubkey ~/.config/solana/id.json

# Fund with SOL (for tx fees)
solana airdrop 0.1  # devnet only; transfer SOL on mainnet

# Check balances
solana balance
spl-token balance EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v  # USDC mint
```

The keypair file is a JSON array of 64 bytes (32-byte secret key + 32-byte public key).

### Option B: Base58 Private Key (Environment Variable)

```bash
# Export from Phantom: Settings > Security > Export Private Key
# Store as environment variable (NEVER commit to source control)
export SOLANA_PRIVATE_KEY="your-base58-private-key"
```

## Signer Creation

### From Environment Variable (Base58)

```typescript
import { getBase58Encoder, createKeyPairSignerFromBytes } from "@solana/kit";

const bytes = getBase58Encoder().encode(process.env.SOLANA_PRIVATE_KEY!);
const signer = await createKeyPairSignerFromBytes(bytes);
```

### From Keypair File (Solana CLI format)

```typescript
import { createKeyPairSignerFromBytes } from "@solana/kit";
import { readFileSync } from "fs";

const keypairPath = process.env.SOLANA_KEYPAIR_PATH ?? "~/.config/solana/id.json";
const keypairBytes = new Uint8Array(JSON.parse(readFileSync(keypairPath, "utf-8")));
const signer = await createKeyPairSignerFromBytes(keypairBytes);
```

## Full Working Example

```typescript
import { x402Client, x402HTTPClient } from "@x402/core/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { toClientSvmSigner } from "@x402/svm";
import { getBase58Encoder, createKeyPairSignerFromBytes } from "@solana/kit";

// Create signer from base58 private key
const bytes = getBase58Encoder().encode(process.env.SOLANA_PRIVATE_KEY!);
const signer = await createKeyPairSignerFromBytes(bytes);

// Initialize x402 client
const client = new x402Client();
registerExactSvmScheme(client, { signer: toClientSvmSigner(signer) });
const httpClient = new x402HTTPClient(client);

// Make a paid request (see wallet/signing-flows for the full paidFetch function)
const response = await httpClient.fetch(
  "https://agent.metengine.xyz/api/v1/markets/trending?timeframe=24h&limit=5",
  { method: "GET" },
);
console.log(response.status, await response.json());
```

## Security Checklist

- [ ] Private key is in an environment variable or file, never hardcoded
- [ ] Keypair file has restricted permissions: `chmod 600 ~/.config/solana/id.json`
- [ ] `.env` file is in `.gitignore`
- [ ] No private key material appears in logs, memory files, or chat
- [ ] Public address stored in `~/.claude/agents/metengine-memory.md` (path + pubkey only)

## Session Memory Entry

After first successful setup, record in `~/.claude/agents/metengine-memory.md`:

```markdown
## Wallet
- provider: phantom/local
- keypair_path: ~/.config/solana/id.json
- public_address: <base58 pubkey>
- sol_balance_ok: true
- usdc_balance_ok: true
- last_balance_check: YYYY-MM-DD
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `Invalid keypair bytes` | Check format: base58 string for env var, JSON array for file |
| `Insufficient SOL` | Transfer SOL for tx fees (~0.001 per payment) |
| `Insufficient USDC` | Transfer USDC (SPL token, mint `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`) |
| `Permission denied` on keypair file | `chmod 600 <path>` |
