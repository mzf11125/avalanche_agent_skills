# Receiving ICM Messages

## The ITeleporterReceiver Interface

Any contract that wants to receive ICM messages must implement this interface:

```solidity
interface ITeleporterReceiver {
    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external;
}
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `sourceBlockchainID` | `bytes32` | The blockchain ID of the chain that sent the message |
| `originSenderAddress` | `address` | The address of the contract/EOA that called `sendCrossChainMessage` on the source chain |
| `message` | `bytes` | The ABI-encoded payload. Decode with `abi.decode()` |

## Access Control: Critical Security Requirement

**Only the Teleporter contract should be able to call `receiveTeleporterMessage`.** Always check `msg.sender`:

```solidity
function receiveTeleporterMessage(
    bytes32 sourceBlockchainID,
    address originSenderAddress,
    bytes calldata message
) external override {
    require(msg.sender == TELEPORTER_ADDRESS, "Only Teleporter");
    // ... your logic
}
```

Without this check, anyone can call your function with arbitrary data.

## Complete Receiver Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract CrossChainReceiver is ITeleporterReceiver {
    address public constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;

    // Store the last received message
    string public lastMessage;
    address public lastSender;
    bytes32 public lastSourceChain;

    event MessageReceived(
        bytes32 indexed sourceBlockchainID,
        address indexed originSender,
        string message
    );

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        // CRITICAL: only Teleporter can call this
        require(msg.sender == TELEPORTER, "Only Teleporter");

        // Decode the payload
        string memory decoded = abi.decode(message, (string));

        // Store and emit
        lastMessage = decoded;
        lastSender = originSenderAddress;
        lastSourceChain = sourceBlockchainID;

        emit MessageReceived(sourceBlockchainID, originSenderAddress, decoded);
    }
}
```

## Decoding Complex Payloads

For structured data, define a struct and use `abi.decode`:

```solidity
struct TransferData {
    address recipient;
    uint256 amount;
    string memo;
}

function receiveTeleporterMessage(
    bytes32 sourceBlockchainID,
    address originSenderAddress,
    bytes calldata message
) external override {
    require(msg.sender == TELEPORTER, "Only Teleporter");
    TransferData memory data = abi.decode(message, (TransferData));
    // use data.recipient, data.amount, data.memo
}
```

On the sender side:
```solidity
bytes memory payload = abi.encode(TransferData({
    recipient: 0x...,
    amount: 1e18,
    memo: "payment"
}));
```

## Handling Failures and Retries

If `receiveTeleporterMessage` reverts, the message is marked as failed. The sender can retry:

```solidity
// On the source chain, retry a failed message
TELEPORTER.retrySendCrossChainMessage(originalMessage);
```

To prevent reverts, use try/catch for risky operations:

```solidity
function receiveTeleporterMessage(...) external override {
    require(msg.sender == TELEPORTER, "Only Teleporter");
    try this._processMessage(message) {
        // success
    } catch {
        // log failure but don't revert — message is consumed
        emit ProcessingFailed(message);
    }
}
```

## Source Chain Validation

Optionally restrict which chains can send messages to your contract:

```solidity
bytes32 public constant ALLOWED_SOURCE_CHAIN = 0x...; // C-Chain blockchain ID

function receiveTeleporterMessage(
    bytes32 sourceBlockchainID,
    address originSenderAddress,
    bytes calldata message
) external override {
    require(msg.sender == TELEPORTER, "Only Teleporter");
    require(sourceBlockchainID == ALLOWED_SOURCE_CHAIN, "Wrong source chain");
    require(originSenderAddress == TRUSTED_SENDER, "Wrong sender");
    // ...
}
```
