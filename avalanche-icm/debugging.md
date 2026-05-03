# ICM Debugging Guide

## Message Stuck in Pending

**Symptom**: `sendCrossChainMessage` succeeded on source chain, but message never arrives on destination.

**Checklist:**
1. Is the relayer running? Check `docker ps` or process list
2. Is the relayer configured to watch the source chain?
3. Does the relayer have balance on the destination chain?
4. Is the destination chain RPC reachable from the relayer?
5. Check relayer logs: `docker logs awm-relayer | grep -i error`

**Check message status via AvaCloud API:**
```bash
curl "https://data-api.avax.network/v1/icm/messages/YOUR_MESSAGE_ID"
```

## Relayer Not Picking Up Messages

**Cause**: Teleporter contract address in relayer config doesn't match deployed address.

**Fix**: Verify the address in `message-contracts` matches `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf`.

**Cause**: WebSocket connection to source chain is failing.

**Fix**: Check `ws-endpoint` in relayer config. Test manually:
```bash
wscat -c wss://your-rpc/ext/bc/CHAIN/ws
```

## Destination Contract Reverting

**Symptom**: Relayer delivers the message (tx on destination chain), but `receiveTeleporterMessage` reverts.

**Cause 1**: `requiredGasLimit` too low.
**Fix**: Increase `requiredGasLimit` in `sendCrossChainMessage`. Estimate gas by calling the function locally.

**Cause 2**: Missing `require(msg.sender == TELEPORTER)` check — wait, if this is missing, the function would succeed but be callable by anyone. If it's present and wrong address, it reverts.
**Fix**: Verify `TELEPORTER` constant matches `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf`.

**Cause 3**: Logic error in `receiveTeleporterMessage`.
**Fix**: Test the function locally with the expected payload:
```typescript
await receiver.receiveTeleporterMessage(
  sourceChainID,
  senderAddress,
  encodedPayload,
  { from: TELEPORTER_ADDRESS }  // impersonate Teleporter in tests
);
```

## Wrong Blockchain ID

**Symptom**: Message sent to wrong chain, or `destinationBlockchainID` doesn't match any configured destination.

**Cause**: Confusing chain ID (EVM integer) with blockchain ID (bytes32 Avalanche identifier).

**Fix**: Get the correct blockchain ID:
```bash
curl -X POST https://api.avax-test.network/ext/bc/P \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"platform.getBlockchains","params":{},"id":1}'
```

Convert base58 to bytes32:
```typescript
import { utils } from "@avalabs/avalanchejs";
const bytes32 = "0x" + Buffer.from(utils.base58.decode(base58BlockchainID)).toString("hex");
```

## Fee Too Low

**Symptom**: Message stays pending indefinitely even though relayer is running.

**Cause**: Fee is zero or too low for public relayers to pick up.

**Fix**: Either run your own relayer (which will relay zero-fee messages) or increase the fee:
```solidity
feeInfo: TeleporterFeeInfo({
    feeTokenAddress: WAVAX_ADDRESS,
    amount: 0.01 ether  // 0.01 AVAX
})
```

## ABI Decode Failure

**Symptom**: `receiveTeleporterMessage` reverts with "abi decode error".

**Cause**: Sender and receiver are using different ABI encoding.

**Fix**: Ensure both sides use the same struct definition and `abi.encode`/`abi.decode` calls:
```solidity
// Sender
bytes memory payload = abi.encode(myStruct);

// Receiver — must use the SAME struct type
MyStruct memory decoded = abi.decode(message, (MyStruct));
```

## Useful Diagnostic Commands

```bash
# Check if a message was delivered on destination chain
cast call RECEIVER_ADDRESS "lastMessage()(string)" --rpc-url YOUR_L1_RPC

# Check Teleporter message nonce
cast call 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf \
  "messageReceived(bytes32,uint256)(bool)" \
  SOURCE_CHAIN_ID MESSAGE_NONCE \
  --rpc-url DESTINATION_RPC

# Check relayer account balance
cast balance RELAYER_ADDRESS --rpc-url DESTINATION_RPC
```
