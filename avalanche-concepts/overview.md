# Avalanche Architecture Overview

## Avalanche Consensus

Avalanche uses a novel family of consensus protocols — Snowball, Snowflake, and Snowman — that achieve sub-second probabilistic finality through repeated random sampling rather than leader-based block production.

### How It Works

1. **Snowball** — A binary decision protocol. A node repeatedly samples a small random subset of validators (k=20 by default) and adopts the majority preference. A "confidence counter" increments on consecutive same-preference rounds; once it exceeds a threshold β, the decision is finalized.

2. **Snowflake** — Extends Snowball with a "conviction" mechanism. The counter resets if the preference ever flips, making the protocol more resistant to adversarial flipping attacks.

3. **Snowman** — The linear-chain variant of Snowflake used for the C-Chain and P-Chain. Blocks are totally ordered; each block references its parent. Validators run Snowman to agree on the canonical chain tip.

### Key Properties

- **Sub-second finality**: Transactions finalize in ~1–2 seconds under normal conditions
- **High throughput**: C-Chain sustains 4,500+ TPS
- **Low energy**: No proof-of-work; validators are lightweight
- **Leaderless**: No single block proposer — all validators participate simultaneously
- **Sybil resistance**: Stake-weighted sampling (more stake = sampled more often)

## The Three-Chain Primary Network

Every Avalanche node validates the **Primary Network**, which consists of three built-in blockchains:

### C-Chain (Contract Chain)

- **VM**: Ethereum Virtual Machine (EVM) — fully compatible with Solidity, ethers.js, Hardhat, Foundry
- **Purpose**: Smart contract execution, DeFi, dApps
- **Gas token**: AVAX
- **Consensus**: Snowman (linear chain)
- **RPC**: `https://api.avax.network/ext/bc/C/rpc` (mainnet), `https://api.avax-test.network/ext/bc/C/rpc` (Fuji)
- **Chain ID**: 43114 (mainnet), 43113 (Fuji)

### P-Chain (Platform Chain)

- **VM**: Platform VM (custom, not EVM)
- **Purpose**: Validator coordination, staking, L1 creation and management
- **Consensus**: Snowman
- **Key operations**: Add validator, add delegator, create L1, add L1 validator
- **API**: `https://api.avax.network/ext/bc/P`

### X-Chain (Exchange Chain)

- **VM**: Avalanche VM (UTXO-based, like Bitcoin)
- **Purpose**: Fast, low-fee asset transfers (AVAX and custom assets)
- **Consensus**: DAG-based Snowball (not linear)
- **Note**: Less relevant for smart contract developers; most activity is on C-Chain

## Avalanche L1s (formerly Subnets)

An **Avalanche L1** is a sovereign blockchain network that:

- Runs its own VM (typically Subnet-EVM, a fork of go-ethereum)
- Has its own set of validators
- Has its own gas token (can be any ERC-20 or native token)
- Achieves the same sub-second finality as the primary network
- Communicates with other L1s and the primary network via ICM (Interchain Messaging)

### Pre-Etna vs Post-Etna

**Before Etna**: Every L1 validator was required to also validate the primary network, which required staking 2,000 AVAX (~$100,000+ at peak prices). This made L1 deployment expensive.

**After Etna (ACP-77)**: L1 validators no longer need to validate the primary network. They pay a small continuous fee in AVAX instead. This reduced the cost of running an L1 validator by ~99.9%.

See `etna-changes.md` for full details.

## Validators

- **Primary network validators**: Stake ≥ 2,000 AVAX on P-Chain, validate C/P/X chains
- **L1-only validators (post-Etna)**: Pay continuous fee, validate only their L1(s)
- **Minimum stake**: 2,000 AVAX for primary network; L1-specific for L1-only validators
- **Uptime requirement**: ≥ 80% uptime to earn staking rewards
- **Delegation**: Primary network validators can accept delegations (up to 5x their own stake)

## AVAX Tokenomics

- **C-Chain gas**: AVAX is burned as gas on C-Chain (deflationary)
- **P-Chain staking**: AVAX locked as stake earns staking rewards
- **L1 validator fees**: Post-Etna, L1 validators pay continuous AVAX fees to the primary network
- **Total supply**: 720 million AVAX (fixed cap)
- **Initial distribution**: 50% to staking rewards pool, 50% to foundation/team/ecosystem

## How Avalanche L1s Communicate

Avalanche L1s communicate natively via **ICM (Interchain Messaging)**, built on **AWM (Avalanche Warp Messaging)**. This is a trust-minimized protocol where messages are signed by the source chain's validators — no external bridge operator required.

See the `avalanche-icm` skill for full details.
