# TeleporterRegistry

## What Is TeleporterRegistry?

TeleporterRegistry tracks all deployed versions of the Teleporter contract. Instead of hardcoding the Teleporter address, use the registry to always get the latest version. This is important for upgrades — when a new Teleporter version is deployed, your contract automatically uses it.

## Address

`TeleporterRegistry` is **not** deployed at a universal address. The canonical addresses are:

| Network | Address |
|---------|---------|
| Mainnet C-Chain | `0x7C43605E14F391720e1b37E49C78C4b03A488d98` |
| Fuji C-Chain | `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228` |
| Custom L1s | Must be deployed separately — see [Deploy TeleporterRegistry to a Subnet](https://github.com/ava-labs/icm-services/blob/main/README.md#deploy-teleporter-to-a-subnet) |

## Interface

```solidity
interface ITeleporterRegistry {
    function getLatestTeleporter() external view returns (ITeleporterMessenger);
    function getTeleporterFromVersion(uint256 version) external view returns (ITeleporterMessenger);
    function getLatestVersion() external view returns (uint256);
    function getVersionFromAddress(address teleporterAddress) external view returns (uint256);
    function isTeleporterAddress(address addr) external view returns (bool);
}
```

## Using the Registry in Your Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/TeleporterRegistryApp.sol";

// Extend TeleporterRegistryApp for built-in registry support
contract MyApp is TeleporterRegistryApp {
    constructor(address registryAddress)
        TeleporterRegistryApp(registryAddress) {}

    function sendMessage(bytes32 destChain, address destAddr, bytes calldata payload)
        external returns (bytes32)
    {
        // Always uses the latest Teleporter version
        return _getTeleporterMessenger().sendCrossChainMessage(
            TeleporterMessageInput({
                destinationBlockchainID: destChain,
                destinationAddress: destAddr,
                feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
                requiredGasLimit: 100_000,
                allowedRelayerAddresses: new address[](0),
                message: payload
            })
        );
    }

    // Implement receiver — TeleporterRegistryApp handles the msg.sender check
    function _receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes memory message
    ) internal override {
        // your logic here
    }
}
```

## Manual Registry Usage

```solidity
ITeleporterRegistry constant REGISTRY =
    ITeleporterRegistry(0x7C43605E14F391720e1b37E49C78C4b03A488d98); // Mainnet C-Chain
    // Fuji C-Chain: 0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228
    // Custom L1s: deploy your own registry

function sendMessage(...) external {
    ITeleporterMessenger teleporter = REGISTRY.getLatestTeleporter();
    teleporter.sendCrossChainMessage(...);
}

// For receiving: check that msg.sender is a valid Teleporter version
function receiveTeleporterMessage(...) external {
    require(REGISTRY.isTeleporterAddress(msg.sender), "Not a Teleporter");
    // ...
}
```
