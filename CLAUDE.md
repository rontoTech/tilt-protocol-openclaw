# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

The canonical guide is @AGENTS.md. The single most important fact: pushing `main` instantly deploys SKILL.md to every live trading agent (served via `GET /api/agents/skill`). Verify contract addresses against `bowstring-ui/src/lib/contracts.ts` before editing, bump `claw.json` + `clawhub.json` together, and sync the vendored copy in `bowstring-backend/skills/tilt-protocol/`. The workspace-level guide (if present) is `../AGENTS.md`.
