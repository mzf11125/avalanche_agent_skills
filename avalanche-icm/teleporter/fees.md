# ICM Fee Mechanics

## Overview

ICM fees compensate relayers for the gas cost of delivering messages on the destination chain. Fees are optional — you can send zero-fee messages, but they may not be picked up by public relayers.

## Zero-Fee Messages (Testing / Private Relayer)

```solidity
feeInfo: TeleporterFeeInfo({
    feeTokenAddress: address(0),
    amount: 0
})
```

Use zero fees when:
- Testing on Fuji with your own relayer
- Running a private L1 with a dedicated relayer you control
- The relayer is configured to relay without fees

## Paying Fees with an ERC-20 Token

```solidity
// 1. Approve Teleporter to spend your fee token
IERC20(feeToken).approve(TELEPORTER_ADDRESS, feeAmount);

// 2. Send with fee
TELEPORTER.sendCrossChainMessage(
    TeleporterMessageInput({
        destinationBlockchainID: destChain,
        destinationAddress: destAddr,
        feeInfo: TeleporterFeeInfo({
            feeTokenAddress: feeToken,   // ERC-20 address
            amount: feeAmount            // Amount in token's decimals
        }),
        requiredGasLimit: 100_000,
        allowedRelayerAddresses: new address[](0),
        message: payload
    })
);
```

The fee token is transferred from `msg.sender` to the Teleporter contract, then paid to the relayer upon delivery.

## Fee Token on C-Chain

On C-Chain, WAVAX (Wrapped AVAX) is the standard fee token:
- Mainnet: `0xB31f66AA3C1e785363F0875A1B74E27b85FD66c7`
- Fuji: `0xd00ae08403B9bbb9124bB305C09058E32C39A48c`

## Estimating the Right Fee Amount

The fee should cover the relayer's gas cost on the destination chain:

```
fee ≈ requiredGasLimit × destinationGasPrice × feeTokenPrice
```

Example: 100,000 gas × 25 nAVAX/gas × AVAX price = fee in AVAX

In practice, start with a small fee (e.g., 0.01 AVAX equivalent) and increase if messages aren't being picked up.

## Restricting Relayers

```solidity
// Only allow a specific relayer address
allowedRelayerAddresses: [0xYourRelayerAddress]

// Allow any relayer (recommended for production with fees)
allowedRelayerAddresses: new address[](0)
```

## Fee Redemption

Relayers call `redeemRelayerRewards(address feeTokenAddress)` on the Teleporter contract to claim accumulated fees.
