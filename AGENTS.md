# tilt-protocol-openclaw — Agent Guide

Last verified: 2026-07-06. The OpenClaw skill that turns any LLM agent into an autonomous Tilt Protocol fund manager. `SKILL.md` is the product.

## ⚠️ Pushing `main` is a production deploy

`GET https://api.tiltprotocol.com/api/agents/skill` serves `SKILL.md` fetched **live from GitHub raw @ main** (implemented in `bowstring-backend/src/agent-api.ts`, with local-file fallback), and agents are instructed to re-fetch it every session. Merging to main instantly changes the instructions every live agent follows — including the AI Arena operators. Review SKILL.md edits with that weight.

After editing SKILL.md, also sync the vendored copy `bowstring-backend/skills/tilt-protocol/SKILL.md` — it is currently drifted (as of 2026-07-06 it is missing the signed-key-revocation section); do not trust it as current.

## Rules

- Version bumps: `claw.json` and `clawhub.json` versions must stay equal (bump together).
- SKILL.md addresses agents in second person with lettered sections (§A trading API, §B bootstrap, §C on-chain cast) — other docs cross-reference by section letter; don't renumber casually.
- Contract addresses in SKILL.md's tables and cast examples have drifted before (old registry/factory addresses in §B examples vs the current table). The canonical live set is `bowstring-ui/src/lib/contracts.ts`; verify addresses there before editing, and check the runtime address book `GET /api/agents/contracts` served by the backend.
- Key behavioral contracts documented here that must track backend reality: unique `client_order_id` + verify-positions-before-retry after nonce-flavored rejections (§A4a); `trade-notes` require a `txHash`; `strategy-posts` rate limit 1 per 5 min, content 5–4000 chars Markdown, title ≤120 chars not repeated in content; limit orders need the vault to `setDelegate` the backend delegate `0xe67B013939D4118333d94B58FAf82936ca7eE978`; slippage minOut = quote × 9996/10000 (backend `SLIPPAGE_BPS=4`).
- `examples/` are narrated agent transcripts (fund creation, rebalance with rationale, hold update) — keep them consistent with SKILL.md when flows change.

## Context

The 7 AI Arena vaults are operated by an external OpenClaw deployment following this skill (all share curator `0x9B37D6acAd3dE36171F03ab3BC061cBe0c36dA64`); that runtime is not in any workspace repo. The separate "Singularity Capital" bot (ERC-8004 agent #612) also runs on this skill via a Railway service — backup/restore scripts live at the workspace root `scripts/`.
