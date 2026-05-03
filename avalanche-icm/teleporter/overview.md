# Teleporter Overview

## What Is Teleporter?

Teleporter is the official high-level SDK for Avalanche ICM. It consists of:

- **`TeleporterMessenger`** — The main contract. Deployed at a deterministic address on every Subnet-EVM chain.
- **`TeleporterRegistry`** — Tracks Teleporter versions. Use this to always get the current address.
- **`ITeleporterMessenger`** — Solidity interface for sending messages.
- **`ITeleporterReceiver`** — Solidity interface for receiving messages.

## Contract Addresses

### Mainnet

| Contract | Address |
|----------|---------|
| TeleporterMessenger v1.0.0 | `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf` |
| TeleporterRegistry | `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228` |

### Fuji Testnet

| Contract | Address |
|----------|---------|
| TeleporterMessenger v1.0.0 | `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf` |
| TeleporterRegistry | `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228` |

These addresses are the same on all Subnet-EVM chains (deterministic deployment).

## Installing the Teleporter Contracts Package

```bash
npm install @avalabs/teleporter-contracts
```

Or with Foundry:
```bash
forge install ava-labs/teleporter
```

## Importing in Solidity

```solidity
import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/TeleporterRegistry.sol";
```

## ITeleporterMessenger Interface Summary

```solidity
interface ITeleporterMessenger {
    // Send a cross-chain message
    function sendCrossChainMessage(
        TeleporterMessageInput calldata messageInput
    ) external returns (bytes32 messageID);

    // Retry a failed message delivery
    function retrySendCrossChainMessage(
        TeleporterMessage calldata message
    ) external;

    // Check if a message has been received
    function messageReceived(
        bytes32 sourceBlockchainID,
        uint256 messageNonce
    ) external view returns (bool);

    // Get the fee info for a message
    function getFeeInfo(
        bytes32 messageID
    ) external view returns (address feeTokenAddress, uint256 amount);
}
```

## Key Structs

```solidity
struct TeleporterMessageInput {
    bytes32 destinationBlockchainID;  // Target chain's blockchain ID
    address destinationAddress;        // Target contract address
    TeleporterFeeInfo feeInfo;         // Fee configuration
    uint256 requiredGasLimit;          // Gas limit for delivery tx
    address[] allowedRelayerAddresses; // Empty = any relayer allowed
    bytes message;                     // ABI-encoded payload
}

struct TeleporterFeeInfo {
    address feeTokenAddress; // ERC-20 token for fee (address(0) = no fee)
    uint256 amount;          // Fee amount (0 = no fee)
}
```
