# genesis.json Configuration

## Minimal Genesis

```json
{
  "config": {
    "chainId": 12345,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "muirGlacierBlock": 0,
    "subnetEVMTimestamp": 0,
    "feeConfig": {
      "gasLimit": 8000000,
      "minBaseFee": 25000000000,
      "targetGas": 15000000,
      "baseFeeChangeDenominator": 36,
      "minBlockGasCost": 0,
      "maxBlockGasCost": 1000000,
      "targetBlockRate": 2,
      "blockGasCostStep": 200000
    }
  },
  "alloc": {
    "0xYourAddress": {
      "balance": "0x52B7D2DCC80CD2E4000000"
    }
  },
  "nonce": "0x0",
  "timestamp": "0x0",
  "extraData": "0x00",
  "gasLimit": "0x7A1200",
  "difficulty": "0x0",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

## `config` Fields

| Field | Description |
|-------|-------------|
| `chainId` | EVM chain ID. Must be unique. Pick a number not in use at [chainlist.org](https://chainlist.org) |
| `subnetEVMTimestamp` | Timestamp to activate Subnet-EVM features. Set to `0` for new chains |
| `feeConfig` | Initial fee configuration (can be changed via FeeManager precompile) |

## `feeConfig` Fields

| Field | Default | Description |
|-------|---------|-------------|
| `gasLimit` | `8000000` | Max gas per block |
| `minBaseFee` | `25000000000` | Minimum base fee in wei (25 gwei) |
| `targetGas` | `15000000` | Target gas usage per block for EIP-1559 |
| `baseFeeChangeDenominator` | `36` | Controls how fast base fee adjusts |
| `targetBlockRate` | `2` | Target seconds per block |
| `minBlockGasCost` | `0` | Minimum gas cost per block |
| `maxBlockGasCost` | `1000000` | Maximum gas cost per block |
| `blockGasCostStep` | `200000` | How much block gas cost changes per second |

## Enabling Precompiles in Genesis

Add a `contractDeployerAllowListConfig`, `contractNativeMinterConfig`, etc. under `config`:

```json
{
  "config": {
    "chainId": 12345,
    "subnetEVMTimestamp": 0,
    "contractNativeMinterConfig": {
      "blockTimestamp": 0,
      "adminAddresses": ["0xYourAdminAddress"]
    },
    "feeManagerConfig": {
      "blockTimestamp": 0,
      "adminAddresses": ["0xYourAdminAddress"]
    },
    "contractDeployerAllowListConfig": {
      "blockTimestamp": 0,
      "adminAddresses": ["0xYourAdminAddress"],
      "enabledAddresses": ["0xDeployerAddress"]
    }
  }
}
```

## `alloc` — Pre-funded Accounts

```json
"alloc": {
  "0xAddress1": { "balance": "0x52B7D2DCC80CD2E4000000" },
  "0xAddress2": { "balance": "0xDE0B6B3A7640000" }
}
```

Convert decimal to hex: `python3 -c "print(hex(1000 * 10**18))"` → `0x3635c9adc5dea00000`

## Warp (ICM) Configuration

To enable ICM on your L1, include the Warp precompile config:

```json
"warpConfig": {
  "blockTimestamp": 0,
  "quorumNumerator": 67
}
```

`quorumNumerator`: percentage of stake weight required to sign a Warp message (default 67 = 67%).
