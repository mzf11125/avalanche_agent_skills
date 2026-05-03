# Relayer Configuration Reference

## Top-Level Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `log-level` | string | `"info"` | Log verbosity: `debug`, `info`, `warn`, `error` |
| `p-chain-api` | object | required | P-Chain API endpoint config |
| `info-api` | object | required | AvalancheGo info API endpoint config |
| `source-blockchains` | array | required | Chains to monitor for outgoing messages |
| `destination-blockchains` | array | required | Chains to deliver messages to |
| `metrics-port` | number | `9090` | Prometheus metrics port |
| `db-path` | string | `"./awm-relayer-db"` | Path to relayer state database |

## API Endpoint Object

```json
{
  "base-url": "https://api.avax.network",
  "query-parameters": {},
  "http-headers": { "Authorization": "Bearer token" }
}
```

## Source Blockchain Object

```json
{
  "subnet-id": "11111111111111111111111111111111LpoYY",
  "blockchain-id": "yH8D7ThNJkxmtkuv2jgBa4P1Rn3Qpr4muVULEqberX5drBMCm",
  "vm": "evm",
  "rpc-endpoint": {
    "base-url": "https://api.avax.network/ext/bc/C/rpc"
  },
  "ws-endpoint": {
    "base-url": "wss://api.avax.network/ext/bc/C/ws"
  },
  "message-contracts": {
    "0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf": {
      "message-format": "teleporter",
      "settings": {
        "reward-address": "0xYourAddress"
      }
    }
  }
}
```

| Field | Description |
|-------|-------------|
| `subnet-id` | The subnet ID (use `11111111111111111111111111111111LpoYY` for primary network) |
| `blockchain-id` | The blockchain ID in base58 |
| `vm` | Always `"evm"` for Subnet-EVM chains |
| `rpc-endpoint` | HTTP RPC endpoint |
| `ws-endpoint` | WebSocket endpoint (for event subscriptions) |
| `message-contracts` | Map of Teleporter contract address → format config |

## Destination Blockchain Object

```json
{
  "subnet-id": "YOUR_SUBNET_ID",
  "blockchain-id": "YOUR_BLOCKCHAIN_ID",
  "vm": "evm",
  "rpc-endpoint": {
    "base-url": "https://your-l1-rpc/ext/bc/CHAIN/rpc"
  },
  "account-private-key": "0x...",
  "kms-key-id": "",
  "kms-aws-region": ""
}
```

| Field | Description |
|-------|-------------|
| `account-private-key` | Private key of the relayer account (needs native token for gas) |
| `kms-key-id` | AWS KMS key ID (alternative to private key for production) |
| `kms-aws-region` | AWS region for KMS |

## Complete Bidirectional Config Example

```json
{
  "log-level": "info",
  "p-chain-api": { "base-url": "https://api.avax.network" },
  "info-api": { "base-url": "https://api.avax.network" },
  "metrics-port": 9090,
  "db-path": "/data/awm-relayer-db",
  "source-blockchains": [
    {
      "subnet-id": "11111111111111111111111111111111LpoYY",
      "blockchain-id": "2q9e4r6Mu3U68nU1fYjgbR6JvwrRx36CohpAX5UQxse55x1Q5",
      "vm": "evm",
      "rpc-endpoint": { "base-url": "https://api.avax.network/ext/bc/C/rpc" },
      "ws-endpoint": { "base-url": "wss://api.avax.network/ext/bc/C/ws" },
      "message-contracts": {
        "0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf": {
          "message-format": "teleporter",
          "settings": { "reward-address": "0xRelayerRewardAddress" }
        }
      }
    },
    {
      "subnet-id": "YOUR_L1_SUBNET_ID",
      "blockchain-id": "YOUR_L1_BLOCKCHAIN_ID",
      "vm": "evm",
      "rpc-endpoint": { "base-url": "https://your-l1-rpc/ext/bc/CHAIN/rpc" },
      "ws-endpoint": { "base-url": "wss://your-l1-rpc/ext/bc/CHAIN/ws" },
      "message-contracts": {
        "0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf": {
          "message-format": "teleporter",
          "settings": { "reward-address": "0xRelayerRewardAddress" }
        }
      }
    }
  ],
  "destination-blockchains": [
    {
      "subnet-id": "YOUR_L1_SUBNET_ID",
      "blockchain-id": "YOUR_L1_BLOCKCHAIN_ID",
      "vm": "evm",
      "rpc-endpoint": { "base-url": "https://your-l1-rpc/ext/bc/CHAIN/rpc" },
      "account-private-key": "0xRelayerPrivateKey"
    },
    {
      "subnet-id": "11111111111111111111111111111111LpoYY",
      "blockchain-id": "2q9e4r6Mu3U68nU1fYjgbR6JvwrRx36CohpAX5UQxse55x1Q5",
      "vm": "evm",
      "rpc-endpoint": { "base-url": "https://api.avax.network/ext/bc/C/rpc" },
      "account-private-key": "0xRelayerPrivateKey"
    }
  ]
}
```
