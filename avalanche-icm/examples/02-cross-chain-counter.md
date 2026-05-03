# Example: Cross-Chain Counter

Increment a counter on a remote chain via ICM.

## Counter Contract (destination chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract CrossChainCounter is ITeleporterReceiver {
    address constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;

    mapping(bytes32 => uint256) public counters; // per source chain
    bytes32 public authorizedSourceChain;
    address public authorizedSender;

    constructor(bytes32 _sourceChain, address _sender) {
        authorizedSourceChain = _sourceChain;
        authorizedSender = _sender;
    }

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == TELEPORTER, "Only Teleporter");
        require(sourceBlockchainID == authorizedSourceChain, "Wrong chain");
        require(originSenderAddress == authorizedSender, "Wrong sender");

        uint256 incrementBy = abi.decode(message, (uint256));
        counters[sourceBlockchainID] += incrementBy;
    }
}
```

## Incrementer Contract (source chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";

contract CounterIncrementer {
    ITeleporterMessenger constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    bytes32 public immutable destChain;
    address public immutable destCounter;

    constructor(bytes32 _destChain, address _destCounter) {
        destChain = _destChain;
        destCounter = _destCounter;
    }

    function increment(uint256 amount) external returns (bytes32) {
        return TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
            destinationBlockchainID: destChain,
            destinationAddress: destCounter,
            feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
            requiredGasLimit: 80_000,
            allowedRelayerAddresses: new address[](0),
            message: abi.encode(amount)
        }));
    }
}
```
