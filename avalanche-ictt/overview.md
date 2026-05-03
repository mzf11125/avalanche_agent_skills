# ICTT Overview

## What Is ICTT?

ICTT (Interchain Token Transfer) is the standard protocol for bridging tokens between Avalanche L1s. It is built on top of ICM/Teleporter and provides a secure, audited implementation for cross-chain token transfers.

GitHub: [ava-labs/avalanche-interchain-token-transfer](https://github.com/ava-labs/avalanche-interchain-token-transfer)

## TokenHome / TokenRemote Pattern

ICTT uses a hub-and-spoke model:

```
Chain A (Home)          Chain B (Remote)
┌─────────────┐         ┌──────────────────┐
│  TokenHome  │◄───────►│  TokenRemote     │
│             │  ICM    │                  │
│ Locks/burns │         │ Mints/burns      │
│ tokens      │         │ wrapped tokens   │
└─────────────┘         └──────────────────┘
```

- **TokenHome**: deployed on the chain where the token originates. Locks (or burns) tokens when they are bridged out.
- **TokenRemote**: deployed on destination chains. Mints wrapped tokens when a transfer arrives; burns them when transferring back.

## Contract Variants

| Contract | Use Case |
|----------|----------|
| `ERC20TokenHome` | Bridge an existing ERC-20 token from its home chain |
| `NativeTokenHome` | Bridge the native gas token from its home chain |
| `ERC20TokenRemote` | Receive bridged tokens as an ERC-20 on the remote chain |
| `NativeTokenRemote` | Receive bridged tokens as the native gas token on the remote chain |

## Transfer Flow

### Home → Remote (Lock & Mint)

1. User calls `send()` on `TokenHome` with amount and destination
2. `TokenHome` locks (ERC-20) or receives (native) the tokens
3. `TokenHome` sends an ICM message to the `TokenRemote`
4. Relayer delivers the message
5. `TokenRemote` mints wrapped tokens to the recipient

### Remote → Home (Burn & Unlock)

1. User calls `send()` on `TokenRemote`
2. `TokenRemote` burns the wrapped tokens
3. `TokenRemote` sends an ICM message to `TokenHome`
4. Relayer delivers the message
5. `TokenHome` unlocks (ERC-20) or sends (native) tokens to the recipient

## Multi-Hop

Tokens can be transferred between two remote chains by routing through the home chain:

```
Remote A → Home → Remote B
```

This is handled automatically by the ICTT contracts.

## Security Model

- The home chain is the source of truth for token supply
- Total supply on all remote chains ≤ locked supply on home chain
- Each TokenRemote must be registered with the TokenHome before transfers work
- Uses ICM's security guarantees (BLS multi-signatures, 67% stake weight)
