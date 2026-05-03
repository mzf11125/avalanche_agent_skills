# Avalanche Developer FAQ

## General

**Q: Is Avalanche EVM compatible?**

Yes. The C-Chain and all Avalanche L1s running Subnet-EVM are fully EVM-compatible. Your Solidity contracts, Hardhat configs, Foundry scripts, and ethers.js/viem code work on Avalanche with only RPC URL and chain ID changes.

---

**Q: What's the difference between a Subnet and an Avalanche L1?**

They're the same thing. "Subnet" is the legacy name; "Avalanche L1" is the new name introduced with the Etna upgrade. All documentation written before late 2024 uses "Subnet." New documentation uses "Avalanche L1."

---

**Q: Do I need to stake 2,000 AVAX to run an L1 validator after Etna?**

No. Post-Etna (ACP-77), L1-only validators pay a small continuous fee in AVAX instead of staking 2,000 AVAX. The 2,000 AVAX requirement only applies to validators of the primary network (C/P/X chains).

---

**Q: How does cross-chain messaging work on Avalanche?**

Avalanche uses ICM (Interchain Messaging), built on AWM (Avalanche Warp Messaging). When a contract on Chain A sends a message, the source chain's validators sign it with BLS keys. An off-chain relayer collects the signatures and submits the aggregate signature to Chain B. Chain B's Subnet-EVM verifies the signature against the known validator set and delivers the message. No external bridge operator is needed.

---

**Q: Can I use Hardhat/Foundry with Avalanche?**

Yes. Both work out of the box. You just need to add Avalanche network configs:

```typescript
// hardhat.config.ts
networks: {
  fuji: {
    url: "https://api.avax-test.network/ext/bc/C/rpc",
    chainId: 43113,
    accounts: [process.env.PRIVATE_KEY!]
  },
  avalanche: {
    url: "https://api.avax.network/ext/bc/C/rpc",
    chainId: 43114,
    accounts: [process.env.PRIVATE_KEY!]
  }
}
```

---

**Q: What's the gas token on an Avalanche L1?**

You choose. When configuring your L1's genesis, you set the native token. It can be:
- AVAX (same as C-Chain)
- A custom token with any name/symbol
- A wrapped ERC-20 (via the NativeAssetCall precompile)

---

**Q: How do I bridge tokens between Avalanche L1s?**

Use ICTT (Interchain Token Transfer). Deploy an `ERC20TokenHome` on the source chain and an `ERC20TokenRemote` on the destination chain. Register the remote with the home, then users can bridge tokens by calling `send` on the TokenHome. See the `avalanche-ictt` skill for full details.

---

**Q: What's the TPS of Avalanche?**

- C-Chain: ~4,500 TPS sustained
- Individual Avalanche L1s: higher, since they don't share block space with other chains
- Finality: ~1–2 seconds for all chains

---

**Q: How is Avalanche different from Polygon/Arbitrum/Optimism?**

Polygon, Arbitrum, and Optimism are Ethereum L2s - they post proofs to Ethereum and inherit its security. Avalanche L1s are sovereign chains with independent validators. Key differences:
- Avalanche L1s don't inherit Ethereum security (pro: sovereignty; con: must bootstrap your own validator set)
- Avalanche L1s can have custom gas tokens; L2s use ETH
- Avalanche L1s have sub-second finality; L2s have Ethereum's finality (~12 min)
- Avalanche cross-chain messaging (ICM) is trust-minimized; most L2 bridges are not

---

**Q: What is Teleporter?**

Teleporter is the high-level SDK for ICM (Interchain Messaging). It's a set of Solidity contracts (`ITeleporterMessenger`, `TeleporterRegistry`) that provide a simple `sendCrossChainMessage` / `receiveTeleporterMessage` API. Under the hood, it uses AWM (Avalanche Warp Messaging). See the `avalanche-icm` skill.

---

**Q: What is the Teleporter contract address?**

Teleporter is deployed at a deterministic address on every Subnet-EVM chain:
- Mainnet & Fuji: `0x253b2784c75e510dD0fF1da844684a1aC0aa5fcf` (TeleporterMessenger v1.0.0)
- Always use TeleporterRegistry to get the latest version:
  - Mainnet C-Chain: `0x7C43605E14F391720e1b37E49C78C4b03A488d98`
  - Fuji C-Chain: `0xF86Cb19Ad8405AEFa7d09C778215D2Cb6eBfB228`
  - Custom L1s: must deploy their own registry

---

**Q: Can I run a non-EVM VM on an Avalanche L1?**

Yes. Avalanche supports custom VMs written in Go. However, Subnet-EVM covers the vast majority of use cases and is the recommended starting point. Custom VMs are for specialized use cases (e.g., a custom consensus mechanism, a non-EVM execution environment).

---

**Q: How do I get test AVAX on Fuji?**

Use the official faucet: `https://faucet.avax.network`. You can get 2 AVAX per day per address. For larger amounts, ask in the Avalanche Discord.

---

**Q: What's the difference between C-Chain and an Avalanche L1?**

C-Chain is the primary EVM chain of the Avalanche primary network. It uses AVAX as gas, is validated by all primary network validators, and is the most decentralized chain in the ecosystem. An Avalanche L1 is a separate chain with its own validators and gas token. L1s are more customizable but have a smaller validator set.

---

**Q: Do Avalanche L1s share security with the primary network?**

No. Each L1 has its own validator set. This is different from Ethereum L2s, which inherit Ethereum's security. The tradeoff: L1s are more sovereign and customizable, but you must bootstrap your own validator set.

---

**Q: What is AvalancheGo?**

AvalancheGo is the official Go implementation of an Avalanche node. It runs the primary network (C/P/X chains) and can run L1 VMs as plugins. It's the only production-ready Avalanche node implementation.

---

**Q: What is the AWM relayer?**

The AWM relayer is an off-chain service that monitors source chains for outgoing ICM messages, collects BLS signatures from validators, and submits the aggregate signature to destination chains. Without a running relayer, ICM messages won't be delivered. You can run your own relayer or use a third-party relayer service.

---

**Q: What precompiles does Subnet-EVM support?**

- `ContractDeployerAllowList` (`0x0200000000000000000000000000000000000000`) - restrict contract deployment
- `TxAllowList` (`0x0200000000000000000000000000000000000002`) - restrict transaction senders
- `NativeAssetBalance` (`0x0100000000000000000000000000000000000001`) - query native token balances
- `NativeAssetCall` (`0x0100000000000000000000000000000000000002`) - call contracts with native value
- `FeeManager` (`0x0200000000000000000000000000000000000003`) - dynamic fee configuration
- `WarpMessenger` (`0x0200000000000000000000000000000000000005`) - low-level Warp messaging

See the `avalanche-evm` skill for full precompile documentation.

---

**Q: Is Avalanche open source?**

Yes. AvalancheGo, Subnet-EVM, Teleporter, ICTT, and all official tooling are open source under the BSD-3-Clause or MIT license on GitHub at `github.com/ava-labs`.
