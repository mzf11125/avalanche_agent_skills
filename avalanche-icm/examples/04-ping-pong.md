# Example: Bidirectional Messaging (Ping-Pong)

Two contracts that send messages back and forth across chains.

## PingPong Contract (deploy on BOTH chains)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract PingPong is ITeleporterReceiver {
    ITeleporterMessenger constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    bytes32 public immutable partnerChain;
    address public partnerContract; // set after both are deployed

    uint256 public pingCount;
    uint256 public pongCount;

    event Pinged(bytes32 from, uint256 count);
    event Ponged(bytes32 from, uint256 count);

    constructor(bytes32 _partnerChain) {
        partnerChain = _partnerChain;
    }

    function setPartner(address _partner) external {
        require(partnerContract == address(0), "Already set");
        partnerContract = _partner;
    }

    function ping() external returns (bytes32) {
        pingCount++;
        return TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
            destinationBlockchainID: partnerChain,
            destinationAddress: partnerContract,
            feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
            requiredGasLimit: 100_000,
            allowedRelayerAddresses: new address[](0),
            message: abi.encode("ping", pingCount)
        }));
    }

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == address(TELEPORTER), "Only Teleporter");
        require(sourceBlockchainID == partnerChain, "Wrong chain");
        require(originSenderAddress == partnerContract, "Wrong partner");

        (string memory msgType, uint256 count) = abi.decode(message, (string, uint256));

        if (keccak256(bytes(msgType)) == keccak256("ping")) {
            emit Pinged(sourceBlockchainID, count);
            // Auto-pong
            pongCount++;
            TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
                destinationBlockchainID: partnerChain,
                destinationAddress: partnerContract,
                feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
                requiredGasLimit: 100_000,
                allowedRelayerAddresses: new address[](0),
                message: abi.encode("pong", pongCount)
            }));
        } else {
            emit Ponged(sourceBlockchainID, count);
        }
    }
}
```

## Setup

```bash
# Deploy on chain A
PING_A=$(forge create PingPong \
  --constructor-args $CHAIN_B_BLOCKCHAIN_ID \
  --rpc-url $CHAIN_A_RPC --private-key $PK \
  | grep "Deployed to:" | awk '{print $3}')

# Deploy on chain B
PING_B=$(forge create PingPong \
  --constructor-args $CHAIN_A_BLOCKCHAIN_ID \
  --rpc-url $CHAIN_B_RPC --private-key $PK \
  | grep "Deployed to:" | awk '{print $3}')

# Link them
cast send $PING_A "setPartner(address)" $PING_B --rpc-url $CHAIN_A_RPC --private-key $PK
cast send $PING_B "setPartner(address)" $PING_A --rpc-url $CHAIN_B_RPC --private-key $PK

# Start ping-pong
cast send $PING_A "ping()" --rpc-url $CHAIN_A_RPC --private-key $PK
```
