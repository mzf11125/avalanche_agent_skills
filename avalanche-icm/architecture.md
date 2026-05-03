# ICM Architecture

## Overview

Avalanche ICM (Interchain Messaging) is a native cross-chain communication protocol. It has two layers:

1. **AWM (Avalanche Warp Messaging)** — the low-level protocol built into Subnet-EVM
2. **Teleporter** — the high-level developer SDK built on top of AWM

## AWM: The Low-Level Protocol

AWM uses BLS (Boneh–Lynn–Shacham) multi-signatures. Every Avalanche validator has a BLS key pair registered on the P-Chain.

### How AWM Works

1. A contract on Chain A calls the `WarpMessenger` precompile to emit a Warp message
2. The message is included in a block on Chain A
3. Validators of Chain A sign the message with their BLS keys
4. An off-chain relayer collects enough signatures to meet the threshold (typically 67% of stake weight)
5. The relayer aggregates the signatures into a single compact BLS aggregate signature
6. The relayer submits the aggregate signature to Chain B
7. Chain B's Subnet-EVM verifies the aggregate signature against Chain A's known validator set (fetched from P-Chain)
8. If valid, the message is delivered to the destination contract

### Trust Model

AWM is **trust-minimized**: the same validators securing Chain A also sign its outgoing messages. There is no external bridge operator. An attacker would need to compromise 67%+ of Chain A's stake weight to forge a message — the same threshold needed to attack the chain itself.

This is fundamentally different from traditional bridges, which rely on a separate multisig or committee.

## Message Lifecycle (ASCII Diagram)

```
Chain A (Source)                    Off-Chain                    Chain B (Destination)
─────────────────                   ─────────                    ────────────────────
Contract calls                          │                              │
sendCrossChainMessage()                 │                              │
        │                               │                              │
        ▼                               │                              │
WarpMessenger precompile                │                              │
emits WarpMessageSent event             │                              │
        │                               │                              │
        ▼                               │                              │
Block finalized on Chain A              │                              │
        │                               │                              │
        │──── Relayer detects ─────────▶│                              │
        │     WarpMessageSent           │                              │
        │                               │                              │
        │                    Relayer queries validators                │
        │                    for BLS signatures                        │
        │                               │                              │
        │                    Aggregate signatures                      │
        │                    (BLS multi-sig)                           │
        │                               │                              │
        │                               │──── Submit to Chain B ──────▶│
        │                               │     (aggregate sig + msg)    │
        │                               │                              │
        │                               │                    Subnet-EVM verifies
        │                               │                    BLS sig vs P-Chain
        │                               │                    validator set
        │                               │                              │
        │                               │                    receiveTeleporterMessage()
        │                               │                    called on destination
        │                               │                    contract
        │                               │                              │
        │                               │                    ✓ Message delivered
```

## Teleporter: The High-Level SDK

Teleporter wraps AWM with a developer-friendly interface:

- **`ITeleporterMessenger`**: The main contract. Call `sendCrossChainMessage()` to send.
- **`ITeleporterReceiver`**: Interface your contract implements to receive messages.
- **`TeleporterRegistry`**: Tracks the latest Teleporter version. Use this instead of hardcoding the address.

### What Teleporter Adds Over Raw AWM

| Feature | Raw AWM | Teleporter |
|---------|---------|------------|
| Message encoding | Manual | Handled |
| Fee management | Manual | Built-in |
| Retry/receipt tracking | Manual | Built-in |
| Allowed relayer filtering | Manual | Built-in |
| Upgradability | N/A | TeleporterRegistry |
| Developer API | Low-level precompile | Simple Solidity interface |

## Relayer

The AWM relayer is an off-chain Go service (`github.com/ava-labs/awm-relayer`). It:

- Monitors source chains for `WarpMessageSent` events
- Queries validators for BLS signatures
- Aggregates signatures
- Submits to destination chains

Without a running relayer, messages are never delivered. You can:
1. Run your own relayer (see `relayer/setup.md`)
2. Use a third-party relayer service
3. Use AvaCloud's managed relayer

## Comparison with Traditional Bridges

| Property | Traditional Bridge | Avalanche ICM |
|----------|-------------------|---------------|
| Trust | Bridge operator multisig | Source chain validators |
| Attack surface | Bridge contract + operator keys | 67%+ of source chain stake |
| Historical hacks | Many ($2B+ lost) | None |
| Latency | Minutes to hours | ~30 seconds |
| Cost | High (bridge fees) | Low (relayer gas only) |
| Customization | Limited | Full (any message payload) |
