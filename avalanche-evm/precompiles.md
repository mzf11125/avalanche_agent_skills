# Subnet-EVM Precompiles

## Overview

Precompiles are built-in contracts at fixed addresses. They extend EVM functionality without deploying Solidity code. Subnet-EVM precompiles use an AllowList pattern: addresses can be `Admin`, `Manager`, or `Enabled`.

- **Admin**: can add/remove other admins, managers, and enabled addresses
- **Manager**: can add/remove enabled addresses
- **Enabled**: has the specific permission (e.g., can deploy contracts)

## ContractDeployerAllowList

**Address**: `0x0200000000000000000000000000000000000000`

Restricts which addresses can deploy contracts.

```solidity
interface IAllowList {
    function setAdmin(address addr) external;
    function setManager(address addr) external;
    function setEnabled(address addr) external;
    function setNone(address addr) external;
    function readAllowList(address addr) external view returns (uint256);
    // Returns: 0=None, 1=Enabled, 2=Admin, 3=Manager
}

IAllowList constant DEPLOYER_ALLOW_LIST =
    IAllowList(0x0200000000000000000000000000000000000000);

// Allow an address to deploy contracts
DEPLOYER_ALLOW_LIST.setEnabled(0xDeployerAddress);
```

## ContractNativeMinter

**Address**: `0x0200000000000000000000000000000000000001`

Mint native tokens (the chain's gas token) to any address.

```solidity
interface INativeMinter {
    function mintNativeCoin(address addr, uint256 amount) external;
    function setAdmin(address addr) external;
    function setEnabled(address addr) external;
    function setNone(address addr) external;
    function readAllowList(address addr) external view returns (uint256);
}

INativeMinter constant MINTER =
    INativeMinter(0x0200000000000000000000000000000000000001);

// Mint 100 native tokens to an address
MINTER.mintNativeCoin(recipient, 100 ether);
```

## TxAllowList

**Address**: `0x0200000000000000000000000000000000000002`

Restricts which addresses can send transactions (permissioned chain).

```solidity
IAllowList constant TX_ALLOW_LIST =
    IAllowList(0x0200000000000000000000000000000000000002);

TX_ALLOW_LIST.setEnabled(0xUserAddress);
```

## FeeManager

**Address**: `0x0200000000000000000000000000000000000003`

Dynamically update fee configuration without a hard fork.

```solidity
interface IFeeManager {
    struct FeeConfig {
        uint256 gasLimit;
        uint256 targetBlockRate;
        uint256 minBaseFee;
        uint256 targetGas;
        uint256 baseFeeChangeDenominator;
        uint256 minBlockGasCost;
        uint256 maxBlockGasCost;
        uint256 blockGasCostStep;
    }
    function setFeeConfig(FeeConfig calldata config) external;
    function getFeeConfig() external view returns (FeeConfig memory);
    function getFeeConfigLastChangedAt() external view returns (uint256);
    function setAdmin(address addr) external;
    function setEnabled(address addr) external;
}

IFeeManager constant FEE_MANAGER =
    IFeeManager(0x0200000000000000000000000000000000000003);

// Lower the minimum base fee
IFeeManager.FeeConfig memory config = FEE_MANAGER.getFeeConfig();
config.minBaseFee = 1 gwei;
FEE_MANAGER.setFeeConfig(config);
```

## RewardManager

**Address**: `0x0200000000000000000000000000000000000004`

Configure where transaction fees go (burn, validators, or a custom address).

```solidity
interface IRewardManager {
    function setRewardAddress(address addr) external;
    function allowFeeRecipients() external;
    function disableRewards() external; // burn fees
    function currentRewardAddress() external view returns (address);
    function areFeeRecipientsAllowed() external view returns (bool);
}

IRewardManager constant REWARD_MANAGER =
    IRewardManager(0x0200000000000000000000000000000000000004);

// Send all fees to a treasury contract
REWARD_MANAGER.setRewardAddress(0xTreasuryAddress);
```

## Warp (ICM)

**Address**: `0x0200000000000000000000000000000000000005`

Low-level AWM interface. In practice, use the Teleporter contract instead of calling Warp directly.

```solidity
interface IWarpMessenger {
    function sendWarpMessage(bytes calldata payload) external returns (bytes32 messageID);
    function getVerifiedWarpMessage(uint32 index) external view returns (WarpMessage memory, bool valid);
    function getBlockchainID() external view returns (bytes32);
}
```

## Importing Precompile Interfaces

```bash
npm install @avalabs/subnet-evm-contracts
```

```solidity
import "@avalabs/subnet-evm-contracts/contracts/interfaces/INativeMinter.sol";
import "@avalabs/subnet-evm-contracts/contracts/interfaces/IFeeManager.sol";
import "@avalabs/subnet-evm-contracts/contracts/interfaces/IAllowList.sol";
```
