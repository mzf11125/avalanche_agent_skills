# Example: Cross-Chain NFT Mint

Trigger an NFT mint on a destination chain by sending an ICM message from the source chain.

## NFT Contract (destination chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterReceiver.sol";

contract CrossChainNFT is ERC721, ITeleporterReceiver {
    address constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;
    bytes32 public immutable authorizedSourceChain;
    address public immutable authorizedMinter;

    uint256 private _nextTokenId;

    struct MintRequest {
        address recipient;
        uint256 tokenId; // 0 = auto-assign
    }

    constructor(bytes32 sourceChain, address minterContract)
        ERC721("CrossChainNFT", "CCNFT")
    {
        authorizedSourceChain = sourceChain;
        authorizedMinter = minterContract;
    }

    function receiveTeleporterMessage(
        bytes32 sourceBlockchainID,
        address originSenderAddress,
        bytes calldata message
    ) external override {
        require(msg.sender == TELEPORTER, "Only Teleporter");
        require(sourceBlockchainID == authorizedSourceChain, "Wrong chain");
        require(originSenderAddress == authorizedMinter, "Wrong minter");

        MintRequest memory req = abi.decode(message, (MintRequest));
        uint256 tokenId = req.tokenId == 0 ? _nextTokenId++ : req.tokenId;
        _mint(req.recipient, tokenId);
    }
}
```

## Minter Contract (source chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/teleporter-contracts/contracts/teleporter/ITeleporterMessenger.sol";

contract NFTMinter {
    ITeleporterMessenger constant TELEPORTER =
        ITeleporterMessenger(0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf);

    bytes32 public immutable destChain;
    address public immutable destNFT;

    struct MintRequest {
        address recipient;
        uint256 tokenId;
    }

    constructor(bytes32 _destChain, address _destNFT) {
        destChain = _destChain;
        destNFT = _destNFT;
    }

    /// @notice Pay on this chain, mint NFT on destination chain
    function mintOnRemoteChain(address recipient) external payable returns (bytes32) {
        require(msg.value >= 0.01 ether, "Insufficient payment");
        return TELEPORTER.sendCrossChainMessage(TeleporterMessageInput({
            destinationBlockchainID: destChain,
            destinationAddress: destNFT,
            feeInfo: TeleporterFeeInfo({ feeTokenAddress: address(0), amount: 0 }),
            requiredGasLimit: 150_000,
            allowedRelayerAddresses: new address[](0),
            message: abi.encode(MintRequest({ recipient: recipient, tokenId: 0 }))
        }));
    }
}
```
