# Example: Cross-Chain Governance

Execute governance decisions from a hub chain on spoke chains via ICM.

## GovernanceHub (source chain — where votes happen)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";

contract GovernanceHub {
    ITeleporterMessenger constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    struct Proposal {
        bytes32 targetChain;
        address targetContract;
        bytes callData;
        uint256 voteCount;
        bool executed;
    }

    mapping(uint256 => Proposal) public proposals;
    mapping(uint256 => mapping(address => bool)) public voted;
    uint256 public proposalCount;
    uint256 public quorum;

    constructor(uint256 _quorum) { quorum = _quorum; }

    function propose(bytes32 targetChain, address targetContract, bytes calldata callData)
        external returns (uint256 id)
    {
        id = proposalCount++;
        proposals[id] = Proposal(targetChain, targetContract, callData, 0, false);
    }

    function vote(uint256 id) external {
        require(!voted[id][msg.sender], "Already voted");
        voted[id][msg.sender] = true;
        proposals[id].voteCount++;
    }

    function execute(uint256 id) external returns (bytes32 messageID) {
        Proposal storage p = proposals[id];
        require(p.voteCount >= quorum, "Quorum not reached");
        require(!p.executed, "Already executed");
        p.executed = true;

        messageID = TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
            destinationBlockchainID: p.targetChain,
            destinationAddress: p.targetContract,
            feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
            requiredGasLimit: 200_000,
            allowedRelayerAddresses: new address[](0),
            message: p.callData
        }));
    }
}
```

## GovernanceExecutor (destination chain — where actions happen)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract GovernanceExecutor is ITeleporterReceiver {
    address constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;
    bytes32 public immutable hubChain;
    address public immutable hubContract;

    // Example governed state
    uint256 public protocolFee;
    address public treasury;

    event GovernanceActionExecuted(bytes32 sourceChain, bytes data);

    constructor(bytes32 _hubChain, address _hubContract) {
        hubChain = _hubChain;
        hubContract = _hubContract;
    }

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == TELEPORTER, "Only Teleporter");
        require(sourceBlockchainID == hubChain, "Wrong hub chain");
        require(originSenderAddress == hubContract, "Wrong hub contract");

        // Decode and execute governance action
        (bytes4 selector, bytes memory args) = abi.decode(message, (bytes4, bytes));
        (bool success,) = address(this).call(abi.encodePacked(selector, args));
        require(success, "Governance action failed");

        emit GovernanceActionExecuted(sourceBlockchainID, message);
    }

    // Governed functions — only callable via ICM from hub
    function setProtocolFee(uint256 newFee) external {
        require(msg.sender == address(this), "Only via governance");
        protocolFee = newFee;
    }

    function setTreasury(address newTreasury) external {
        require(msg.sender == address(this), "Only via governance");
        treasury = newTreasury;
    }
}
```

## Usage

```typescript
// Propose: set protocol fee to 50 bps on the L1
const callData = ethers.AbiCoder.defaultAbiCoder().encode(
  ["bytes4", "bytes"],
  [
    executor.interface.getFunction("setProtocolFee").selector,
    ethers.AbiCoder.defaultAbiCoder().encode(["uint256"], [50])
  ]
);

const proposalId = await hub.propose(L1_BLOCKCHAIN_ID, executorAddress, callData);
await hub.vote(proposalId);
// ... collect more votes ...
await hub.execute(proposalId); // sends ICM message to L1
```
