# ICM Security Considerations

## 1. Always Validate `msg.sender`

The most critical check: only the Teleporter contract should call `receiveTeleporterMessage`.

```solidity
function receiveTeleporterMessage(
    bytes32 sourceBlockchainID,
    address originSenderAddress,
    bytes calldata message
) external override {
    require(msg.sender == TELEPORTER_ADDRESS, "Only Teleporter");
    // ...
}
```

Without this, anyone can call your function with arbitrary data.

If using `TeleporterRegistryApp`, the base class handles this — but you must still call `super` or use `_receiveTeleporterMessage`.

## 2. Validate Source Chain and Sender

Don't trust messages from unknown chains or senders:

```solidity
bytes32 constant TRUSTED_SOURCE_CHAIN = 0x...; // C-Chain blockchain ID
address constant TRUSTED_SENDER = 0x...;        // Your sender contract

function receiveTeleporterMessage(
    bytes32 sourceBlockchainID,
    address originSenderAddress,
    bytes calldata message
) external override {
    require(msg.sender == TELEPORTER_ADDRESS, "Only Teleporter");
    require(sourceBlockchainID == TRUSTED_SOURCE_CHAIN, "Wrong source chain");
    require(originSenderAddress == TRUSTED_SENDER, "Wrong sender");
    // ...
}
```

## 3. Replay Protection

Teleporter assigns each message a unique nonce. The Teleporter contract itself prevents replay — you don't need to implement this yourself. Each `messageID` can only be delivered once.

## 4. Reentrancy

`receiveTeleporterMessage` can be called by the Teleporter contract during a transaction. If your function calls external contracts, use the checks-effects-interactions pattern or a reentrancy guard:

```solidity
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract SafeReceiver is ITeleporterReceiver, ReentrancyGuard {
    function receiveTeleporterMessage(...) external override nonReentrant {
        require(msg.sender == TELEPORTER_ADDRESS, "Only Teleporter");
        // ...
    }
}
```

## 5. Don't Revert Unnecessarily

If `receiveTeleporterMessage` reverts, the message is marked failed and must be retried. For non-critical errors, emit an event instead of reverting:

```solidity
function receiveTeleporterMessage(...) external override {
    require(msg.sender == TELEPORTER_ADDRESS, "Only Teleporter");
    try this._process(message) {
        // ok
    } catch (bytes memory err) {
        emit ProcessingFailed(message, err); // don't revert
    }
}
```

## 6. ABI Encoding Consistency

Ensure sender and receiver use identical struct definitions. A mismatch causes silent data corruption or a revert on decode.

## 7. Fee Token Approval

When paying fees, only approve the exact amount needed — not `type(uint256).max`:

```solidity
IERC20(feeToken).approve(TELEPORTER_ADDRESS, feeAmount); // exact amount
```

## 8. Relayer Private Key Security

The relayer account private key has access to funds on destination chains. Use AWS KMS or a hardware wallet for production:

```json
{
  "kms-key-id": "arn:aws:kms:us-east-1:123456789:key/...",
  "kms-aws-region": "us-east-1"
}
```

Never commit private keys to source control.
