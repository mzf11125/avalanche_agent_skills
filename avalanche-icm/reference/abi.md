# ITeleporterMessenger ABI

## Minimal ABI (sending + receiving)

```json
[
  {
    "type": "function",
    "name": "sendCrossChainMessage",
    "inputs": [
      {
        "name": "messageInput",
        "type": "tuple",
        "components": [
          { "name": "destinationBlockchainID", "type": "bytes32" },
          { "name": "destinationAddress", "type": "address" },
          {
            "name": "feeInfo",
            "type": "tuple",
            "components": [
              { "name": "feeTokenAddress", "type": "address" },
              { "name": "amount", "type": "uint256" }
            ]
          },
          { "name": "requiredGasLimit", "type": "uint256" },
          { "name": "allowedRelayerAddresses", "type": "address[]" },
          { "name": "message", "type": "bytes" }
        ]
      }
    ],
    "outputs": [{ "name": "messageID", "type": "bytes32" }],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "messageReceived",
    "inputs": [
      { "name": "sourceBlockchainID", "type": "bytes32" },
      { "name": "messageNonce", "type": "uint256" }
    ],
    "outputs": [{ "name": "", "type": "bool" }],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "retrySendCrossChainMessage",
    "inputs": [
      {
        "name": "message",
        "type": "tuple",
        "components": [
          { "name": "sourceBlockchainID", "type": "bytes32" },
          { "name": "originSenderAddress", "type": "address" },
          { "name": "destinationBlockchainID", "type": "bytes32" },
          { "name": "destinationAddress", "type": "address" },
          { "name": "nonce", "type": "uint256" },
          { "name": "requiredGasLimit", "type": "uint256" },
          { "name": "allowedRelayerAddresses", "type": "address[]" },
          { "name": "receipts", "type": "tuple[]" },
          { "name": "message", "type": "bytes" }
        ]
      }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "redeemRelayerRewards",
    "inputs": [{ "name": "feeTokenAddress", "type": "address" }],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "event",
    "name": "SendCrossChainMessage",
    "inputs": [
      { "name": "messageID", "type": "bytes32", "indexed": true },
      { "name": "destinationBlockchainID", "type": "bytes32", "indexed": true },
      { "name": "message", "type": "tuple", "indexed": false, "components": [] },
      { "name": "feeInfo", "type": "tuple", "indexed": false, "components": [
        { "name": "feeTokenAddress", "type": "address" },
        { "name": "amount", "type": "uint256" }
      ]}
    ]
  },
  {
    "type": "event",
    "name": "ReceiveCrossChainMessage",
    "inputs": [
      { "name": "messageID", "type": "bytes32", "indexed": true },
      { "name": "sourceBlockchainID", "type": "bytes32", "indexed": true },
      { "name": "deliverer", "type": "address", "indexed": false },
      { "name": "rewardRedeemer", "type": "address", "indexed": false },
      { "name": "message", "type": "tuple", "indexed": false, "components": [] }
    ]
  }
]
```

## ITeleporterReceiver ABI

```json
[
  {
    "type": "function",
    "name": "receiveTeleporterMessage",
    "inputs": [
      { "name": "sourceBlockchainID", "type": "bytes32" },
      { "name": "originSenderAddress", "type": "address" },
      { "name": "message", "type": "bytes" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  }
]
```

## TypeScript Usage

```typescript
import { ethers } from "ethers";

const TELEPORTER_ABI = [
  "function sendCrossChainMessage((bytes32 destinationBlockchainID, address destinationAddress, (address feeTokenAddress, uint256 amount) feeInfo, uint256 requiredGasLimit, address[] allowedRelayerAddresses, bytes message) messageInput) returns (bytes32 messageID)",
  "function messageReceived(bytes32 sourceBlockchainID, uint256 messageNonce) view returns (bool)",
  "event SendCrossChainMessage(bytes32 indexed messageID, bytes32 indexed destinationBlockchainID, tuple message, tuple feeInfo)"
];

const teleporter = new ethers.Contract(
  "0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf",
  TELEPORTER_ABI,
  signer
);
```
