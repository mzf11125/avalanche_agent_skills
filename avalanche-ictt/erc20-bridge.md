# Bridging ERC-20 Tokens with ICTT

## Setup

```bash
forge install ava-labs/avalanche-interchain-token-transfer --no-commit
```

```toml
# foundry.toml remappings
"@avalabs/ictt/=lib/avalanche-interchain-token-transfer/contracts/"
```

## Deploy ERC20TokenHome (source chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

import "@avalabs/ictt/TokenHome/ERC20TokenHome.sol";

// Deploy this on the chain where your ERC-20 token lives
// Constructor args:
//   teleporterRegistryAddress: 0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228
//   teleporterManager: your admin address
//   tokenAddress: your ERC-20 token address
//   tokenDecimals: your token's decimals (e.g., 18)
```

```bash
forge create @avalabs/ictt/TokenHome/ERC20TokenHome.sol:ERC20TokenHome \
  --constructor-args \
    0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228 \
    $ADMIN_ADDRESS \
    $TOKEN_ADDRESS \
    18 \
  --rpc-url $SOURCE_CHAIN_RPC \
  --private-key $PK
```

## Deploy ERC20TokenRemote (destination chain)

```solidity
import "@avalabs/ictt/TokenRemote/ERC20TokenRemote.sol";

// Constructor args:
//   settings: TokenRemoteSettings struct
//   tokenName: name for the wrapped token
//   tokenSymbol: symbol for the wrapped token
//   tokenDecimals: decimals (should match home token)
```

```bash
# TokenRemoteSettings: (teleporterRegistry, teleporterManager, homeBlockchainID, homeTokenTransferrerAddress, tokenDecimals)
forge create @avalabs/ictt/TokenRemote/ERC20TokenRemote.sol:ERC20TokenRemote \
  --constructor-args \
    "(0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228,$ADMIN,$SOURCE_BLOCKCHAIN_ID,$TOKEN_HOME_ADDRESS,18)" \
    "Wrapped MyToken" \
    "wMYT" \
    18 \
  --rpc-url $DEST_CHAIN_RPC \
  --private-key $PK
```

## Register Remote with Home

After deploying both contracts, register the remote:

```bash
# Call registerWithHome() on the TokenRemote — sends an ICM message to TokenHome
cast send $TOKEN_REMOTE_ADDRESS \
  "registerWithHome((address,uint256))" \
  "(address(0),0)" \
  --rpc-url $DEST_CHAIN_RPC \
  --private-key $PK
```

Wait for the relayer to deliver the registration message (~30s on Fuji).

## Send Tokens: Home → Remote

```solidity
import "@avalabs/ictt/interfaces/ITokenTransferrer.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

// 1. Approve TokenHome to spend your tokens
IERC20(tokenAddress).approve(tokenHomeAddress, amount);

// 2. Send
ITokenTransferrer(tokenHomeAddress).send(
    SendTokensInput({
        destinationBlockchainID: destBlockchainID,
        destinationTokenTransferrerAddress: tokenRemoteAddress,
        recipient: recipientAddress,
        primaryFeeTokenAddress: address(0), // no ICM fee
        primaryFee: 0,
        secondaryFee: 0,
        requiredGasLimit: 250_000,
        multiHopFallback: address(0)
    }),
    amount
);
```

## Send Tokens: Remote → Home

```solidity
// Approve TokenRemote to spend wrapped tokens
IERC20(tokenRemoteAddress).approve(tokenRemoteAddress, amount);

// Send back to home chain
ITokenTransferrer(tokenRemoteAddress).send(
    SendTokensInput({
        destinationBlockchainID: homeBlockchainID,
        destinationTokenTransferrerAddress: tokenHomeAddress,
        recipient: recipientAddress,
        primaryFeeTokenAddress: address(0),
        primaryFee: 0,
        secondaryFee: 0,
        requiredGasLimit: 250_000,
        multiHopFallback: address(0)
    }),
    amount
);
```

## TypeScript: Send ERC-20 via ICTT

```typescript
import { ethers } from "ethers";

const TOKEN_HOME_ABI = [
  "function send((bytes32,address,address,address,uint256,uint256,uint256,address) input, uint256 amount)",
];
const ERC20_ABI = ["function approve(address spender, uint256 amount) returns (bool)"];

async function bridgeTokens(amount: bigint) {
  const token = new ethers.Contract(TOKEN_ADDRESS, ERC20_ABI, signer);
  await (await token.approve(TOKEN_HOME_ADDRESS, amount)).wait();

  const home = new ethers.Contract(TOKEN_HOME_ADDRESS, TOKEN_HOME_ABI, signer);
  await (await home.send(
    [DEST_BLOCKCHAIN_ID, TOKEN_REMOTE_ADDRESS, RECIPIENT, ethers.ZeroAddress, 0n, 0n, 250000n, ethers.ZeroAddress],
    amount
  )).wait();
}
```
