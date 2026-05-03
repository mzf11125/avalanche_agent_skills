# Validator Setup for Avalanche L1s

## Post-Etna (ACP-77): Sovereign Validators

After the Etna upgrade, Avalanche L1s manage their own validator sets independently of the Primary Network. Validators no longer need to stake AVAX on the Primary Network to validate an L1.

## Adding a Validator to Your L1

### Using Avalanche CLI

```bash
# Install Avalanche CLI
curl -sSfL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | sh

# Add validator to your L1
avalanche validator addValidator myL1 \
  --node-id NodeID-XXXX \
  --weight 100 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2025-01-01T00:00:00Z
```

### Using the P-Chain API

```bash
# Get your node ID
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNodeID","params":{},"id":1}'

# Add validator via P-Chain API
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "method": "platform.addSubnetValidator",
    "params": {
      "nodeID": "NodeID-XXXX",
      "subnetID": "YOUR_SUBNET_ID",
      "startTime": 1704067200,
      "endTime": 1735689600,
      "weight": 100,
      "username": "YOUR_USERNAME",
      "password": "YOUR_PASSWORD"
    },
    "id": 1
  }'
```

## Tracking an L1

For a node to validate an L1, it must track the L1's blockchain:

```json
// /etc/avalanchego/chains/YOUR_BLOCKCHAIN_ID/config.json
{
  "pruning-enabled": false
}
```

Add to AvalancheGo config:
```json
{
  "track-subnets": "YOUR_SUBNET_ID"
}
```

Or via flag:
```bash
avalanchego --track-subnets=YOUR_SUBNET_ID
```

## Staking Keys

AvalancheGo generates staking keys on first run at `~/.avalanchego/staking/`:
- `staker.crt` — TLS certificate
- `staker.key` — TLS private key
- `signer.key` — BLS signing key (used for Warp/ICM)

**Back these up.** Losing them means losing your node identity.

## Checking Validator Status

```bash
# List current validators for your L1
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "method": "platform.getCurrentValidators",
    "params": { "subnetID": "YOUR_SUBNET_ID" },
    "id": 1
  }'
```

## Minimum Stake (Primary Network)

For Primary Network validation (required pre-Etna for L1 validators):
- Mainnet: 2,000 AVAX
- Fuji: 1 AVAX (testnet)
- Minimum duration: 2 weeks
- Maximum duration: 1 year
