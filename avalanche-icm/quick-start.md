# ICM Quick Start: Send Your First Cross-Chain Message

Send a message from Fuji C-Chain to a custom L1 in ~15 minutes.

## Prerequisites

- Node.js 18+, npm
- A funded Fuji wallet (get AVAX at `https://faucet.avax.network`)
- Your L1's RPC URL and chain ID
- Hardhat installed

## Step 1: Install Dependencies

```bash
mkdir icm-demo && cd icm-demo
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npm install @avalabs/teleporter-contracts ethers
npx hardhat init  # choose TypeScript project
```

## Step 2: Configure Hardhat

```typescript
// hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: "0.8.25",
  networks: {
    fuji: {
      url: "https://api.avax-test.network/ext/bc/C/rpc",
      chainId: 43113,
      accounts: [process.env.PRIVATE_KEY!],
    },
    myL1: {
      url: process.env.L1_RPC_URL!,
      chainId: parseInt(process.env.L1_CHAIN_ID!),
      accounts: [process.env.PRIVATE_KEY!],
    },
  },
};
export default config;
```

```bash
# .env
PRIVATE_KEY=0x...
L1_RPC_URL=https://your-l1-rpc.example.com/ext/bc/YOUR_CHAIN_ID/rpc
L1_CHAIN_ID=12345
```

## Step 3: Teleporter Contract Address

Teleporter is pre-deployed on every Subnet-EVM chain at:
```
0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf
```

Use TeleporterRegistry to always get the latest version:
- Mainnet C-Chain: `0x7C43605E14F391720e1b37E49C78C4b03A488d98`
- Fuji C-Chain: `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228`
- Custom L1s: deploy your own registry

## Step 4: Write the Sender Contract

```solidity
// contracts/Sender.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract Sender {
    ITeleporterMessenger public constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    event MessageSent(bytes32 indexed messageID, bytes32 destinationChainID);

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
        emit MessageSent(messageID, destinationChainID);
    }
}
```

## Step 5: Write the Receiver Contract

```solidity
// contracts/Receiver.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract Receiver is ITeleporterReceiver {
    address public constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;

    string public lastMessage;
    address public lastSender;

    event MessageReceived(bytes32 indexed sourceChainID, address indexed sender, string message);

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == TELEPORTER, "Only Teleporter can call this");
        string memory decoded = abi.decode(message, (string));
        lastMessage = decoded;
        lastSender = originSenderAddress;
        emit MessageReceived(sourceBlockchainID, originSenderAddress, decoded);
    }
}
```

## Step 6: Deploy Both Contracts

```bash
# Deploy Receiver on your L1 first
npx hardhat run scripts/deploy-receiver.ts --network myL1

# Deploy Sender on Fuji C-Chain
npx hardhat run scripts/deploy-sender.ts --network fuji
```

```typescript
// scripts/deploy-receiver.ts
import { ethers } from "hardhat";
async function main() {
  const Receiver = await ethers.getContractFactory("Receiver");
  const receiver = await Receiver.deploy();
  await receiver.waitForDeployment();
  console.log("Receiver deployed to:", await receiver.getAddress());
}
main();
```

## Step 7: Send the Message

```typescript
// scripts/send-message.ts
import { ethers } from "hardhat";

const SENDER_ADDRESS = "0x...";   // deployed on Fuji
const RECEIVER_ADDRESS = "0x..."; // deployed on your L1
const DESTINATION_CHAIN_ID = "0x..."; // your L1's blockchain ID (bytes32)

async function main() {
  const sender = await ethers.getContractAt("Sender", SENDER_ADDRESS);
  const tx = await sender.sendMessage(
    DESTINATION_CHAIN_ID,
    RECEIVER_ADDRESS,
    "Hello from C-Chain!"
  );
  const receipt = await tx.wait();
  console.log("Message sent! TX:", receipt.hash);
}
main();
```

```bash
npx hardhat run scripts/send-message.ts --network fuji
```

## Step 8: Verify Delivery

```typescript
// scripts/check-delivery.ts
import { ethers } from "hardhat";

const RECEIVER_ADDRESS = "0x...";

async function main() {
  const receiver = await ethers.getContractAt("Receiver", RECEIVER_ADDRESS);
  // Poll until message arrives (relayer delivers within ~30 seconds)
  for (let i = 0; i < 20; i++) {
    const msg = await receiver.lastMessage();
    if (msg) {
      console.log("Message received:", msg);
      return;
    }
    console.log("Waiting for delivery...");
    await new Promise(r => setTimeout(r, 5000));
  }
  console.log("Timeout - check relayer logs");
}
main();
```

```bash
npx hardhat run scripts/check-delivery.ts --network myL1
```

## Getting Your L1's Blockchain ID

The `destinationChainID` is the blockchain ID in bytes32 format. Get it from:

```bash
# AvalancheGo API
curl -X POST https://api.avax-test.network/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getBlockchains","params":{},"id":1}'
```

Or convert from the base58 blockchain ID:
```typescript
import { utils } from "@avalabs/avalanchejs";
const blockchainIDHex = "0x" + Buffer.from(utils.base58.decode("YOUR_CHAIN_ID")).toString("hex");
```
