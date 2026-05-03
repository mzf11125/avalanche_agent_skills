# Hardhat Setup for Avalanche

## Install

```bash
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat init
```

## hardhat.config.ts

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: "0.8.25",
  networks: {
    // Fuji C-Chain
    fuji: {
      url: "https://api.avax-test.network/ext/bc/C/rpc",
      chainId: 43113,
      accounts: [process.env.PRIVATE_KEY!],
    },
    // Mainnet C-Chain
    mainnet: {
      url: "https://api.avax.network/ext/bc/C/rpc",
      chainId: 43114,
      accounts: [process.env.PRIVATE_KEY!],
    },
    // Your custom L1
    myL1: {
      url: process.env.L1_RPC_URL!,
      chainId: parseInt(process.env.L1_CHAIN_ID!),
      accounts: [process.env.PRIVATE_KEY!],
    },
  },
};

export default config;
```

## Deploy Script

```typescript
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with:", deployer.address);

  const MyContract = await ethers.getContractFactory("MyContract");
  const contract = await MyContract.deploy(/* constructor args */);
  await contract.waitForDeployment();

  console.log("Deployed to:", await contract.getAddress());
}

main().catch(console.error);
```

```bash
npx hardhat run scripts/deploy.ts --network fuji
npx hardhat run scripts/deploy.ts --network myL1
```

## Verify on Snowtrace (C-Chain)

```bash
npm install --save-dev @nomicfoundation/hardhat-verify

# hardhat.config.ts — add etherscan config:
etherscan: {
  apiKey: {
    avalancheFujiTestnet: "snowtrace",  // no key needed
    avalanche: "snowtrace",
  },
  customChains: [
    {
      network: "avalancheFujiTestnet",
      chainId: 43113,
      urls: {
        apiURL: "https://api.routescan.io/v2/network/testnet/evm/43113/etherscan",
        browserURL: "https://testnet.snowtrace.io",
      },
    },
  ],
},

# Verify
npx hardhat verify --network fuji DEPLOYED_ADDRESS "constructor arg 1"
```

## Testing with Hardhat

```typescript
import { expect } from "chai";
import { ethers } from "hardhat";

describe("MyContract", () => {
  it("should work", async () => {
    const [owner] = await ethers.getSigners();
    const MyContract = await ethers.getContractFactory("MyContract");
    const contract = await MyContract.deploy();

    // Test against local hardhat network (no Avalanche-specific features)
    // For precompile testing, use a local Avalanche network
    expect(await contract.someView()).to.equal(expectedValue);
  });
});
```

## Impersonating Teleporter in Tests

```typescript
// Impersonate the Teleporter contract to test receiveTeleporterMessage
await network.provider.request({
  method: "hardhat_impersonateAccount",
  params: ["0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf"],
});
const teleporter = await ethers.getSigner("0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf");
await receiver.connect(teleporter).receiveTeleporterMessage(
  sourceChainID, senderAddress, encodedPayload
);
```
