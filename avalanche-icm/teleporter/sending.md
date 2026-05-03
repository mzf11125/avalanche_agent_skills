# Sending ICM Messages

## The `sendCrossChainMessage` Function

```solidity
function sendCrossChainMessage(
    TeleporterMessageInput calldata messageInput
) external returns (bytes32 messageID);
```

Returns a `messageID` (bytes32) that uniquely identifies the message. Use this to track delivery.

## TeleporterMessageInput Fields

| Field | Type | Description |
|-------|------|-------------|
| `destinationBlockchainID` | `bytes32` | The blockchain ID of the destination chain (not chain ID — see below) |
| `destinationAddress` | `address` | The contract on the destination chain that will receive the message |
| `feeInfo` | `TeleporterFeeInfo` | Fee paid to the relayer (use `{feeTokenAddress: address(0), amount: 0}` for zero fee) |
| `requiredGasLimit` | `uint256` | Gas limit for the delivery transaction on the destination chain |
| `allowedRelayerAddresses` | `address[]` | Whitelist of relayer addresses. Empty array = any relayer allowed |
| `message` | `bytes` | ABI-encoded payload. Decode this in `receiveTeleporterMessage` |

## Blockchain ID vs Chain ID

- **Chain ID** (e.g., 43113): The EVM chain ID used for transaction signing
- **Blockchain ID** (bytes32): The Avalanche-level identifier for the chain, derived from the genesis hash

Get the blockchain ID:
```bash
# From AvalancheGo API
curl -X POST https://api.avax-test.network/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getBlockchains","params":{},"id":1}'
```

Convert base58 blockchain ID to bytes32:
```typescript
import { utils } from "@avalabs/avalanchejs";
const hex = "0x" + Buffer.from(utils.base58.decode("2q9e4r6Mu3U68nU1fYjgbR6JvwrRx36CohpAX5UQxse55x1Q5")).toString("hex");
```

C-Chain Fuji blockchain ID: `0x7fc93d85c6d62be589f5f8f4f0e7e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5e5`

## Complete Sender Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";

contract CrossChainSender {
    ITeleporterMessenger public constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    event MessageSent(
        bytes32 indexed messageID,
        bytes32 indexed destinationChainID,
        address destinationAddress
    );

    /// @notice Send a string message to a contract on another chain
    function sendMessage(
        bytes32 destinationChainID,
        address destinationAddress,
        string calldata message
    ) external returns (bytes32 messageID) {
        messageID = TELEPORTER.sendCrossChainMessage(
            TeleporterMessageInput({
                destinationBlockchainID: destinationChainID,
                destinationAddress: destinationAddress,
                feeInfo: TeleporterFeeInfo({
                    feeTokenAddress: address(0),
                    amount: 0
                }),
                requiredGasLimit: 100_000,
                allowedRelayerAddresses: new address[](0),
                message: abi.encode(message)
            })
        );
        emit MessageSent(messageID, destinationChainID, destinationAddress);
    }

    /// @notice Send arbitrary bytes to a contract on another chain
    function sendBytes(
        bytes32 destinationChainID,
        address destinationAddress,
        bytes calldata payload,
        uint256 gasLimit
    ) external returns (bytes32 messageID) {
        messageID = TELEPORTER.sendCrossChainMessage(
            TeleporterMessageInput({
                destinationBlockchainID: destinationChainID,
                destinationAddress: destinationAddress,
                feeInfo: TeleporterFeeInfo({
                    feeTokenAddress: address(0),
                    amount: 0
                }),
                requiredGasLimit: gasLimit,
                allowedRelayerAddresses: new address[](0),
                message: payload
            })
        );
    }
}
```

## TypeScript: Triggering a Send

```typescript
import { ethers } from "ethers";

const SENDER_ABI = [
  "function sendMessage(bytes32 destinationChainID, address destinationAddress, string message) returns (bytes32)"
];

async function sendICMMessage() {
  const provider = new ethers.JsonRpcProvider("https://api.avax-test.network/ext/bc/C/rpc");
  const signer = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  const sender = new ethers.Contract(SENDER_ADDRESS, SENDER_ABI, signer);

  const tx = await sender.sendMessage(
    DESTINATION_BLOCKCHAIN_ID,  // bytes32
    RECEIVER_ADDRESS,
    "Hello from C-Chain!"
  );
  const receipt = await tx.wait();
  console.log("Sent! TX hash:", receipt.hash);

  // Extract messageID from logs
  const iface = new ethers.Interface(SENDER_ABI);
  for (const log of receipt.logs) {
    try {
      const parsed = iface.parseLog(log);
      if (parsed?.name === "MessageSent") {
        console.log("Message ID:", parsed.args.messageID);
      }
    } catch {}
  }
}
```

## Setting `requiredGasLimit`

Set this to the gas your `receiveTeleporterMessage` function needs, plus a buffer:

```solidity
// Simple string storage: ~50,000 gas
requiredGasLimit: 100_000  // 2x buffer

// Complex logic (token minting, state updates): ~200,000 gas
requiredGasLimit: 300_000

// If too low: message delivered but execution reverts on destination
// If too high: wastes relayer gas (relayer still gets paid the fee)
```
