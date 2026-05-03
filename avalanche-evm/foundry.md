# Foundry Setup for Avalanche

## Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

## foundry.toml

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.25"

[rpc_endpoints]
fuji = "https://api.avax-test.network/ext/bc/C/rpc"
mainnet = "https://api.avax.network/ext/bc/C/rpc"
myL1 = "${L1_RPC_URL}"

[etherscan]
fuji = { key = "snowtrace", url = "https://api.routescan.io/v2/network/testnet/evm/43113/etherscan" }
mainnet = { key = "snowtrace", url = "https://api.routescan.io/v2/network/mainnet/evm/43114/etherscan" }
```

## Install Dependencies

```bash
# Teleporter contracts
forge install ava-labs/teleporter --no-commit

# OpenZeppelin
forge install OpenZeppelin/openzeppelin-contracts --no-commit

# Subnet-EVM precompile interfaces
forge install ava-labs/subnet-evm --no-commit
```

Add remappings to `foundry.toml`:

```toml
remappings = [
  "@avalabs/teleporter-contracts/=lib/teleporter/contracts/",
  "@openzeppelin/contracts/=lib/openzeppelin-contracts/contracts/",
  "@avalabs/subnet-evm-contracts/=lib/subnet-evm/precompile/",
]
```

## Deploy

```bash
# Deploy to Fuji
forge create src/MyContract.sol:MyContract \
  --rpc-url fuji \
  --private-key $PRIVATE_KEY \
  --constructor-args "arg1" 123

# Deploy to custom L1
forge create src/MyContract.sol:MyContract \
  --rpc-url $L1_RPC_URL \
  --private-key $PRIVATE_KEY
```

## Deploy Script (Solidity)

```solidity
// script/Deploy.s.sol
pragma solidity ^0.8.25;
import "forge-std/Script.sol";
import "../src/MyContract.sol";

contract DeployScript is Script {
    function run() external {
        vm.startBroadcast();
        new MyContract();
        vm.stopBroadcast();
    }
}
```

```bash
forge script script/Deploy.s.sol --rpc-url fuji --broadcast --private-key $PRIVATE_KEY
```

## Testing

```bash
forge test                    # run all tests
forge test -vvv               # verbose output
forge test --match-test testFoo  # run specific test
forge test --fork-url fuji    # fork Fuji for integration tests
```

### Testing ICM Receivers

```solidity
// test/MyReceiver.t.sol
pragma solidity ^0.8.25;
import "forge-std/Test.sol";
import "../src/MyReceiver.sol";

contract MyReceiverTest is Test {
    address constant TELEPORTER = 0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf;
    MyReceiver receiver;

    function setUp() public {
        receiver = new MyReceiver();
    }

    function testReceiveMessage() public {
        bytes32 sourceChain = bytes32(uint256(1));
        address sender = address(0xBEEF);
        bytes memory payload = abi.encode("hello");

        // Impersonate Teleporter
        vm.prank(TELEPORTER);
        receiver.receiveTeleporterMessage(sourceChain, sender, payload);

        assertEq(receiver.lastMessage(), "hello");
    }

    function testRejectsNonTeleporter() public {
        vm.expectRevert("Only Teleporter");
        receiver.receiveTeleporterMessage(bytes32(0), address(0), "");
    }
}
```

## Verify

```bash
forge verify-contract \
  --chain-id 43113 \
  --etherscan-api-key snowtrace \
  --verifier-url https://api.routescan.io/v2/network/testnet/evm/43113/etherscan \
  DEPLOYED_ADDRESS \
  src/MyContract.sol:MyContract
```

## Useful cast Commands

```bash
# Call a view function
cast call $CONTRACT "myView()(uint256)" --rpc-url fuji

# Send a transaction
cast send $CONTRACT "myFunction(uint256)" 42 --rpc-url fuji --private-key $PK

# Get account balance
cast balance $ADDRESS --rpc-url fuji

# Decode calldata
cast 4byte-decode 0xabcdef12...

# Convert units
cast to-wei 1.5 ether
cast from-wei 1500000000000000000
```
