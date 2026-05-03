# Subnet-EVM Overview

## What Is Subnet-EVM?

Subnet-EVM is Avalanche's customizable EVM implementation. Every Avalanche L1 that runs an EVM chain uses Subnet-EVM (or a fork of it). It is fully EVM-compatible - any Solidity contract that runs on Ethereum runs on Subnet-EVM without modification.

## Key Differences from Ethereum

| Feature | Ethereum | Subnet-EVM |
|---------|----------|------------|
| Consensus | PoS (Gasper) | Snowman (Avalanche consensus) |
| Block time | ~12s | Configurable, typically 1–2s |
| Native token | ETH | Configurable (default: AVAX on C-Chain) |
| Precompiles | Fixed set | Extensible with custom precompiles |
| Fee mechanism | EIP-1559 | Configurable via FeeManager precompile |
| Chain ID | Fixed | Configurable per L1 |
| Validator set | Global | Per-L1 (post-Etna: sovereign validators) |

## Subnet-EVM vs C-Chain

The C-Chain is itself a Subnet-EVM chain - it's the primary EVM chain on Avalanche. Custom L1s use the same Subnet-EVM binary with different genesis configurations.

## Precompile System

Subnet-EVM includes stateful precompiles at fixed addresses that extend EVM functionality:

| Precompile | Address | Purpose |
|-----------|---------|---------|
| ContractDeployerAllowList | `0x0200000000000000000000000000000000000000` | Restrict who can deploy contracts |
| ContractNativeMinter | `0x0200000000000000000000000000000000000001` | Mint native tokens |
| TxAllowList | `0x0200000000000000000000000000000000000002` | Restrict who can send transactions |
| FeeManager | `0x0200000000000000000000000000000000000003` | Dynamically update fee config |
| RewardManager | `0x0200000000000000000000000000000000000004` | Configure fee distribution |
| Warp | `0x0200000000000000000000000000000000000005` | AWM/ICM message signing |

## EVM Compatibility

- Solidity versions: all versions supported by the EVM opcode set
- Opcodes: full EVM opcode support including PUSH0 (Shanghai)
- JSON-RPC: full Ethereum JSON-RPC compatibility (`eth_*`, `net_*`, `web3_*`)
- Tools: MetaMask, Hardhat, Foundry, ethers.js, viem - all work without modification
- EIPs: EIP-1559 (type 2 transactions), EIP-2930 (access lists), EIP-4895 (withdrawals not applicable)

## GitHub

[github.com/ava-labs/subnet-evm](https://github.com/ava-labs/subnet-evm)
