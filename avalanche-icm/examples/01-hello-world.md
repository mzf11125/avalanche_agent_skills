# Example: Hello World Cross-Chain Message

Send a string from C-Chain to an L1, receive and store it.

## Sender (deploy on C-Chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";

contract HelloSender {
    ITeleporterMessenger constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    function send(bytes32 destChain, address destContract, string calldata greeting)
        external returns (bytes32)
    {
        return TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
            destinationBlockchainID: destChain,
            destinationAddress: destContract,
            feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
            requiredGasLimit: 100_000,
            allowedRelayerAddresses: new address[](0),
            message: abi.encode(greeting)
        }));
    }
}
```

## Receiver (deploy on destination L1)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract HelloReceiver is ITeleporterReceiver {
    address constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;

    string public lastGreeting;

    event Greeted(bytes32 sourceChain, address sender, string greeting);

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == TELEPORTER, "Only Teleporter");
        lastGreeting = abi.decode(message, (string));
        emit Greeted(sourceBlockchainID, originSenderAddress, lastGreeting);
    }
}
```

## Deploy & Test

```bash
# Deploy receiver on L1
forge create HelloReceiver --rpc-url $L1_RPC --private-key $PK

# Deploy sender on C-Chain
forge create HelloSender --rpc-url https://api.avax-test.network/ext/bc/C/rpc --private-key $PK

# Send message
cast send $SENDER_ADDRESS \
  "send(bytes32,address,string)" \
  $L1_BLOCKCHAIN_ID $RECEIVER_ADDRESS "Hello from C-Chain!" \
  --rpc-url https://api.avax-test.network/ext/bc/C/rpc \
  --private-key $PK

# After relayer delivers (~30s), check receiver
cast call $RECEIVER_ADDRESS "lastGreeting()(string)" --rpc-url $L1_RPC
```
