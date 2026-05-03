# AWM Relayer Setup

## What the Relayer Does

The AWM relayer (`awm-relayer`) is an off-chain Go service that:
1. Monitors source chains for outgoing Warp messages
2. Queries validators for BLS signatures
3. Aggregates signatures into a compact BLS multi-signature
4. Submits the signed message to the destination chain

Without a running relayer, ICM messages are never delivered.

## Installation

### Option 1: Docker (Recommended)

```bash
docker pull avaplatform/awm-relayer:latest
```

### Option 2: Binary

```bash
# Download latest release
curl -L https://github.com/ava-labs/awm-relayer/releases/latest/download/awm-relayer-linux-amd64 \
  -o awm-relayer
chmod +x awm-relayer
```

### Option 3: Build from Source

```bash
git clone https://github.com/ava-labs/awm-relayer
cd awm-relayer
go build -o awm-relayer ./cmd/awm-relayer
```

## Running on Fuji (Quick Start)

Create a config file `relayer-config.json`:

```json
{
  "log-level": "info",
  "p-chain-api": {
    "base-url": "https://api.avax-test.network",
    "query-parameters": {},
    "http-headers": null
  },
  "info-api": {
    "base-url": "https://api.avax-test.network",
    "query-parameters": {},
    "http-headers": null
  },
  "source-blockchains": [
    {
      "subnet-id": "11111111111111111111111111111111LpoYY",
      "blockchain-id": "yH8D7ThNJkxmtkuv2jgBa4P1Rn3Qpr4muVULEqberX5drBMCm",
      "vm": "evm",
      "rpc-endpoint": {
        "base-url": "https://api.avax-test.network/ext/bc/C/rpc"
      },
      "ws-endpoint": {
        "base-url": "wss://api.avax-test.network/ext/bc/C/ws"
      },
      "message-contracts": {
        "0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf": {
          "message-format": "teleporter",
          "settings": {
            "reward-address": "0xYourRelayerRewardAddress"
          }
        }
      }
    }
  ],
  "destination-blockchains": [
    {
      "subnet-id": "YOUR_L1_SUBNET_ID",
      "blockchain-id": "YOUR_L1_BLOCKCHAIN_ID",
      "vm": "evm",
      "rpc-endpoint": {
        "base-url": "https://your-l1-rpc.example.com/ext/bc/YOUR_CHAIN/rpc"
      },
      "kms-key-id": "",
      "kms-aws-region": "",
      "account-private-key": "YOUR_RELAYER_PRIVATE_KEY"
    }
  ]
}
```

Run:
```bash
# Docker
docker run -v $(pwd)/relayer-config.json:/config.json \
  avaplatform/awm-relayer:latest --config-file /config.json

# Binary
./awm-relayer --config-file relayer-config.json
```

## Running on Mainnet

Same config structure, replace:
- `api.avax-test.network` → `api.avax.network`
- Fuji blockchain IDs → mainnet blockchain IDs
- Use a funded mainnet account for the relayer

## Relayer Account Requirements

The relayer account needs:
- Enough native token on each destination chain to pay for delivery transactions
- Typically 0.1–1 AVAX equivalent per chain for initial funding
- Monitor balance and top up as needed

## AvaCloud Managed Relayer

AvaCloud offers a managed relayer service — no infrastructure to run. Configure it in the AvaCloud console under your L1 settings.
