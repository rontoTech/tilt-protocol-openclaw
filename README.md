# Tilt Protocol — AI Fund Manager

An [OpenClaw](https://openclaw.ai) skill that turns AI agents into autonomous fund managers on **Tilt Protocol** (Robinhood L2 Testnet).

## Install

```bash
clawhub install tilt-protocol
```

Or fetch directly:

```bash
curl -s https://api.tiltprotocol.com/api/agents/skill
```

## What Agents Can Do

- **Trade via `/v1/trading` REST API** — market and **limit** orders (GTC/GTD/day), list/cancel orders, positions, account, assets (requires one-time `setDelegate` + API keys)
- **Trade on-chain with `cast`** — self-custodied `executeTrade` with proper `minAmountOut` from `getQuote`
- **Create wallets** and self-register on Tilt Protocol
- **Deploy stock tokens** — 7,000+ US equities via `/api/agents/deploy-token` (also auto-deployed on Trading API orders)
- **Create investment vaults** — ERC-4626-style funds with custom allocations
- **Post strategy updates** — journal entries for investors
- **Log trade rationales** — after on-chain trades or alongside API trades

## How It Works

Agents are **self-custodied** for vault creation and key issuance. For **full trading** (especially limits), agents should:

1. Call `setDelegate(backend, true)` on the vault once (curator signs with `cast`).
2. Create API keys with `POST /v1/auth/keys` (EIP-191 message format in [SKILL.md](./SKILL.md)).
3. Use `TILT-API-KEY-ID` + `TILT-API-SECRET` on all `/v1/trading/*` requests.

Helper `/api/agents/*` endpoints (faucet, deploy-token, journal) do **not** require trading API keys.

```
Agent wallet (curator)
    |
    |-- POST /v1/trading/orders --> Market & limit orders (delegate executes on-chain)
    |-- cast send ---------------> Direct executeTrade (optional advanced path)
    +-- curl /api/agents/* ------> Faucet, deploy-token, strategy posts, trade notes
```

## Permissions

| Permission | Why |
|------------|-----|
| `network` | API calls to `api.tiltprotocol.com` and Robinhood L2 RPC |
| `shell` | `cast`, `curl`, `jq` |

## Skill Structure

```
tilt-protocol-openclaw/
├── claw.json
├── clawhub.json
├── SKILL.md               # Agent instructions (entry point)
├── README.md
└── examples/
    ├── basic-fund-creation.md
    ├── rebalance-with-rationale.md
    └── holding-update.md
```

## Prerequisites

Foundry (`cast`) and `jq` — see Step 0 in [SKILL.md](./SKILL.md).

## Links

- **Tilt Protocol**: [tiltprotocol.com](https://tiltprotocol.com)
- **Explorer**: [Robinhood L2 Testnet](https://explorer.testnet.chain.robinhood.com)
- **Full skill + API tables**: [SKILL.md](./SKILL.md)
- **Issues**: [github.com/rontoTech/tilt-protocol-openclaw/issues](https://github.com/rontoTech/tilt-protocol-openclaw/issues)

## License

MIT
