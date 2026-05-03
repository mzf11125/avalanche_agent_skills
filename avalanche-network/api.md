# AvalancheGo API Reference

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `/ext/bc/C/rpc` | C-Chain EVM JSON-RPC |
| `/ext/bc/C/ws` | C-Chain WebSocket |
| `/ext/bc/P` | P-Chain API |
| `/ext/bc/X` | X-Chain API |
| `/ext/info` | Node info API |
| `/ext/health` | Health check |
| `/ext/metrics` | Prometheus metrics |
| `/ext/bc/YOUR_CHAIN/rpc` | Custom L1 EVM RPC |

## Info API

```bash
# Get node ID
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNodeID","params":{},"id":1}'

# Get node version
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNodeVersion","params":{},"id":1}'

# Get network name
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNetworkName","params":{},"id":1}'

# Check if chain is bootstrapped
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.isBootstrapped","params":{"chain":"C"},"id":1}'

# Get peers
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.peers","params":{},"id":1}'
```

## P-Chain API

```bash
# Get current validators for a subnet
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc":"2.0",
    "method":"platform.getCurrentValidators",
    "params":{"subnetID":"YOUR_SUBNET_ID"},
    "id":1
  }'

# Get all blockchains
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getBlockchains","params":{},"id":1}'

# Get blockchain ID for a chain alias
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getBlockchainID","params":{"alias":"C"},"id":1}'

# Get AVAX balance
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc":"2.0",
    "method":"platform.getBalance",
    "params":{"addresses":["P-avax1..."]},
    "id":1
  }'
```

## C-Chain EVM API

Standard Ethereum JSON-RPC - use any Ethereum library:

```bash
# Get block number
curl -X POST http://localhost:9650/ext/bc/C/rpc \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Get balance
curl -X POST http://localhost:9650/ext/bc/C/rpc \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xAddress","latest"],"id":1}'
```

## Health API

```bash
curl -X POST http://localhost:9650/ext/health \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"health.health","params":{},"id":1}'
```

Response: `{"checks": {...}, "healthy": true}`

## Public API Endpoints

| Network | URL |
|---------|-----|
| Mainnet C-Chain | `https://api.avax.network/ext/bc/C/rpc` |
| Fuji C-Chain | `https://api.avax-test.network/ext/bc/C/rpc` |
| Mainnet P-Chain | `https://api.avax.network/ext/bc/P` |
| Fuji P-Chain | `https://api.avax-test.network/ext/bc/P` |
