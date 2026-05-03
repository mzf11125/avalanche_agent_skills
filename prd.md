# Product Requirements Document
# Avalanche Developer AI Toolkit
### `avalanche_agent_skills` + `avalanche-mcp-server`

**Version:** 1.0  
**Status:** Draft  
**Author:** [Your Name]  
**Program Target:** Retro9000 - Avalanche L1 & Infrastructure Tooling Round  
**Next Snapshot Deadline:** July 14, 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Vision & Goals](#3-vision--goals)
4. [Target Users](#4-target-users)
5. [Success Metrics](#5-success-metrics)
6. [Product 1 - `avalanche_agent_skills`](#6-product-1--avalanche_agent_skills)
7. [Product 2 - `avalanche-mcp-server`](#7-product-2--avalanche-mcp-server)
8. [Technical Architecture](#8-technical-architecture)
9. [Retro9000 Alignment](#9-retro9000-alignment)
10. [Roadmap](#10-roadmap)
11. [Open Source & Licensing](#11-open-source--licensing)
12. [Out of Scope](#12-out-of-scope)

---

## 1. Executive Summary

The **Avalanche Developer AI Toolkit** is a two-layer open-source project that makes building on Avalanche L1s dramatically easier for developers using AI-assisted workflows.

**Layer 1 - `avalanche_agent_skills`:** A modular set of installable agent skill packs (compatible with Claude, Cursor, Windsurf, and any tool supporting the Agent Skills open standard) covering Avalanche L1 concepts, EVM development, Interchain Messaging (ICM), Interchain Token Transfer (ICTT), and network/validator operations.

**Layer 2 - `avalanche-mcp-server`:** A Model Context Protocol (MCP) server that gives AI agents the ability to query live Avalanche network data, inspect ICM messages, check validator health, and scaffold L1 deployments - directly from within any MCP-compatible development environment.

Together, these tools create the AI-native developer layer for the Avalanche ecosystem. Any developer using an AI coding assistant can now understand, query, and build on Avalanche L1s without leaving their tool.

---

## 2. Problem Statement

### The Avalanche L1 Onboarding Gap

Avalanche's Etna upgrade and the Retro9000 program have lowered the cost of deploying an L1 by 99.9%, creating a surge of new L1 builders. However, the developer experience for *learning and building on* Avalanche L1s has not kept pace with AI-first development workflows.

Specifically:

- **AI assistants are blind to Avalanche-specific primitives.** When a developer asks Claude or Cursor how to use ICM or deploy an Avalanche L1, the model either hallucinates, falls back to generic EVM advice, or produces outdated information. The Etna upgrade and ACP-77 are recent enough that most LLM training data predates them.

- **There is no live data layer for AI agents.** Avalanche's Builder Hub provides AI-friendly static documentation endpoints, but no MCP server exists that lets an agent *query the live network* - validator status, cross-chain messages, L1 metrics - in real time.

- **Tooling context is scattered.** AvalancheGo, Subnet-EVM, AvaCloud, AWM relayer, ICM, ICTT, Teleporter - each has its own docs, SDKs, and deployment patterns. No unified agent context exists that covers the full stack.

### Why This Matters for Retro9000

Retro9000 is funding the builders of Avalanche L1s. Every team that submits a project first has to figure out how to build on Avalanche. The harder that onboarding is, the fewer projects ship - and the fewer projects ship, the smaller the ecosystem. This toolkit reduces friction for every L1 builder in the program.

---

## 3. Vision & Goals

**Vision:** Make Avalanche L1 development the fastest, most AI-native blockchain developer experience in the world.

### Goals

**G1 - Reduce AI hallucination for Avalanche-specific code.** Developers who install `avalanche_agent_skills` should be able to write correct ICM, ICTT, and Subnet-EVM code in their AI assistant on the first attempt.

**G2 - Give AI agents live Avalanche network access.** Developers who run `avalanche-mcp-server` should be able to ask their AI agent questions like "what's the ICM message volume between my L1 and C-Chain today?" and get a real answer.

**G3 - Explicitly support ICM and ICTT.** These are Avalanche-native protocols that Retro9000 explicitly preferences in its evaluation criteria. Both products are designed around them.

**G4 - Be the default AI layer for Avalanche L1 builders.** Within six months of launch, this toolkit should be referenced in Avalanche Discord, Avalanche Academy, and the Builder Hub docs.

---

## 4. Target Users

### Primary: Avalanche L1 Builders

Developers who are building or operating an Avalanche L1 - whether through Retro9000 or otherwise. They are:
- Familiar with EVM/Solidity but new to Avalanche-specific tooling
- Using AI coding assistants daily (Claude, Cursor, Windsurf, GitHub Copilot)
- Struggling to get useful answers from LLMs about ICM, Subnet-EVM customization, and AvalancheGo node setup

### Secondary: DevOps / Validator Operators

Teams running validator nodes for Avalanche L1s who want to monitor health and troubleshoot issues through conversational AI queries rather than raw CLI commands.

### Tertiary: Avalanche dApp Developers

Developers building on existing Avalanche L1s (not deploying their own) who need to understand how to interact with ICM/ICTT from their smart contracts or frontend.

---

## 5. Success Metrics

### Adoption Metrics (Primary - measured at Retro9000 snapshot)

| Metric | Target (July 14 Snapshot) |
|--------|--------------------------|
| GitHub stars - agent skills | ≥ 100 |
| GitHub stars - MCP server | ≥ 50 |
| NPM installs - MCP server | ≥ 200 |
| `npx skills add` installs | ≥ 300 |
| Retro9000 community votes | Top 10 tooling projects |

### Quality Metrics

| Metric | Target |
|--------|--------|
| ICM/ICTT skill coverage | 100% of official Avalanche ICM docs |
| MCP tool response accuracy | ≥ 95% on test suite |
| Documented working code examples | ≥ 100 |
| README / SKILL.md completeness | Passes Avalanche Foundation tooling checklist |

### Community Metrics

| Metric | Target |
|--------|--------|
| Discord mentions (Avalanche server) | ≥ 20 organic |
| External blog posts / tutorials referencing the toolkit | ≥ 3 |
| L1 teams publicly citing the toolkit | ≥ 5 |

---

## 6. Product 1 - `avalanche_agent_skills`

### 6.1 Overview

A set of modular agent skill packs installable via `npx skills add`, compatible with Claude Code, Cursor, Windsurf, GitHub Copilot, and any tool supporting the Agent Skills open standard (launched December 2025).

Each skill is a structured markdown knowledge base that an AI agent loads as context, making it an expert in that domain without needing to be prompted or retrained.

### 6.2 Installation

```bash
# Install all skills
npx skills add https://github.com/[username]/avalanche_agent_skills

# Install individual skills
npx skills add https://github.com/[username]/avalanche_agent_skills --skill avalanche-concepts
npx skills add https://github.com/[username]/avalanche_agent_skills --skill avalanche-evm
npx skills add https://github.com/[username]/avalanche_agent_skills --skill avalanche-icm
npx skills add https://github.com/[username]/avalanche_agent_skills --skill avalanche-ictt
npx skills add https://github.com/[username]/avalanche_agent_skills --skill avalanche-network
```

### 6.3 Skill Modules

---

#### Skill 1: `avalanche-concepts`

**Purpose:** Foundational knowledge about Avalanche's architecture, the Etna upgrade, and L1 primitives. Eliminates hallucinations about how Avalanche differs from other EVM chains.

**Key Topics:**
- Avalanche consensus mechanism (Snowball, Snowflake, Snowman)
- The three-chain primary network (C-Chain, P-Chain, X-Chain)
- Subnet vs. Avalanche L1 nomenclature (post-Etna)
- ACP-77 and what changed with Etna
- Permissioned vs. permissionless L1s
- Validators and the P-Chain staking model
- AVAX tokenomics and fee mechanics
- How Avalanche L1s communicate (ICM overview)
- Comprehensive glossary (80+ terms)

**Content Specs:**
- Quick-start conceptual guide (10 minutes to read)
- Side-by-side comparison: Ethereum L2 vs. Avalanche L1
- Common misconceptions and corrections
- Links to canonical Avalanche Academy resources

**File structure:**
```
avalanche-concepts/
  SKILL.md              # Agent skill manifest
  overview.md           # Architecture overview
  etna-changes.md       # What changed with Etna / ACP-77
  vs-ethereum.md        # Comparison guide
  glossary.md           # 80+ term glossary
  faq.md                # Common developer questions
```

---

#### Skill 2: `avalanche-evm`

**Purpose:** Complete guide to building smart contracts on Avalanche L1s using Subnet-EVM. Covers customizations unique to Avalanche that generic EVM docs don't cover.

**Key Topics:**
- Subnet-EVM vs. standard EVM: what's different
- Precompiles (NativeAssetBalance, WarpMessenger, FeeManager, AllowList)
- Genesis file configuration
- Native token minting and management
- Custom gas fee configuration (fee manager precompile)
- Deploying contracts to Avalanche C-Chain vs. L1s
- Hardhat and Foundry configuration for Avalanche
- TypeScript/ethers.js integration patterns
- ABI encoding for Avalanche-specific precompiles
- Testing on Fuji testnet

**Content Specs:**
- 10-minute quick start: deploy a contract to a local Avalanche L1
- Complete precompile reference with ABI signatures
- Working Hardhat config template
- Working Foundry config template
- 20+ annotated Solidity examples
- Common deployment errors and fixes

**File structure:**
```
avalanche-evm/
  SKILL.md
  quick-start.md
  precompiles/
    overview.md
    native-asset.md
    warp-messenger.md
    fee-manager.md
    allow-list.md
  deployment/
    hardhat.md
    foundry.md
    avacloud.md
  testing.md
  examples/             # 20+ .sol files
```

---

#### Skill 3: `avalanche-icm` ⭐ (Priority Skill)

**Purpose:** Complete guide to Avalanche Interchain Messaging (ICM) - the native cross-chain communication protocol. This is the highest-priority skill because Retro9000 explicitly preferences ICM integration.

**Key Topics:**
- ICM architecture and Warp messaging
- AWM (Avalanche Warp Messaging) protocol internals
- Teleporter: the high-level ICM SDK
- Sending a cross-chain message: step-by-step
- Receiving and verifying ICM messages in Solidity
- Message fees and delivery guarantees
- ICM relayer: setup, configuration, monitoring
- Teleporter Registry and contract addresses
- ICM on Fuji testnet
- Debugging failed cross-chain messages
- ICM message indexing and querying
- Security considerations and trust assumptions

**Content Specs:**
- 15-minute quick start: send your first ICM message C-Chain → L1
- Complete Teleporter contract interface reference
- Full relayer configuration reference
- Working end-to-end example (Solidity + TypeScript)
- Message lifecycle diagram (text-based for agent context)
- 15+ annotated code examples
- Troubleshooting guide for common ICM failures

**File structure:**
```
avalanche-icm/
  SKILL.md
  quick-start.md
  architecture.md
  teleporter/
    overview.md
    sending.md
    receiving.md
    registry.md
    fees.md
  relayer/
    setup.md
    configuration.md
    monitoring.md
  debugging.md
  security.md
  examples/             # 15+ examples
  reference/
    contract-addresses.md
    abi.md
```

---

#### Skill 4: `avalanche-ictt` ⭐ (Priority Skill)

**Purpose:** Complete guide to Avalanche Interchain Token Transfer (ICTT) - the native cross-chain token bridge. Also explicitly preferred by Retro9000.

**Key Topics:**
- ICTT architecture and token flow
- Native token transfers vs. ERC-20 transfers
- Deploying a TokenHome contract (source chain)
- Deploying a TokenRemote contract (destination chain)
- ERC-20 cross-chain transfers: complete walkthrough
- Native asset bridging (AVAX and custom native tokens)
- Collateral requirements and locking mechanics
- Multi-hop transfers across multiple L1s
- ICTT fee mechanics
- Registering token remotes
- Frontend integration patterns
- Security model and trust assumptions

**Content Specs:**
- 15-minute quick start: bridge an ERC-20 from C-Chain to an L1
- Complete TokenHome and TokenRemote interface reference
- Working deployment scripts (TypeScript)
- Working frontend example (ethers.js)
- 10+ annotated code examples
- Common failure modes and debugging guide

**File structure:**
```
avalanche-ictt/
  SKILL.md
  quick-start.md
  architecture.md
  token-home/
    overview.md
    deployment.md
    configuration.md
  token-remote/
    overview.md
    deployment.md
    registration.md
  transfers/
    erc20.md
    native.md
    multi-hop.md
  fees.md
  frontend.md
  security.md
  examples/
```

---

#### Skill 5: `avalanche-network`

**Purpose:** Network infrastructure, node operations, validator setup, and monitoring for Avalanche L1s.

**Key Topics:**
- AvalancheGo: installation, configuration, upgrade management
- Node types: validator vs. API node vs. archival
- Becoming a validator on an Avalanche L1
- P-Chain staking transactions
- Docker deployment for production validators
- Avalanche L1 genesis configuration
- Custom VM deployment
- Monitoring with Prometheus + Grafana (AvalancheGo metrics)
- AvaCloud: managed L1 deployment
- Builder Console: managing L1s from the UI
- Network upgrade patterns
- Troubleshooting: node sync issues, validator problems

**Content Specs:**
- Complete AvalancheGo config reference
- Docker Compose templates for validator + monitoring stack
- Prometheus metric catalog for AvalancheGo
- Grafana dashboard template (JSON)
- Step-by-step validator setup guide
- AvaCloud vs. self-hosted comparison

**File structure:**
```
avalanche-network/
  SKILL.md
  avalanchego/
    installation.md
    configuration.md
    upgrades.md
  validators/
    setup.md
    p-chain-staking.md
    l1-validators.md
  docker/
    production.md
    docker-compose.yml  # template
  monitoring/
    prometheus.md
    grafana.md
    dashboard.json      # template
  avacloud.md
  genesis.md
  troubleshooting.md
```

---

### 6.4 Repository Structure

```
avalanche_agent_skills/
├── README.md                    # Project overview + quick start
├── LICENSE                      # MIT
├── CONTRIBUTING.md
├── CHANGELOG.md
├── package.json                 # skills CLI metadata
├── skillsrc.json                # skill registry config
├── avalanche-concepts/
├── avalanche-evm/
├── avalanche-icm/
├── avalanche-ictt/
└── avalanche-network/
```

### 6.5 Documentation Stats (Target)

| Stat | Target |
|------|--------|
| Skill modules | 5 |
| Total markdown files | ≥ 60 |
| Total lines of documentation | ≥ 25,000 |
| Working code examples | ≥ 100 |
| Official Avalanche doc coverage | ≥ 75% |

---

## 7. Product 2 - `avalanche-mcp-server`

### 7.1 Overview

A Model Context Protocol (MCP) server that exposes live Avalanche network data and developer utilities as tools callable by any MCP-compatible AI agent. Unlike the agent skills (which are static knowledge), the MCP server gives agents the ability to *act* and *query* in real time.

### 7.2 Installation & Setup

```bash
# Install globally
npm install -g avalanche-mcp-server

# Or run directly
npx avalanche-mcp-server

# Configure in Claude Desktop (claude_desktop_config.json)
{
  "mcpServers": {
    "avalanche": {
      "command": "npx",
      "args": ["avalanche-mcp-server"],
      "env": {
        "AVACLOUD_API_KEY": "your-key-here"   // optional, for enhanced data
      }
    }
  }
}
```

### 7.3 MCP Tools Specification

The server exposes tools across four categories.

---

#### Category 1: Network & L1 Data

---

**Tool: `get_avalanche_l1s`**

Fetch the list of live Avalanche L1s with key metadata.

```typescript
// Input
{
  status?: "live" | "testnet" | "all"   // default: "live"
  limit?: number                         // default: 20
}

// Output
{
  l1s: [{
    id: string            // blockchain ID
    name: string
    vmId: string
    validators: number
    transactions_24h: number
    active_addresses_24h: number
    tvl_usd?: number
  }]
}
```

**Data source:** AvaCloud Data API / public Avalanche APIs

---

**Tool: `get_l1_stats`**

Fetch real-time statistics for a specific Avalanche L1.

```typescript
// Input
{
  blockchain_id: string     // e.g. "2q9e4r6Mu3U68nU1fYjgbR6JvwrRx36CohpAX5UQxse55x1Q5" 
                            // OR chain name e.g. "dexalot"
  timeframe?: "24h" | "7d" | "30d"  // default: "24h"
}

// Output
{
  name: string
  blockchain_id: string
  stats: {
    transactions: number
    active_addresses: number
    gas_used: number
    avg_tps: number
    peak_tps: number
    validators: number
    tvl_usd?: number
  }
  timeframe: string
  updated_at: string
}
```

---

**Tool: `get_validator_info`**

Check the status of a validator on any Avalanche L1 or the primary network.

```typescript
// Input
{
  node_id: string           // e.g. "NodeID-5mb46qkSBj81k9g9e1af1uAGbFjGcr1LL"
  blockchain_id?: string    // if omitted, checks primary network
}

// Output
{
  node_id: string
  status: "active" | "inactive" | "pending"
  stake_amount?: number
  uptime_percent: number
  connected: boolean
  delegators?: number
  validation_start: string
  validation_end: string
  rewards_address?: string
}
```

---

**Tool: `get_network_overview`**

High-level Avalanche network health snapshot.

```typescript
// Input: none

// Output
{
  primary_network: {
    c_chain_tps: number
    total_validators: number
    total_l1s: number
    staked_avax: number
    avax_price_usd?: number
  }
  top_l1s_by_activity: L1Summary[]
  network_health: "healthy" | "degraded" | "outage"
  updated_at: string
}
```

---

#### Category 2: ICM (Interchain Messaging)

---

**Tool: `get_icm_messages`**

Fetch recent ICM messages between chains, with filtering options.

```typescript
// Input
{
  source_chain?: string       // blockchain ID or "c-chain"
  destination_chain?: string  // blockchain ID or "c-chain"
  limit?: number              // default: 20, max: 100
  status?: "pending" | "delivered" | "failed" | "all"
  from_timestamp?: string     // ISO 8601
}

// Output
{
  messages: [{
    message_id: string
    source_chain: string
    destination_chain: string
    sender: string            // address
    status: "pending" | "delivered" | "failed"
    timestamp: string
    fee_paid: string          // in AVAX
    payload_hex: string
    delivery_tx?: string      // tx hash on destination
  }]
  total: number
}
```

---

**Tool: `get_icm_stats`**

ICM volume and performance statistics between chains.

```typescript
// Input
{
  source_chain?: string
  destination_chain?: string
  timeframe?: "24h" | "7d" | "30d"  // default: "24h"
}

// Output
{
  total_messages: number
  delivered: number
  pending: number
  failed: number
  avg_delivery_time_seconds: number
  total_fees_avax: string
  top_routes: [{
    source: string
    destination: string
    message_count: number
  }]
}
```

---

**Tool: `check_icm_message`**

Look up the delivery status of a specific ICM message by ID or tx hash.

```typescript
// Input
{
  message_id?: string
  source_tx_hash?: string
  source_chain: string
}

// Output
{
  message_id: string
  status: "pending" | "delivered" | "failed"
  source_chain: string
  destination_chain: string
  source_tx: string
  delivery_tx?: string
  delivery_timestamp?: string
  failure_reason?: string
  fee_paid: string
}
```

---

#### Category 3: Token & ICTT

---

**Tool: `get_ictt_tokens`**

List tokens deployed via ICTT (Interchain Token Transfer), with their home and remote chain info.

```typescript
// Input
{
  chain?: string              // filter by chain
  token_address?: string      // filter by address
}

// Output
{
  tokens: [{
    name: string
    symbol: string
    token_home_chain: string
    token_home_address: string
    remotes: [{
      chain: string
      remote_address: string
      collateral_locked: string
      supply_remote: string
    }]
    total_bridged_usd?: number
  }]
}
```

---

**Tool: `get_ictt_transfers`**

Fetch recent token transfers via ICTT.

```typescript
// Input
{
  token_address?: string
  chain?: string
  limit?: number             // default: 20
  direction?: "in" | "out" | "all"
}

// Output
{
  transfers: [{
    transfer_id: string
    token: string
    amount: string
    from_chain: string
    to_chain: string
    sender: string
    recipient: string
    status: "pending" | "complete"
    timestamp: string
    source_tx: string
    destination_tx?: string
  }]
}
```

---

#### Category 4: Developer Utilities

---

**Tool: `scaffold_l1_deployment`**

Generate a complete deployment scaffold for a new Avalanche L1.

```typescript
// Input
{
  l1_name: string
  vm_type: "subnet-evm" | "custom-evm"
  permissioned: boolean
  native_token_symbol: string
  native_token_name: string
  initial_supply?: string          // default: "240000000000000000000000000"
  enable_icm: boolean              // default: true
  enable_fee_manager: boolean      // default: false
  fee_config?: {
    gas_limit?: number
    target_block_rate?: number
    min_base_fee?: number
  }
}

// Output
{
  files: [{
    path: string
    content: string
  }]
  // Includes:
  // - genesis.json
  // - hardhat.config.ts
  // - deploy/01_deploy_l1.ts
  // - .env.example
  // - README.md
  setup_commands: string[]
}
```

---

**Tool: `get_contract_addresses`**

Fetch official Avalanche contract addresses (Teleporter, ICTT contracts, etc.) for any network.

```typescript
// Input
{
  network: "mainnet" | "fuji"
  contracts?: string[]    // e.g. ["teleporter", "teleporter-registry", "ictt-home"]
}

// Output
{
  network: string
  addresses: {
    [contract_name: string]: string
  }
  updated_at: string
}
```

---

**Tool: `decode_warp_message`**

Decode a raw Avalanche Warp message payload for debugging.

```typescript
// Input
{
  payload_hex: string
  message_type?: "icm" | "ictt" | "unknown"
}

// Output
{
  decoded: {
    message_type: string
    source_chain_id: string
    fields: Record<string, any>
  }
  raw_hex: string
}
```

---

**Tool: `check_avacloud_status`**

Check the operational status of AvaCloud services.

```typescript
// Input: none

// Output
{
  status: "operational" | "degraded" | "outage"
  services: [{
    name: string
    status: string
    last_incident?: string
  }]
  updated_at: string
}
```

---

### 7.4 Tool Summary Table

| Tool | Category | Data Source | Auth Required |
|------|----------|-------------|---------------|
| `get_avalanche_l1s` | Network | AvaCloud / public APIs | No |
| `get_l1_stats` | Network | AvaCloud Data API | Optional (key for higher limits) |
| `get_validator_info` | Network | AvalancheGo P-Chain API | No |
| `get_network_overview` | Network | Multiple | No |
| `get_icm_messages` | ICM | AvaCloud / AWM Indexer | Optional |
| `get_icm_stats` | ICM | AvaCloud / AWM Indexer | Optional |
| `check_icm_message` | ICM | AvaCloud / AWM Indexer | Optional |
| `get_ictt_tokens` | ICTT | AvaCloud / on-chain | Optional |
| `get_ictt_transfers` | ICTT | AvaCloud / on-chain | Optional |
| `scaffold_l1_deployment` | Dev Utils | Local generation | No |
| `get_contract_addresses` | Dev Utils | Static + verified | No |
| `decode_warp_message` | Dev Utils | Local decoding | No |
| `check_avacloud_status` | Dev Utils | AvaCloud status API | No |

**Total: 13 tools across 4 categories**

---

### 7.5 Configuration

The server reads from environment variables with sensible defaults. All tools work without any API key using public endpoints, with higher rate limits available via AvaCloud.

```env
# Optional - enables higher rate limits and additional data
AVACLOUD_API_KEY=

# Optional - override default RPC endpoints
AVAX_MAINNET_RPC=https://api.avax.network/ext/bc/C/rpc
AVAX_FUJI_RPC=https://api.avax-test.network/ext/bc/C/rpc

# Optional - default network (default: mainnet)
DEFAULT_NETWORK=mainnet

# Optional - request timeout in ms (default: 10000)
REQUEST_TIMEOUT_MS=10000
```

### 7.6 Repository Structure

```
avalanche-mcp-server/
├── README.md
├── LICENSE                          # MIT
├── CONTRIBUTING.md
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts                     # MCP server entry point
│   ├── tools/
│   │   ├── network/
│   │   │   ├── get-l1s.ts
│   │   │   ├── get-l1-stats.ts
│   │   │   ├── get-validator-info.ts
│   │   │   └── get-network-overview.ts
│   │   ├── icm/
│   │   │   ├── get-icm-messages.ts
│   │   │   ├── get-icm-stats.ts
│   │   │   └── check-icm-message.ts
│   │   ├── ictt/
│   │   │   ├── get-ictt-tokens.ts
│   │   │   └── get-ictt-transfers.ts
│   │   └── utils/
│   │       ├── scaffold-l1.ts
│   │       ├── get-contract-addresses.ts
│   │       ├── decode-warp-message.ts
│   │       └── check-avacloud-status.ts
│   ├── api/
│   │   ├── avacloud.ts              # AvaCloud API client
│   │   ├── avalanchego.ts           # AvalancheGo RPC client
│   │   └── indexer.ts               # AWM indexer client
│   ├── types/
│   │   └── index.ts                 # Shared TypeScript types
│   └── utils/
│       ├── format.ts
│       └── validate.ts
├── tests/
│   ├── tools/                       # Unit tests per tool
│   └── integration/                 # Integration tests vs. Fuji
└── docs/
    ├── tools/                       # Per-tool documentation
    └── examples/                    # Example agent interactions
```

---

## 8. Technical Architecture

### 8.1 Agent Skills Architecture

```
Developer's AI Tool (Claude / Cursor / Windsurf)
         │
         │  loads at startup
         ▼
┌─────────────────────────────────────────────────┐
│            avalanche_agent_skills               │
│                                                 │
│  ┌──────────────┐  ┌──────────────────────────┐│
│  │ avalanche-   │  │ avalanche-icm            ││
│  │ concepts     │  │ (Teleporter, AWM,        ││
│  └──────────────┘  │  relayer, debugging)     ││
│  ┌──────────────┐  └──────────────────────────┘│
│  │ avalanche-   │  ┌──────────────────────────┐│
│  │ evm          │  │ avalanche-ictt           ││
│  │ (Subnet-EVM, │  │ (TokenHome, TokenRemote, ││
│  │  precompiles)│  │  bridging patterns)      ││
│  └──────────────┘  └──────────────────────────┘│
│  ┌──────────────┐                               │
│  │ avalanche-   │                               │
│  │ network      │                               │
│  │ (validators, │                               │
│  │  monitoring) │                               │
│  └──────────────┘                               │
└─────────────────────────────────────────────────┘
```

### 8.2 MCP Server Architecture

```
Developer's AI Tool (Claude / Cursor / any MCP client)
         │
         │  MCP protocol (JSON-RPC over stdio/SSE)
         ▼
┌─────────────────────────────────────────────────┐
│           avalanche-mcp-server                  │
│                                                 │
│  ┌────────────┐  ┌────────────┐  ┌───────────┐ │
│  │  Network   │  │    ICM     │  │   ICTT    │ │
│  │   Tools    │  │   Tools    │  │   Tools   │ │
│  └─────┬──────┘  └─────┬──────┘  └─────┬─────┘ │
│        │               │               │        │
│  ┌─────┴───────────────┴───────────────┴─────┐  │
│  │              API Clients                  │  │
│  │  AvaCloud API │ AvalancheGo RPC │ Indexer │  │
│  └─────┬─────────────────┬──────────────┬────┘  │
└────────┼─────────────────┼──────────────┼───────┘
         │                 │              │
         ▼                 ▼              ▼
   AvaCloud           AvalancheGo    AWM Message
   Data API           P/C/X-Chain    Indexer
                      RPC endpoints
```

### 8.3 Data Sources

| Source | Used For | Auth |
|--------|----------|------|
| AvaCloud Data API (`developers.avacloud.io`) | L1 stats, ICM/ICTT data, validator info | API key (optional) |
| AvalancheGo P-Chain RPC | Validator info, staking data | None |
| AvalancheGo C-Chain RPC | Block data, contract calls | None |
| AWM Message Indexer | ICM message tracking | None (public) |
| AvaCloud Status API | Service health | None |
| Local generation | Scaffold tool | None |

### 8.4 Technology Stack

| Component | Technology |
|-----------|-----------|
| MCP Server runtime | Node.js + TypeScript |
| MCP SDK | `@modelcontextprotocol/sdk` |
| HTTP client | `axios` |
| EVM interaction | `ethers.js` v6 |
| Agent Skills format | Anthropic Agent Skills open standard |
| Test framework | Vitest |
| Linting | ESLint + Prettier |

---

## 9. Retro9000 Alignment

### 9.1 Eligibility Checklist

| Requirement | Status | Evidence |
|-------------|--------|----------|
| Open-source | ✅ | MIT license on both repos |
| Publicly usable | ✅ | `npx` install for skills; npm package for MCP |
| Documentation | ✅ | 25,000+ lines across skill modules + MCP tool docs |
| Supports Avalanche L1 builders | ✅ | Entire product is designed for L1 builders |
| ICM/ICTT integration (preferred) | ✅ | Dedicated skill modules + 5 live MCP tools |
| Live on mainnet | ✅ | MCP server queries mainnet; skills reference mainnet contracts |

### 9.2 Evaluation Criteria Mapping

| Retro9000 Criterion | How This Project Addresses It |
|---------------------|-------------------------------|
| Development progress | Public GitHub with commit history; versioned releases |
| Ecosystem relevance | Directly lowers L1 builder onboarding friction |
| Technical merit | Live MCP tools querying real network data; comprehensive skill coverage |
| Community engagement | Retro9000 votes; Avalanche Discord presence; builder testimonials |
| ICM/ICTT integration | 2 dedicated skill modules + 5 ICM/ICTT MCP tools - maximum emphasis |

### 9.3 Submission Narrative

> The Avalanche Developer AI Toolkit is the missing AI-native layer for Avalanche L1 builders. Every developer building a Retro9000 project who uses an AI coding assistant now has access to accurate, up-to-date Avalanche knowledge and live network data - directly in their tool. We built this because we saw Avalanche L1 teams wasting hours debugging hallucinated ICM code. No more.

---

## 10. Roadmap

### Phase 1 - Foundation (Weeks 1–3)

**`avalanche_agent_skills` v0.1**
- [ ] `avalanche-concepts` skill - complete
- [ ] `avalanche-icm` skill - complete (priority)
- [ ] Repository structure, LICENSE, CONTRIBUTING.md
- [ ] GitHub Actions for linting/validation
- [ ] Submit to Retro9000 early for community votes

**`avalanche-mcp-server` v0.1**
- [ ] MCP server scaffolding with `@modelcontextprotocol/sdk`
- [ ] `get_network_overview` tool
- [ ] `get_l1_stats` tool
- [ ] `get_icm_messages` tool
- [ ] `check_icm_message` tool
- [ ] Published to npm
- [ ] Basic README with Claude Desktop setup instructions

**Community**
- [ ] Post in Avalanche Discord (#developer-tools)
- [ ] Create Retro9000 project listing

---

### Phase 2 - Expansion (Weeks 4–6)

**`avalanche_agent_skills` v0.2**
- [ ] `avalanche-evm` skill - complete
- [ ] `avalanche-ictt` skill - complete (priority)
- [ ] `avalanche-network` skill - complete
- [ ] CHANGELOG and versioning

**`avalanche-mcp-server` v0.2**
- [ ] `get_icm_stats` tool
- [ ] `get_ictt_tokens` tool
- [ ] `get_ictt_transfers` tool
- [ ] `get_validator_info` tool
- [ ] `get_avalanche_l1s` tool
- [ ] Integration tests vs. Fuji testnet

---

### Phase 3 - Polish & Growth (Weeks 7–10)

**Both products**
- [ ] `scaffold_l1_deployment` tool (MCP)
- [ ] `decode_warp_message` tool (MCP)
- [ ] `get_contract_addresses` tool (MCP)
- [ ] `check_avacloud_status` tool (MCP)
- [ ] End-to-end usage examples (blog post / video)
- [ ] Avalanche Academy integration pitch
- [ ] 5+ L1 builder teams onboarded and providing testimonials
- [ ] Retro9000 community vote campaign

**Pre-snapshot checklist (July 14)**
- [ ] Both repos public with full commit history
- [ ] ≥ 100 GitHub stars (agent skills)
- [ ] ≥ 200 npm installs (MCP server)
- [ ] Project listed on Retro9000 Discover page
- [ ] Regular update posts on Retro9000 dashboard
- [ ] Community votes accumulated

---

## 11. Open Source & Licensing

Both repositories are released under the **MIT License**.

**Why MIT:** Maximum permissiveness for the ecosystem. L1 teams can fork, embed, and redistribute without restriction. This aligns with Retro9000's preference for openly usable tooling.

**Governance:** Initially maintained by the founding team. CONTRIBUTING.md will define the PR process, issue labels, and versioning scheme (semver). External contributions welcome from Day 1.

**Publishing:**
- `avalanche_agent_skills` - GitHub only (consumed via `npx skills add`)
- `avalanche-mcp-server` - GitHub + npm (`avalanche-mcp-server`)

---

## 12. Out of Scope

The following are explicitly not in scope for v1 and the Retro9000 submission:

- **A web UI or dashboard** - this is a developer tool; no frontend
- **Wallet or key management** - the MCP server is read-only + scaffold-only; no signing
- **Support for non-Avalanche chains** - the product is Avalanche-only; no multi-chain abstraction
- **Custom VM (not Subnet-EVM) development guides** - Subnet-EVM covers the vast majority of L1s
- **Paid or freemium tiers** - fully free and open-source for the Retro9000 period
- **Mobile** - developer tooling; desktop/CLI only
- **GraphQL or REST API wrapping** - the MCP server is MCP-only; no separate HTTP API

---

*This document is a living PRD. Sections will be updated as the product evolves toward the July 14, 2026 Retro9000 snapshot.*