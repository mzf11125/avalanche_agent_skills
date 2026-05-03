# Avalanche Developer AI Toolkit - `avalanche_agent_skills`

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Retro9000](https://img.shields.io/badge/Retro9000-Tooling-red)](https://retro9000.avax.network)

Modular AI agent skill packs that make building on Avalanche L1s dramatically easier for developers using AI-assisted workflows. Install once - your AI assistant becomes an Avalanche expert.

## Quick Install

```bash
# All skills
npx skills add https://github.com/your-username/avalanche_agent_skills

# Individual skills
npx skills add https://github.com/your-username/avalanche_agent_skills --skill avalanche-concepts
npx skills add https://github.com/your-username/avalanche_agent_skills --skill avalanche-evm
npx skills add https://github.com/your-username/avalanche_agent_skills --skill avalanche-icm
npx skills add https://github.com/your-username/avalanche_agent_skills --skill avalanche-ictt
npx skills add https://github.com/your-username/avalanche_agent_skills --skill avalanche-network
```

## Skill Modules

| Skill | Description |
|-------|-------------|
| [`avalanche-concepts`](./avalanche-concepts/) | Avalanche architecture, Etna/ACP-77, consensus, L1 primitives, 80+ term glossary |
| [`avalanche-evm`](./avalanche-evm/) | Subnet-EVM, precompiles, genesis config, Hardhat/Foundry setup, 20+ examples |
| [`avalanche-icm`](./avalanche-icm/) ⭐ | ICM/Teleporter/AWM, relayer setup, cross-chain messaging, 15+ examples |
| [`avalanche-ictt`](./avalanche-ictt/) ⭐ | ICTT token bridging, TokenHome/TokenRemote, ERC-20 & native transfers |
| [`avalanche-network`](./avalanche-network/) | AvalancheGo, validators, Docker, Prometheus/Grafana monitoring |

⭐ = Priority skills for Retro9000 (ICM/ICTT explicitly preferred)

## MCP Server

For live Avalanche network data in your AI agent, see the companion project:
**[avalanche-mcp-server](https://github.com/your-username/avalanche-mcp-server)** - 13 MCP tools for querying L1 stats, ICM messages, ICTT transfers, and more.

## What Problems This Solves

- **AI hallucination on Avalanche code** - LLMs trained before Etna produce wrong ICM/ICTT code. These skills inject accurate, up-to-date context.
- **Scattered tooling docs** - AvalancheGo, Subnet-EVM, Teleporter, ICTT, AWM relayer all in one place.
- **No AI-native onboarding** - Developers can now ask their AI assistant Avalanche questions and get correct answers on the first try.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs welcome - especially additional code examples and coverage of new Avalanche features.

## License

MIT - see [LICENSE](LICENSE).
