# Avalanche Developer Glossary

80+ terms for Avalanche L1 developers, alphabetically sorted.

---

**ACP (Avalanche Community Proposal)** - A formal proposal to change the Avalanche protocol, similar to Ethereum's EIPs. ACP-77 introduced the Etna upgrade.

**ACP-77** - The community proposal that defined the Etna upgrade. Removed the requirement for L1 validators to stake 2,000 AVAX on the primary network, replacing it with a continuous fee model.

**AllowList** - A Subnet-EVM precompile that restricts who can deploy contracts or send transactions on an L1. Roles: Admin, Manager, Enabled. Address: `0x0200000000000000000000000000000000000000` (ContractDeployerAllowList).

**AVAX** - The native token of the Avalanche network. Used as gas on C-Chain, staked by primary network validators, and paid as continuous fees by L1 validators post-Etna.

**AvaCloud** - Avalanche's managed cloud platform for deploying and operating L1s without managing infrastructure. Provides APIs, monitoring, and a web console.

**AvalancheGo** - The official Go implementation of an Avalanche node. Runs the primary network and can run L1 VMs as plugins.

**AWM (Avalanche Warp Messaging)** - The low-level cross-chain messaging protocol built into Subnet-EVM. Validators sign outgoing messages with BLS keys; receiving chains verify the aggregate signature. Teleporter is built on top of AWM.

**BLS (Boneh–Lynn–Shacham)** - The signature scheme used by Avalanche validators to sign Warp messages. BLS signatures are aggregatable, meaning N validator signatures can be combined into one compact signature.

**Bootstrapping** - The process a new Avalanche node goes through to sync the blockchain state from peers before it can participate in consensus. Can take hours to days depending on chain history.

**C-Chain (Contract Chain)** - The EVM-compatible chain in the Avalanche primary network. Runs Solidity smart contracts. Chain ID: 43114 (mainnet), 43113 (Fuji). Gas token: AVAX.

**Chain ID** - A unique integer identifying a blockchain for EVM transaction signing (EIP-155). C-Chain mainnet: 43114. Fuji: 43113. Each Avalanche L1 has its own chain ID.

**Collateral** - In ICTT, the amount of tokens locked in a TokenHome contract to back the supply of tokens minted on remote chains. Collateral must be fully funded before transfers can proceed.

**Continuous Fee** - Post-Etna fee model for L1 validators. Instead of staking 2,000 AVAX, validators pay an ongoing fee proportional to the number of active validators on the L1.

**ContractDeployerAllowList** - A Subnet-EVM precompile that restricts which addresses can deploy smart contracts on the L1. Precompile address: `0x0200000000000000000000000000000000000000`.

**Delegator** - A token holder who delegates their AVAX stake to a primary network validator to earn a share of staking rewards without running a node.

**Etna** - The Avalanche network upgrade (late 2024) that implemented ACP-77, dramatically reducing L1 deployment costs and introducing the continuous fee model for L1 validators.

**EVM (Ethereum Virtual Machine)** - The execution environment for Solidity smart contracts. Avalanche's C-Chain and L1s running Subnet-EVM are EVM-compatible.

**FeeManager** - A Subnet-EVM precompile that allows authorized addresses to dynamically change the gas fee configuration of an L1. Address: `0x0200000000000000000000000000000000000003`.

**Finality** - The point at which a transaction is irreversible. Avalanche achieves finality in ~1–2 seconds. Ethereum achieves economic finality in ~12 minutes.

**Fuji** - The Avalanche public testnet. Chain IDs: C-Chain 43113, P-Chain/X-Chain use "fuji" network. Faucet: `https://faucet.avax.network`.

**Gas** - The unit measuring computational work in EVM transactions. On C-Chain, gas is paid in AVAX. On Avalanche L1s, gas is paid in the L1's configured gas token.

**Genesis** - The first block of a blockchain, containing the initial state: token allocations, precompile configurations, gas parameters, and chain metadata. Defined in `genesis.json` for Subnet-EVM chains.

**Glacier API** - AvaCloud's indexed data API (also called the Data API). Provides REST endpoints for querying transactions, balances, ICM messages, and more. Base URL: `https://data-api.avax.network/v1`.

**ICM (Interchain Messaging)** - Avalanche's native cross-chain communication protocol. Built on AWM. The high-level SDK is Teleporter. Allows smart contracts on one L1 to send messages to contracts on another L1.

**ICTT (Interchain Token Transfer)** - Avalanche's native cross-chain token bridge protocol. Uses ICM under the hood. Tokens are locked in a TokenHome contract and minted by TokenRemote contracts on destination chains.

**ITeleporterMessenger** - The Solidity interface for the Teleporter contract. Key function: `sendCrossChainMessage`. Deployed at a deterministic address on every Subnet-EVM chain.

**ITeleporterReceiver** - The Solidity interface that receiving contracts must implement. Key function: `receiveTeleporterMessage(bytes32 sourceBlockchainID, address originSenderAddress, bytes calldata message)`.

**L1 (Avalanche L1)** - A sovereign blockchain in the Avalanche ecosystem. Formerly called a "Subnet." Has its own validators, VM, and gas token. Communicates with other L1s via ICM.

**L1 Validator** - A node that validates an Avalanche L1. Post-Etna, does not need to validate the primary network. Managed by a Validator Manager contract.

**Mainnet** - The live Avalanche production network. C-Chain chain ID: 43114.

**MCP (Model Context Protocol)** - An open standard by Anthropic for connecting AI agents to external tools and data sources. The `avalanche-mcp-server` implements MCP to give AI agents live Avalanche network access.

**NativeAssetBalance** - A Subnet-EVM precompile that allows contracts to query the native token balance of any address. Address: `0x0100000000000000000000000000000000000001`.

**NativeAssetCall** - A Subnet-EVM precompile that allows contracts to call other contracts with native token value. Address: `0x0100000000000000000000000000000000000002`.

**NativeTokenHome** - An ICTT contract deployed on the source chain that locks native tokens (e.g., AVAX) and initiates cross-chain transfers.

**NativeTokenRemote** - An ICTT contract deployed on the destination chain that mints a wrapped version of the native token when a transfer arrives.

**Node ID** - A unique identifier for an Avalanche node, derived from its TLS certificate. Format: `NodeID-<base58>`. Example: `NodeID-5mb46qkSBj81k9g9e1af1uAGbFjGcr1LL`.

**P-Chain (Platform Chain)** - The Avalanche primary network chain responsible for validator coordination, staking, and L1 management. Not EVM-compatible. API: `https://api.avax.network/ext/bc/P`.

**PoA Validator Manager** - A smart contract that manages L1 validators in a permissioned (Proof of Authority) model. An admin controls who can be a validator.

**PoS Validator Manager** - A smart contract that manages L1 validators in a permissionless (Proof of Stake) model. Validators stake the L1's native token.

**Precompile** - A smart contract built into the Subnet-EVM at a fixed address that provides functionality not available in standard EVM (e.g., native token minting, fee management, Warp messaging). Enabled/configured in genesis.

**Primary Network** - The three built-in chains of Avalanche: C-Chain, P-Chain, and X-Chain. Every Avalanche node validates the primary network (or pays a fee to opt out post-Etna).

**Relayer** - An off-chain service that monitors source chains for outgoing ICM messages, collects validator signatures, and submits them to destination chains. The AWM relayer is the official implementation.

**Retro9000** - Avalanche Foundation's retroactive funding program for L1 and infrastructure builders. Snapshot date: July 14, 2026.

**RPC (Remote Procedure Call)** - The JSON-RPC API used to interact with Avalanche nodes. C-Chain RPC is Ethereum-compatible (`eth_*` methods). P-Chain has its own API.

**Snowball** - The base Avalanche consensus protocol. A node repeatedly samples random peers and adopts the majority preference, incrementing a confidence counter until a threshold is reached.

**Snowflake** - An extension of Snowball with a conviction counter that resets on preference flips, making it more resistant to adversarial manipulation.

**Snowman** - The linear-chain variant of Snowflake used for C-Chain and P-Chain. Produces a totally ordered sequence of blocks.

**Snowtrace** - The Avalanche block explorer. URL: `https://snowtrace.io` (mainnet), `https://testnet.snowtrace.io` (Fuji).

**Staking** - Locking AVAX on the P-Chain to become a primary network validator or delegator. Minimum: 2,000 AVAX for validators, 25 AVAX for delegators.

**Subnet** - The legacy name for what is now called an Avalanche L1. Still used in older documentation and tooling.

**Subnet-EVM** - The official EVM implementation for Avalanche L1s. A fork of go-ethereum with Avalanche-specific additions: precompiles, Warp messaging, configurable gas, and genesis customization.

**Teleporter** - The high-level ICM SDK. A set of smart contracts (ITeleporterMessenger, TeleporterRegistry) that provide a developer-friendly API for cross-chain messaging on top of AWM.

**TeleporterRegistry** - A contract that tracks the latest Teleporter contract address, allowing dApps to always use the current version without hardcoding addresses.

**TokenHome** - An ICTT contract on the source chain that locks tokens and initiates cross-chain transfers. Variants: `ERC20TokenHome`, `NativeTokenHome`.

**TokenRemote** - An ICTT contract on the destination chain that mints tokens when a transfer arrives. Variants: `ERC20TokenRemote`, `NativeTokenRemote`.

**TPS (Transactions Per Second)** - Throughput metric. Avalanche C-Chain: ~4,500 TPS. Individual L1s can achieve higher TPS since they don't share block space.

**TxAllowList** - A Subnet-EVM precompile that restricts which addresses can send transactions on the L1. Address: `0x0200000000000000000000000000000000000002`.

**Uptime** - The percentage of time a validator is online and participating in consensus. Primary network validators need ≥80% uptime to earn staking rewards.

**Validator** - A node that participates in Avalanche consensus. Primary network validators stake AVAX; L1-only validators (post-Etna) pay continuous fees.

**Validator Manager** - A smart contract (PoA or PoS variant) that manages the validator set of an Avalanche L1 post-Etna.

**VM (Virtual Machine)** - The execution environment for a blockchain. Avalanche supports pluggable VMs. Subnet-EVM is the standard EVM VM. Custom VMs can be written in Go.

**Warp Message** - A cross-chain message signed by validators using BLS keys. The low-level primitive underlying ICM/Teleporter.

**WarpMessenger** - A Subnet-EVM precompile at address `0x0200000000000000000000000000000000000005` that provides low-level access to send and receive Warp messages.

**X-Chain (Exchange Chain)** - The UTXO-based asset exchange chain in the Avalanche primary network. Used for fast AVAX transfers. Not EVM-compatible.
