---
name: tilt-protocol
environment: mainnet
description: Manage isolated immutable tokenized-equity vaults on Tilt Protocol using the Trading API and investor-signed basket transactions.
metadata: {"openclaw": {"homepage": "https://tiltprotocol.com"}}
---

# Tilt Protocol — mainnet beta fund manager

**Status: configured and closed; activation pending.** Immutable basket contracts were deployed on September 22, 2026, on Robinhood Chain **4663**. Health recovery and funded verification are still pending. This beta has not been independently audited. Mainnet transactions move real value and can lose the entire deposit. Do not transact until beta activation is confirmed and the deployment's live permissions permit the operation. Not financial advice.

Use this document only after the user explicitly selects mainnet. The default `SKILL.md` remains testnet, chain **46630**, at `https://api.tiltprotocol.com`, with the app at `https://testnet.tiltprotocol.com`. Do not automatically move a testnet agent, its orders, balances or API keys to mainnet.

After explicit mainnet selection, set `MAINNET_API_URL=https://tilt-mainnet-api-production.up.railway.app`. This separate API is deployed, but its availability does not announce beta activation. Fetch this document each session from `GET ${MAINNET_API_URL}/api/agents/skill`. Read `GET /api/agents/contracts` on that same origin, verify `chainId === 4663`, then check the connected RPC's `eth_chainId`. Mainnet responses include `vaultModel: "immutable-basket-v1"`, current ABIs, and only explicitly configured factory/router/lens addresses. A missing address means that feature is unavailable. Do not use testnet addresses or legacy `createUserVaultWithFees`, `deposit`, `redeem`, target-weight or allocation methods.

The mainnet app is `https://app.tiltprotocol.com`. Compare these deployed addresses with the live address book before signing:

| Contract | Mainnet address |
|---|---|
| USDG | `0x5fc5360D0400a0Fd4f2af552ADD042D716F1d168` |
| BasketVaultFactory | `0x463A6Db2731d4c5C5e9d937f7F9ccC82c2178D37` |
| VaultRegistry | `0xA977dE2a44e7eA96075b0b222D785a4103Bce3E2` |
| MainnetExecutionEngine | `0x2F73BBB4E73ad2CEdb2336ee74202D88D2f9709F` |
| ChainlinkPriceRouter | `0xA8ce6799F609E393Be18C685C05bffc29C27C0Cc` |
| TradeDelegateProxyV2 | `0x0f060b7b6331C27423c50fd386E27832AA410055` |
| BasketRouter | `0xB200A060eC342Ef65471595e0D506C830deAd8c1` |
| BasketVaultLens | `0x02eEFa923022E45F6f998e91a7Ad79bEbA9D5214` |
| OperationalHealthGate | `0xa4d43f4eeE47ba94CFC7f9F8B14DA6C89B7c06a4` |

Administration temporarily uses EOA `0x5387284206D648afE82240d2E303c20dc402D022`; it is not a Safe. A Safe handover is planned after audit. Immutable vault code does not remove trusted administrator powers over feeds, approved assets/providers and protocol fee settings.

Keep wallet keys and API secrets private. Never send a wallet private key to Tilt. The API cannot withdraw investor funds. There is no mainnet faucet or agent-triggered deployment of stock tokens.

## A. Manager trading

### A1. Authenticate with current vault authority

Use the vault's current curator or an active delegate. `POST /v1/auth/keys` requires `wallet_address`, `vault_address`, `chain_id: 4663`, `timestamp`, `nonce`, and `signature`. Generate a cryptographically random 32-byte nonce, encoded as `0x` plus 64 lowercase hex characters. Timestamp is Unix seconds, at or before server time and less than 300 seconds old.

Sign these exact EIP-191 UTF-8 bytes, using lowercase addresses:

The `Domain` below is a fixed protocol signature domain. Preserve it exactly even though mainnet HTTP requests use the separate `MAINNET_API_URL`; the chain ID distinguishes the environments.

```text
Tilt Protocol API authorization v2
Domain: https://api.tiltprotocol.com
Chain ID: 4663
Action: create-key
Wallet: {wallet_address_lowercase}
Vault: {vault_address_lowercase}
Permissions: trading
Timestamp: {unix_seconds}
Nonce: {nonce}

This key authorizes trading in this vault. It cannot withdraw funds.
```

Save the returned `key_id` and one-time `secret` securely. Authenticated trading requests use both `TILT-API-KEY-ID` and `TILT-API-SECRET`. Signatures are single-use. Old unbound keys must be recreated.

To revoke a mainnet key, the key creator or the vault's current curator signs these exact EIP-191 bytes. Use a lowercase signing address, the exact key ID, a fresh nonce and the same timestamp rules as creation:

```text
Tilt Protocol API authorization v2
Domain: https://api.tiltprotocol.com
Chain ID: 4663
Action: revoke-key
Wallet: {signing_wallet_lowercase}
Key: {key_id}
Timestamp: {unix_seconds}
Nonce: {nonce}
```

Send `wallet_address`, `signature`, `timestamp`, `nonce` and `chain_id: 4663` as JSON to `DELETE ${MAINNET_API_URL}/v1/auth/keys/{key_id}`. Another delegate cannot revoke the key. The original creator can revoke its key after losing vault authority. A successful response includes `success: true`; an invalid, expired or replayed proof returns HTTP403. `GET ${MAINNET_API_URL}/v1/auth/keys?wallet={wallet_address}` lists key metadata without secrets. To rotate, revoke the old key and create a replacement with a new nonce; never reuse a signed proof. The default testnet skill uses its own deployed authentication flow and must not be substituted here.

Authority is rechecked for every privileged request and immediately before settlement. Revoking a key or removing its wallet as a curator/delegate prevents subsequent execution of its resting orders. RPC or Redis failures authorize no action. A transaction already broadcast cannot be revoked by deleting a key.

### A2. Read the current portfolio and approved market

Use `GET /v1/trading/account`, `/positions`, `/orders`, and `/assets`. Only the canonical admin-approved active stock registry is tradable. Positions use the live ERC-8056 display multiplier; execution and custody use raw token units. A missing cost basis or unavailable valuation must not be treated as zero investment value. Raw basket exits remain available when NAV cannot be computed.

On mainnet, `/assets` and `/assets/:symbol` return `price: null` with `price_available: false` when the validated reference is unavailable. `status: "active"` describes registry/feed and engine-allowlist eligibility; `tradable` also requires healthy stock/base references and an unpaused engine. Execution still checks vault permissions, caps, liquidity and issuer restrictions. Never gate raw exits or claims on this discovery flag. `/account` returns HTTP503/code `50310005`, `Vault valuation temporarily unavailable`, when the basket lens reports unavailable NAV; unexpected RPC/ABI/configuration failures remain generic HTTP500.

The agent discovery routes `/api/agents/tokens`, `/tokens/:symbol` and `/stocks` return the enabled mainnet registry intersected with the engine allowlist. Their token-price fields are `priceUsd` and `priceAvailable`, with null/false when validated pricing is unavailable. Sponsored `/api/agents/register` returns HTTP409 on mainnet; use your own wallet for direct registration. Faucet and token deployment remain testnet-only.

USDG is the beta base asset. The initial equity allowlist is **AAPL, MSFT, TSLA, GOOGL and NVDA**. Query the live registry before each new intent; do not assume additional listed Robinhood assets are enabled. Execution uses approved **0x and Uniswap** providers. Initial turnover limits are **1,000 USDG per vault** and **10,000 USDG protocol-wide**, conservatively counting the current and previous UTC day's usage. These are trading limits, not deposit or loss limits; a new UTC day does not necessarily restore full capacity.

Trading is subject to the configured router's `marketOpen`, feed freshness, health status and corporate-action halt checks. It is not restricted by the legacy testnet exchange-hours calendar. A fresh feed alone does not override a closed market. Market policy still depends on verified protocol operations; do not infer that the market is open from a wall clock.

The separate beta calendar uses regular U.S. equity sessions **09:30–16:00 America/New_York**, excluding holidays and observing early closes. Published 2026/2027 calendar coverage closes unsupported years. The upcoming early closes are **13:00 New York time on November 27 and December 24, 2026**, and **November 26, 2027**. This is Tilt's narrower beta policy, not Robinhood's complete issuer trading schedule. Current chain flags and validated prices still govern fills; mainnet DAY expiry remains UTC-based.

When configured, Tilt's operational-health gate uses expiring operator observations and an administrator-controlled recovery period. It is not a Chainlink sequencer-uptime oracle and does not prove uninterrupted chain access. Error `42210012` means pricing health is unavailable or recovering. Do not bypass the gate or try to reopen it as a manager. Raw basket exits and reserved-token claims remain independent of this gate; an unavailable performance valuation is waived on exit. Optional cash conversion still depends on executable liquidity and investor amount bounds.

### A3. Submit an order

`POST /v1/trading/orders` accepts:

| Field | Meaning |
|---|---|
| `symbol` | Active approved ticker |
| `side` | `buy` or `sell` |
| `type` | `market` or `limit` |
| `qty` or `notional` | Exactly one positive decimal string: display shares or base-currency amount |
| `limit_price` | Positive base currency per display share; required for limits |
| `time_in_force` | Limits support `day`, `gtc`, `gtd`; IOC/FOK are market-only compatibility values |
| `expires_at` | Valid future ISO-8601 timestamp required for `gtd` |
| `client_order_id` | Unique stable ID for this intended order; preserve on retries |
| `sell_entire_balance` | Sell only; omit notional and omit qty or use `"0"`; sells available holdings excluding reserved claims |
| `slippage_bps` | Optional mainnet quote tolerance, capped at 50 basis points |

Mainnet `day` orders expire at the end of their UTC creation date. GTC persists until filled or canceled; GTD expires at its explicit timestamp. The keeper polls at a default 30 seconds and schedules bounded batches fairly across vaults. Each vault may hold at most 1,000 live orders; capacity rejection is `42210021`.

A limit price controls both the price trigger and the on-chain minimum output at full integer precision. An exact-output quote whose worst-case input breaches the limit remains unfilled. Quotes route through approved settlement targets and registered spenders; the manager cannot direct assets to an arbitrary recipient.

### A4. Recover without duplicate orders

HTTP **202** with `status: "pending_new"` means the order is in flight or needs reconciliation. It may already have executed even if `tx_hash` is missing. Keep the existing `client_order_id`, poll `GET /v1/trading/orders/{id}`, and inspect the confirmed chain result. **Never create a replacement under a new ID while the outcome is unknown.**

A durable broadcast-intent marker precedes sending. Expiring a lease is not proof that no transaction was sent. Known hashes reconcile from receipts; a missing receipt remains pending. A broadcast intent without a recoverable hash requires operator reconciliation and is never automatically retried. Confirmed fills use actual receipt amounts. Missing/unparseable fill amounts remain pending rather than being fabricated from quotes.

`DELETE /v1/trading/orders/{id}` cancels only a resting order. HTTP409 means settlement is already in flight. Cancellation and expiry cannot override a broadcast.

## B. Create an immutable vault

Read the configured `basketVaultFactory` and its ABI from the address book, verify deployment code and that `creationPaused()` is false, then read `baseAsset()` and `newVaultConfigHash()` together. A closed factory is not permission to bypass the gate or create a replacement deployment. Read token decimals; approve only the reviewed factory for the intended seed amount. Call:

```text
createVault(name,symbol,managementBps,performanceBps,seedAmount,metadataURI,expectedConfig)
```

The expected configuration hash binds the base asset, execution engine, price router, default delegate and administrator. If configuration changes before inclusion, the call reverts; review the new configuration before trying again. Vault management/performance fee rates and code are immutable. Each vault owns its assets. Manager/delegate authority allows approved trading, not withdrawing investors' holdings.

Entry/exit fees start at zero, with administrator-controlled caps of 1% and 2%; changes can be immediate. Tilt's manager-fee share initially is 10% and can change globally or per vault. Entry/exit fees go to Tilt. Check the current terms before approving or signing. A new base asset applies to new vaults; moving an existing investment requires explicit investor migration.

## C. Investor deposits and exits

### C1. Raw basket operations

Read `previewMintBasket(shares)` or `previewRedeemBasket(shares)` for the basket hash, token list, raw amounts and fee shares. Read every token's decimals. Review quantity bounds, fee ceiling and deadline, then sign `mintBasket` or `redeemBasket` using the address book's current ABI. Basket changes or bounds breached before execution cause a revert.

A raw redemption remains callable during pause, market closure or unavailable prices. A token that cannot transfer becomes a proportional claim; inspect `unclaimedEmergency(account,token)` and use `claim(token,receiver,minAmount)` after transfers are possible. Do not assume every entitlement was delivered immediately. Uncomputable performance fees are waived on that exit; management fees still accrue.

### C2. Optional base-currency conversion

`POST /v1/vaults/{vault}/basket-quote` is public and read-only, rate-limited to ten requests per minute per IP. It creates no authorization and broadcasts nothing. Send:

- `mode`: `deposit` or `redeem`.
- `caller`: the investor wallet that will sign.
- `shares`: positive integer string in raw vault-share units.
- `maxBaseIn`: required raw cash budget for deposit; or `minBaseOut`: required raw cash floor for redemption.
- `maxFeeBps`: accepted entry or exit fee ceiling; optional `slippageBps` between 0 and 50.

The response contains `approval`, `transaction`, `expiresAt` in Unix seconds, `basketHash`, `tokenAmounts`, `tokenAmountsKind`, `previewTokenAmounts`, `estimatedBaseAmount`, fee data and `residualTokensPossible`. It expires within sixty seconds. Independently verify the chain, vault, locally configured router, token/spender, shares, cash bound, fee ceiling and encoded deadline before signing. Use only the needed allowance. The endpoint never needs a wallet secret.

Deposits buy the existing proportional basket before minting net shares. Cash redemption sells conservative token minima (10 basis points below preview entitlements) and **returns any unsold tokens**. It is not a guaranteed all-cash exit. Required venue liquidity or token restrictions may make cash conversion fail; raw basket redemption remains the fallback.

## D. Unsupported legacy administration

The agent name, description, pause, unpause and cost-basis backfill endpoints return HTTP409 on mainnet. Trading keys do not authorize protocol administration, and legacy historical replay cannot safely overwrite multiplier-tagged costs. Authorized administration uses the current on-chain interface directly. Unknown historical cost basis needs operator reconciliation.

The current manager API deployment supports its explicitly configured base asset, execution engine and price router. A future vault using a different configuration returns `42210017` until that generation is supported; the API never silently applies USDG sizing to another base currency. Investor basket quotes read the base asset from the vault.
