---
skill: polymarket/proxy-wallet
requires: [core/security, polymarket/overview]
triggers: ["proxy.*wallet", "polymarket.*trade", "clob.*order", "magic.*link"]
level: implementation
---

# Polymarket Proxy Wallet

Polymarket uses a CREATE2-derived proxy wallet (Gnosis Safe) for all trading. The proxy is deterministically derived from the signer's EOA address.

## CREATE2 Derivation

| Constant | Value |
|----------|-------|
| Proxy Factory | `0xaB45c5A4B0c941a2F231C04C3f49182e1A254052` |
| Init Code Hash | `0xd21df8dc65880a8606f09fe0ce3df9b8869287ab0b058be05aa9e8af6330a00b` |

Formula:
```
salt = keccak256(abi.encodePacked(eoa_address))    // 20 bytes, left-padded to 32
proxy = address(keccak256(0xff ++ factory ++ salt ++ init_code_hash)[12:])
```

Python derivation:
```python
from web3 import Web3

PROXY_FACTORY = "0xaB45c5A4B0c941a2F231C04C3f49182e1A254052"
INIT_CODE_HASH = bytes.fromhex(
    "d21df8dc65880a8606f09fe0ce3df9b8869287ab0b058be05aa9e8af6330a00b"
)

salt = Web3.keccak(bytes.fromhex(eoa_address[2:].lower().zfill(64)))
proxy = Web3.to_checksum_address(
    Web3.keccak(
        b"\xff"
        + bytes.fromhex(PROXY_FACTORY[2:])
        + salt
        + INIT_CODE_HASH
    )[12:]
)
```

## Contract Addresses (Polygon)

| Contract | Address |
|----------|---------|
| Proxy Factory | `0xaB45c5A4B0c941a2F231C04C3f49182e1A254052` |
| Relay Hub | `0xD216153c06E857cD7f72665E0aF1d7D82172F494` |
| USDC.e | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` |
| CTF (Conditional Tokens) | `0x4d97dcd97ec945f40cf65f87097ace5ea0476045` |
| CTF Exchange | `0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E` |
| Neg Risk CTF Exchange | `0xC5d563A36AE78145C45a50134d48A1215220f80a` |
| Neg Risk Adapter | `0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296` |

## CLOB Credential Flow

1. Initialize `ClobClient` with signer key, `chain_id=137`, `signature_type=2` (POLY_GNOSIS_SAFE), and `funder=proxy_address`
2. Call `create_or_derive_api_creds()` -- signs a CLOB nonce with the signer key to derive deterministic API key + secret
3. Set creds on client: `clob_client.set_api_creds(creds)`

Signature type `2` tells the CLOB the signer operates through a proxy. The CLOB verifies the signer owns the proxy via the CREATE2 derivation.

## Token Approval Flow

Four spender contracts need ERC-20 approval from the proxy wallet:

| Token | Spender | Purpose |
|-------|---------|---------|
| USDC.e | CTF Exchange | Standard market orders |
| USDC.e | Neg Risk CTF Exchange | Neg-risk market orders |
| CTF | CTF Exchange | Selling/redeeming positions |
| CTF | Neg Risk CTF Exchange | Selling/redeeming neg-risk positions |

Approvals are set via the Relay Hub as gasless meta-transactions. The `ClobClient.set_allowances()` method handles this automatically.

## Gasless Relay Transactions

The proxy wallet uses the Relay Hub for gasless transactions. The signer signs a meta-transaction off-chain, and the relayer submits it on-chain (paying gas). This means:

- Users do not need MATIC for gas
- Only USDC.e is needed for trading
- Approvals and order settlements are relayed

## Order Placement

```python
from py_clob_client.clob_types import OrderArgs
from py_clob_client.order_builder.constants import BUY

order_args = OrderArgs(
    price=0.50,
    size=10.0,
    side=BUY,
    token_id=TOKEN_ID,
)
result = clob_client.create_and_post_order(order_args)
```

The `ClobClient` builds the order, signs it with the signer key (using EIP-712 typed data), and posts to `https://clob.polymarket.com`.

## Reference

- [Polymarket CLOB Client (Python)](https://github.com/Polymarket/py-clob-client)
- [Magic Proxy Builder Example](https://github.com/Polymarket/magic-proxy-builder-example)
- [Polymarket CLOB API Docs](https://docs.polymarket.com/)
