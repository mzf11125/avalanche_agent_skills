# Contributing to avalanche-agent-skills

Thank you for helping improve the Avalanche Developer AI Toolkit!

## Fork & PR Workflow

1. Fork the repo and create a branch: `git checkout -b feat/your-feature`
2. Make your changes following the guidelines below
3. Commit using [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `docs:`, `chore:`
4. Open a PR against `main` with a clear description of what changed and why

## Issue Labels

| Label | Use for |
|-------|---------|
| `bug` | Incorrect information in a skill |
| `enhancement` | New content or improved coverage |
| `skill-content` | Changes to skill markdown files |
| `docs` | README, CONTRIBUTING, or meta docs |

## Adding a New Skill Module

1. Create a directory: `avalanche-<name>/`
2. Add `SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: avalanche-<name>
   description: <detailed description - this is what triggers agent invocation>
   version: 0.1.0
   ---
   ```
3. Add your content files (markdown)
4. Register the skill in `skillsrc.json`
5. Add a row to the skills table in `README.md`

## Content Standards

- All code examples must be complete and runnable (no `...` placeholders in critical paths)
- Solidity examples: include SPDX license, pragma, imports, and comments
- TypeScript examples: use ethers.js v6, include imports
- Verify contract addresses against official Avalanche docs before committing
- Keep each markdown file focused on one topic
