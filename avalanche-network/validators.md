# Validator Setup for Avalanche L1s

## Post-Etna (ACP-77): Sovereign Validators

After the Etna upgrade, Avalanche L1s manage their own validator sets independently of the Primary Network. Validators no longer need to stake AVAX on the Primary Network to validate an L1.

## Adding a Primary Network Validator

### Step 1: Get Your Node ID and BLS Credentials

```bash
curl -X POST http://localhost:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNodeID","params":{},"id":1}'
```

Response includes your node ID **and** BLS credentials (required for validator registration):

```json
{
  "jsonrpc": "2.0",
  "result": {
    "nodeID": "NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD",
    "nodePOP": {
      "publicKey": "0x8f95423f7142d00a48e1014a3de8d28907d420dc33b3052a6dee03a3f2941a393c2351e354704ca66a3fc29870282e15",
      "proofOfPossession": "0x86a3ab4c45cfe31cae34c1d06f212434ac71b1be6cfe046c80c162e057614a94a5bc9f1ded1a7029deb0ba4ca7c9b71411e293438691be79c2dbf19d1ca7c3eadb9c756246fc5de5b7b89511c7d7302ae051d9e03d7991138299b5ed6a570a98"
    }
  },
  "id": 1
}
```

### Step 2: Install platform-cli

```bash
curl -sSfL https://build.avax.network/install/platform-cli | sh
```

### Step 3: Generate a Key

```bash
platform keys generate --name mykey
# Or import an existing key:
# platform keys import --name mykey --private-key <PRIVATE_KEY>
```

### Step 4: Add Validator

**Fuji Testnet** (minimum 1 AVAX, minimum 24 hours):

```bash
platform validator add \
  --node-id <YOUR_NODE_ID> \
  --stake 1 \
  --duration 336h \
  --delegation-fee 0.02 \
  --node-endpoint http://<YOUR_NODE_IP>:9650 \
  --key-name mykey \
  --network fuji
```

**Mainnet** (minimum 2,000 AVAX, minimum 2 weeks):

```bash
platform validator add \
  --node-id <YOUR_NODE_ID> \
  --stake 2000 \
  --duration 336h \
  --delegation-fee 0.02 \
  --node-endpoint http://<YOUR_NODE_IP>:9650 \
  --key-name mykey \
  --network mainnet
```

**Mainnet with Ledger** (hardware wallet):

```bash
platform validator add \
  --node-id <YOUR_NODE_ID> \
  --stake 2000 \
  --duration 336h \
  --delegation-fee 0.02 \
  --node-endpoint http://<YOUR_NODE_IP>:9650 \
  --ledger \
  --network mainnet
```

### Key Flags

| Flag | Description |
|------|-------------|
| `--node-id` | Your node's NodeID (from `info.getNodeID`) |
| `--stake` | Amount in AVAX (min 1 on Fuji, 2000 on Mainnet) |
| `--duration` | Validation duration (min `336h` = 14 days on Mainnet, `24h` on Fuji) |
| `--delegation-fee` | Percentage retained by validator (min `0.02` = 2%) |
| `--node-endpoint` | Auto-fetches BLS credentials from your running node |
| `--reward-address` | Optional: specify a different reward address |
| `--network` | `fuji`, `mainnet`, or `local` |
| `--key-name` | Name of the stored key to sign the transaction |
| `--ledger` | Use a Ledger hardware wallet instead of a stored key |

### Step 5: Verify

```bash
# Check pending validators (before start time)
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getPendingValidators","params":{},"id":1}'

# Check current validators (after start time)
curl -X POST http://localhost:9650/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getCurrentValidators","params":{},"id":1}'
```

## Adding a Validator to Your L1

### Using Avalanche CLI

```bash
# Install Avalanche CLI
curl -sSfL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | sh

# Add validator to your L1
avalanche validator addValidator myL1 \
  --node-id NodeID-XXXX \
  --weight 100
```

## Staking Parameters

| Parameter | Mainnet | Fuji |
|-----------|---------|------|
| Min validator stake | 2,000 AVAX | 1 AVAX |
| Min delegator stake | 25 AVAX | 1 AVAX |
| Min validation duration | 2 weeks (336h) | 24 hours |
| Max validation duration | 1 year | 1 year |
| Min delegation fee | 2% | 2% |
| Max validator weight | min(3,000,000 AVAX, 5× own stake) | same |
| Uptime for reward | ≥ 80% | ≥ 80% |

## Tracking an L1

For a node to validate an L1, it must track the L1's blockchain. Add to AvalancheGo config:

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
- `staker.crt` - TLS certificate
- `staker.key` - TLS private key
- `signer.key` - BLS signing key (used for Warp/ICM)

**Back these up.** Losing them means losing your node identity. You do not need AVAX funds on your validating node - keep most funds in cold storage.

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
