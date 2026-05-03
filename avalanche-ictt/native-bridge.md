# Bridging Native Tokens with ICTT

## Use Cases

- Bridge AVAX from C-Chain to an L1 (where it becomes an ERC-20 or native token)
- Bridge your L1's native gas token to C-Chain or another L1
- Create a "wrapped native" token on a remote chain

## NativeTokenHome (source chain)

Locks native tokens (e.g., AVAX) sent to it and triggers a mint on the remote chain.

```bash
forge create @avalabs/ictt/TokenHome/NativeTokenHome.sol:NativeTokenHome \
  --constructor-args \
    0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228 \
    $ADMIN_ADDRESS \
    $WRAPPED_NATIVE_TOKEN_ADDRESS \
  --rpc-url $SOURCE_CHAIN_RPC \
  --private-key $PK
```

`WRAPPED_NATIVE_TOKEN_ADDRESS`: the WAVAX/WETH address on the source chain. On Fuji C-Chain: `0xd00ae08403B9bbb9124bB305C09058E32C39A48c`.

## NativeTokenRemote (destination chain)

Receives bridged native tokens and makes them the native gas token of the destination chain. Requires the `ContractNativeMinter` precompile to be enabled and the `NativeTokenRemote` address to be an enabled minter.

```bash
forge create @avalabs/ictt/TokenRemote/NativeTokenRemote.sol:NativeTokenRemote \
  --constructor-args \
    "(0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228,$ADMIN,$SOURCE_BLOCKCHAIN_ID,$NATIVE_TOKEN_HOME_ADDRESS,18)" \
    "MyToken" \
    "MTK" \
    1000000000000000000000000 \
    false \
  --rpc-url $DEST_CHAIN_RPC \
  --private-key $PK
```

Constructor args:
- `TokenRemoteSettings` struct
- `nativeAssetSymbol`: symbol for the native token
- `initialReserveImbalance`: initial supply already in circulation (usually 0 for new chains)
- `burnedFeesReportingRewardPercentage`: set to `false` to disable

## Enable Minting on Destination Chain

```bash
# Grant NativeTokenRemote permission to mint native tokens
cast send 0x0200000000000000000000000000000000000001 \
  "setEnabled(address)" \
  $NATIVE_TOKEN_REMOTE_ADDRESS \
  --rpc-url $DEST_CHAIN_RPC \
  --private-key $ADMIN_PK
```

## Register and Send

```bash
# Register remote with home (same as ERC-20 flow)
cast send $NATIVE_TOKEN_REMOTE_ADDRESS \
  "registerWithHome((address,uint256))" \
  "(address(0),0)" \
  --rpc-url $DEST_CHAIN_RPC \
  --private-key $PK

# Send native tokens from home to remote (send ETH/AVAX with the call)
cast send $NATIVE_TOKEN_HOME_ADDRESS \
  "send((bytes32,address,address,address,uint256,uint256,uint256,address))" \
  "($DEST_BLOCKCHAIN_ID,$NATIVE_TOKEN_REMOTE_ADDRESS,$RECIPIENT,address(0),0,0,300000,address(0))" \
  --value 1ether \
  --rpc-url $SOURCE_CHAIN_RPC \
  --private-key $PK
```

## ERC20TokenRemote as Wrapped Native

If you want bridged native tokens to appear as an ERC-20 (not as the gas token) on the destination chain, use `ERC20TokenRemote` instead of `NativeTokenRemote`. This is simpler — no minter precompile setup needed.
