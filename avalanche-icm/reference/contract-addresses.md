# ICM Contract Addresses

## Teleporter Messenger

Same address on all Subnet-EVM chains (mainnet and Fuji):

```
0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf
```

## TeleporterRegistry

`TeleporterRegistry` is **not** deployed at a universal address. Canonical addresses:

| Network | Address |
|---------|---------|
| Mainnet C-Chain | `0x7C43605E14F391720e1b37E49C78C4b03A488d98` |
| Fuji C-Chain | `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228` |
| Custom L1s | Must be deployed separately |

## WAVAX (Wrapped AVAX) — Fee Token

| Network | Address |
|---------|---------|
| Mainnet C-Chain | `0xB31f66AA3C1e785363F0875A1B74E27b85FD66c7` |
| Fuji C-Chain | `0xd00ae08403B9bbb9124bB305C09058E32C39A48c` |

## C-Chain Blockchain IDs

| Network | Blockchain ID (base58) | Blockchain ID (bytes32) |
|---------|------------------------|-------------------------|
| Mainnet | `2q9e4r6Mu3U68nU1fYjgbR6JvwrRx36CohpAX5UQxse55x1Q5` | `0x9f3be606497285d0ffbb5ac9ba24aa60346a9b1812479ed66cb329f394a4b1c7` |
| Fuji | `yH8D7ThNJkxmtkuv2jgBa4P1Rn3Qpr4muVULEqberX5drBMCm` | `0x7fc93d85c6d62be589f5f8f4f0e7e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5` |

> **Note**: Always verify blockchain IDs from the official AvalancheGo API — they are derived from genesis hashes and are chain-specific.

## Verifying Addresses On-Chain

```bash
# Verify Teleporter is deployed at the expected address
cast code 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf --rpc-url https://api.avax.network/ext/bc/C/rpc
# Should return non-empty bytecode

# Check latest Teleporter version via registry (Mainnet C-Chain)
cast call 0x7C43605E14F391720e1b37E49C78C4b03A488d98 \
  "getLatestVersion()(uint256)" \
  --rpc-url https://api.avax.network/ext/bc/C/rpc
```

## Source

- [Teleporter GitHub](https://github.com/ava-labs/teleporter)
- [AWM Relayer GitHub](https://github.com/ava-labs/awm-relayer)
