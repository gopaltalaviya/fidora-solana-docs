# Fidora: status, 2026-11-07

Every "done" line was checked live on a local `solana-test-validator`
(Agave 3.0.10), not against a mock.

| Item | State | Evidence |
|---|---|---|
| Anchor program, ported from Pactora/NEAR, plus `decline_job` | **done** | `verify-solana`: 23/23 live. Each new check was red on the old program first |
| Rolling-cap unit tests | **done** | 7/7: six ported from NEAR, plus a Clock step back (red on the old code: it panicked) |
| IDL without the Anchor CLI | **done** | `scripts/build-idl.mjs`, the same steps as `anchor-lang-idl` 0.1.4; 36 instructions |
| `fidora-escrow` x402 scheme on the official x402 v2 packages | **done** | `scripts/x402-smoke.ts` live: 402 quote, lock, deliver with `sha256(body)`, approve, and a failure refunded via decline |
| Unit tests | **done** | 46: facilitator tamper refusals, the pay and judge gates, trust completeness, the chain layer. Mutation-checked: removing the amount check, or the foreign-instruction check, turns a test red |
| Hostile workers, live (`scripts/x402-attacks.ts`) | **done** | 6/6: lying hash, forged job id, swapped agent, delivery failing after the lock, plus the honest baseline. All four attacks were red on the old orchestrator — one locked money with the wrong agent |
| One command for every live check | **done** | `npm run test:live` → `All live checks passed.` |
| Agents: plan, discover on chain, trust gate, select, x402 pay, judge, approve or dispute | **done** | `npm run demo:local`: `Demo passed.` from a fresh chain with cached models |
| Failed-payment handling | **done** | The demo: worker model down → `decline_job` → purse restored |
| Attack: farm detection, injected orchestrator, stolen key | **done** | The demo: `COUNTERPARTY_BLOCKED`, spend ceiling, `PerTaskCapExceeded`, `NotAgentOwner`, purses unchanged, I1 |
| Devnet deployment, and `exact` on devnet | **waiting** | Needs devnet SOL in the owner's own wallet, named by `FIDORA_DEPLOYER_KEYPAIR`. Test keys are saved before they are funded and swept back to it (`scripts/sweep-check.ts`, live) |
| Aurora Intents | **not started** | Needs an Aurora Client Portal API key and mainnet funds (Aurora has no testnet) |
| Web interface | **done** (read-only) | [fidora-solana-interface](https://github.com/gopaltalaviya/fidora-solana-interface): marketplace, agent profiles with the trust verdict, escrow job timelines, activity feed, status with invariant I1 — all read from the chain by its own `/api/v1` routes. 110 unit + 60 browser tests. `DEMO_HOLD=1 npm run demo:local`, then `FIDORA_CLUSTER=localnet pnpm dev` |
| GitHub repos | **done** | [fidora-solana-programs](https://github.com/gopaltalaviya/fidora-solana-programs), [fidora-solana-backend](https://github.com/gopaltalaviya/fidora-solana-backend), [fidora-solana-docs](https://github.com/gopaltalaviya/fidora-solana-docs) |

Found and fixed along the way:
- **Confirming a decoded transaction.** A transaction decoded from base64
  (as every x402 payload is) has no `lastValidBlockHeight`, and confirming it
  by block window falsely reported landed payments as expired. Confirmation
  now polls the signature's status.
- **Additive enrichment.** x402 requires settlement enrichment to be
  additive, so each phase has its own payload key.
- **Platform tools.** The ones bundled with Agave 3.0 cannot build current
  crates; `--tools-version v1.58` is used.
