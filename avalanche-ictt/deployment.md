# ICTT Deployment Guide

## Prerequisites

- Both chains must have Teleporter deployed at `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf`
- Both chains must have TeleporterRegistry deployed:
  - Fuji C-Chain: `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228`
  - Mainnet C-Chain: `0x7C43605E14F391720e1b37E49C78C4b03A488d98`
  - Custom L1s: must deploy their own registry
- A running AWM relayer configured for both chains
- For `NativeTokenRemote`: `ContractNativeMinter` precompile enabled on destination chain

## Step-by-Step: ERC-20 Bridge

### 1. Deploy TokenHome on source chain

```bash
export SOURCE_RPC="https://api.avax-test.network/ext/bc/C/rpc"
export DEST_RPC="https://your-l1-rpc/ext/bc/CHAIN/rpc"
export REGISTRY="0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228"  # Fuji; use 0x7C43605E14F391720e1b37E49C78C4b03A488d98 for Mainnet

TOKEN_HOME=$(forge create lib/avalanche-interchain-token-transfer/contracts/TokenHome/ERC20TokenHome.sol:ERC20TokenHome \
  --constructor-args $REGISTRY $ADMIN $TOKEN_ADDRESS 18 \
  --rpc-url $SOURCE_RPC --private-key $PK \
  | grep "Deployed to:" | awk '{print $3}')

echo "TokenHome: $TOKEN_HOME"
```

### 2. Deploy TokenRemote on destination chain

```bash
# Get source chain blockchain ID (bytes32)
SOURCE_BLOCKCHAIN_ID=$(cast call 0x0200000000000000000000000000000000000005 \
  "getBlockchainID()(bytes32)" --rpc-url $SOURCE_RPC)

TOKEN_REMOTE=$(forge create lib/avalanche-interchain-token-transfer/contracts/TokenRemote/ERC20TokenRemote.sol:ERC20TokenRemote \
  --constructor-args \
    "($REGISTRY,$ADMIN,$SOURCE_BLOCKCHAIN_ID,$TOKEN_HOME,18)" \
    "Wrapped Token" "wTKN" 18 \
  --rpc-url $DEST_RPC --private-key $PK \
  | grep "Deployed to:" | awk '{print $3}')

echo "TokenRemote: $TOKEN_REMOTE"
```

### 3. Register TokenRemote with TokenHome

```bash
# This sends an ICM message from TokenRemote to TokenHome
cast send $TOKEN_REMOTE \
  "registerWithHome((address,uint256))" \
  "(0x0000000000000000000000000000000000000000,0)" \
  --rpc-url $DEST_RPC --private-key $PK

echo "Waiting for relayer to deliver registration..."
sleep 30

# Verify registration
cast call $TOKEN_HOME \
  "registeredRemotes(bytes32,address)(bool,uint256,uint256,bool)" \
  $DEST_BLOCKCHAIN_ID $TOKEN_REMOTE \
  --rpc-url $SOURCE_RPC
# Should return: true, ...
```

### 4. Test a Transfer

```bash
AMOUNT="1000000000000000000"  # 1 token (18 decimals)

# Approve
cast send $TOKEN_ADDRESS \
  "approve(address,uint256)" $TOKEN_HOME $AMOUNT \
  --rpc-url $SOURCE_RPC --private-key $PK

# Send
cast send $TOKEN_HOME \
  "send((bytes32,address,address,address,uint256,uint256,uint256,address),uint256)" \
  "($DEST_BLOCKCHAIN_ID,$TOKEN_REMOTE,$RECIPIENT,0x0000000000000000000000000000000000000000,0,0,250000,0x0000000000000000000000000000000000000000)" \
  $AMOUNT \
  --rpc-url $SOURCE_RPC --private-key $PK

echo "Waiting for delivery..."
sleep 30

# Check balance on destination
cast call $TOKEN_REMOTE \
  "balanceOf(address)(uint256)" $RECIPIENT \
  --rpc-url $DEST_RPC
```

## Troubleshooting

**Registration not completing**: Check relayer is running and configured for both chains.

**Transfer stuck**: Verify `requiredGasLimit` is sufficient (250,000 is usually enough for simple transfers).

**`registeredRemotes` returns false**: Registration ICM message not yet delivered. Wait for relayer or check relayer logs.

**Minting fails on NativeTokenRemote**: Ensure the NativeTokenRemote address is enabled in the `ContractNativeMinter` precompile.
