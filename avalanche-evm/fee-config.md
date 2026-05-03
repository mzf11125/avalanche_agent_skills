# Fee Configuration

## Default Fee Parameters (C-Chain)

| Parameter | Value |
|-----------|-------|
| Min base fee | 25 gwei |
| Target block rate | 2 seconds |
| Gas limit | 15,000,000 |
| Target gas per block | 15,000,000 |

## Custom Fee Token

On a custom L1, the native gas token is whatever you configure in genesis. It does not have to be AVAX. The token is pre-allocated in `alloc` and can be minted via the `ContractNativeMinter` precompile.

## Updating Fees Dynamically (FeeManager Precompile)

```solidity
import "@avalabs/subnet-evm-contracts/contracts/interfaces/IFeeManager.sol";

IFeeManager constant FEE_MANAGER =
    IFeeManager(0x0200000000000000000000000000000000000003);

function lowerFees() external onlyAdmin {
    IFeeManager.FeeConfig memory config = FEE_MANAGER.getFeeConfig();
    config.minBaseFee = 1 gwei;       // lower minimum
    config.targetBlockRate = 1;        // 1 second blocks
    FEE_MANAGER.setFeeConfig(config);
}
```

## Zero-Fee Chain

For a private L1 where you want free transactions:

```json
"feeConfig": {
  "gasLimit": 20000000,
  "minBaseFee": 0,
  "targetGas": 100000000,
  "baseFeeChangeDenominator": 48,
  "minBlockGasCost": 0,
  "maxBlockGasCost": 0,
  "targetBlockRate": 2,
  "blockGasCostStep": 0
}
```

> Note: Setting `minBaseFee` to 0 means transactions can have zero gas price. This is fine for private/permissioned chains.

## Fee Distribution (RewardManager)

By default, fees go to the block producer. Configure alternatives:

```solidity
IRewardManager constant REWARD_MANAGER =
    IRewardManager(0x0200000000000000000000000000000000000004);

// Option 1: Send fees to a specific address (e.g., DAO treasury)
REWARD_MANAGER.setRewardAddress(0xTreasuryAddress);

// Option 2: Allow validators to set their own fee recipient
REWARD_MANAGER.allowFeeRecipients();

// Option 3: Burn all fees
REWARD_MANAGER.disableRewards();
```

## EIP-1559 on Subnet-EVM

Subnet-EVM uses EIP-1559 by default. Transactions include:
- `maxFeePerGas`: maximum total fee per gas
- `maxPriorityFeePerGas`: tip to block producer

The base fee adjusts based on block fullness relative to `targetGas`.
