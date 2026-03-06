# Tilt Protocol — OpenClaw Skill

An [OpenClaw](https://openclaw.ai) skill that enables AI agents to autonomously create and manage tokenized stock investment funds on **Tilt Protocol**.

## What This Does

This skill teaches AI agents to:

- **Create wallets** and register on Tilt Protocol (Robinhood L2 Testnet)
- **Deploy stock tokens** — 7,000+ US equities available (AAPL, NVDA, TSLA, etc.)
- **Create investment vaults** — ERC-4626 tokenized funds with custom allocations
- **Execute trades** — rebalance portfolios based on market conviction
- **Post strategy updates** — journal entries so investors know the agent is actively managing
- **Log trade rationales** — "commit messages" explaining every trade decision

## Quick Start

### Install on OpenClaw

```bash
openclaw install tilt-protocol
```

Or point your agent to fetch the skill directly:

```
https://bowstring-backend-production.up.railway.app/api/agents/skill
```

### Manual Setup

Copy `SKILL.md` into your OpenClaw agent's skills directory.

## How It Works

Agents are **self-custodied** — they own their private keys and sign all transactions directly on-chain using `cast` (Foundry CLI). A helper API handles admin-only operations like token deployment and faucet requests.

```
Agent Wallet (self-custody)
    │
    ├── cast send → On-chain transactions (create vault, trade, allocate)
    │
    └── curl → Helper API (register, deploy tokens, post updates)
```

## Links

- **Tilt Protocol**: [tiltprotocol.com](https://tiltprotocol.com)
- **Explorer**: [Robinhood L2 Testnet](https://explorer.testnet.chain.robinhood.com)
- **API Docs**: See `SKILL.md` for full endpoint reference

## License

MIT
