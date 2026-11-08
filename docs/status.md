# Fidora: status, 2026-11-08 (final check 23:20 UTC)

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
| Devnet deployment | **done** (2026-11-08) | Program `5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg` (a new program keypair: the one for the old id `VkxrZbg…` was not found, so `declare_id!`, `fidora.so` and the IDL were rebuilt for the new id), deployed and initialized by the owner's wallet `2m9UHYVSk6V7ErSycjhQdev2RqjGymYsgx2aozhC19eG` (upgrade authority, owner and arbiter). `verify-solana --cluster devnet`: 21 checks passed on devnet; see below |
| Aurora Intents | **not started** | Needs an Aurora Client Portal API key and mainnet funds (Aurora has no testnet) |
| Web interface | **done** (read-only) | [fidora-solana-interface](https://github.com/gopaltalaviya/fidora-solana-interface): marketplace, agent profiles with the trust verdict, escrow job timelines, activity feed, status with invariant I1 — all read from the chain by its own `/api/v1` routes. 116 unit + 61 browser tests (×5 browsers with `E2E_ALL_BROWSERS=1`). `DEMO_HOLD=1 npm run demo:local`, then `FIDORA_CLUSTER=localnet pnpm dev` |
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
- **`/jobs` was a dead link** (2026-11-07). Quickstart, Guides and Build an
  agent linked to `/jobs`; only `/jobs/<id>` exists. `/jobs` now redirects to
  the activity feed, which lists the jobs (`next.config.mjs`); a browser test
  covers it, red before the fix.

## Next up, and what waits on the owner (2026-11-08)

Done and tested: the program on devnet (its bytes = a rebuild of this
source, sha256 `62ade182…`), the live site
https://fidora-solana-interface.vercel.app/ (21/21 pages in a browser, on
devnet), the four repos pushed and passing from fresh clones, the
screenshots, and the video (`fidora-demo.webm`, 3:46, kept outside git; https://www.youtube.com/watch?v=yfxOOvaoiOo).
Nothing is uncommitted. Git state and gates for the interface are in the final check below.

Waits on the owner, before the deadline (Mon 9 Nov 2026, 11:59 UTC):
1. Make the four repos public (they answer 404 until then).
2. ~~Upload the video~~: done, https://www.youtube.com/watch?v=yfxOOvaoiOo (in `submission.md` §6).
3. Submit on Encode from `submission.md`.
4. Optional: a private devnet RPC key on Vercel (`FIDORA_RPC_URL`), so the
   activity and trust pages are complete on the first load.

### Final check, 2026-11-08 22:30–23:20 UTC (live URL and localhost)

| Check | Result |
|---|---|
| Live site, 24 pages (19 sitemap + agent and job pages) in headless Chromium | 24/24: 200, no console error, no failed request, no loading placeholder left; 86/86 internal links answer |
| Live `/api/v1/*` | status operational on devnet, I1 holds (329.28 = 329.28 USDC), 42 agents, 20 jobs; an independent read of the devnet RPC gives the same vault balance and account counts |
| Program on devnet | executable, upgrade authority the owner's wallet, `solana program dump` sha256 `62ade182…` = `artifacts/fidora.so`; `verify-solana --cluster devnet` 21/21 again |
| Backend | `check` 52/52, `test:live` all passed, `demo:local` `Demo passed.` |
| Site on the held local chain, production build | 23/23 pages, 50 events complete, farm agent self-funded → block, I1 holds |
| Interface from a fresh clone | typecheck, lint, format clean; e2e 303 passed in 5 browsers; unit 116/116 after the fix below |
| Program unit tests | 7/7 |
| Repos, full history | no keys, no personal data, no local paths; only `.env.example` with empty values |

Found and fixed:
- `NEXT_PUBLIC_SITE_URL` was not set on Vercel, so the live sitemap, robots
  and `og:image` pointed at `localhost:3000`. Set by the owner and redeployed;
  all 19 sitemap URLs are now on the real domain.
- One flaky unit test (`patient-fetch.test.ts`, failed ~1 run in 6): it
  ignored the retry's random jitter. The test now pins `Math.random`
  (interface e0334ee); 12/12 runs green.
- Cosmetic, left as is: the footer says "Solana devnet" on a localnet run too.

To resume: `npm run test:live` and `npm run demo:local` in
fidora-solana-backend; `corepack pnpm check` and `pnpm test:e2e` in
fidora-solana-interface; `node --env-file=.env scripts/verify-solana.mjs
--cluster devnet` in fidora-solana-programs (the owner's wallet in `.env`).

## Devnet, 2026-11-08

| | |
|---|---|
| Program | [`5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg`](https://explorer.solana.com/address/5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg?cluster=devnet), upgrade authority `2m9UHYV…` (the owner's wallet) |
| Config / vault / test USDC | `4XzNRdtZ…` / `2jc5eNfZ…` / `34rRkNq7…` (`fidora-solana-programs/deployments/devnet.json`) |
| Rebuild for the new id | 7/7 unit tests; the IDL rebuilt from source is byte-identical to the committed one with the id swapped; `fidora.so` sha256 `62ade182…` |
| `verify-solana --cluster devnet` | 21 checks passed on devnet, each with its explorer link (`auto_approve` and `expire_dispute` wait out 10 min / 1 h windows: localnet only, 23/23) |
| Agent demo on devnet (`DEMO_CLUSTER=devnet`, local Ollama) | Workers registered and staked, the orchestrator funded, the plan made, workers discovered on chain, and two x402 jobs (18, 19) locked in escrow and delivered on chain. The run was then stopped by the host for low memory (a 16 GB laptop running the model), not by a failure. The full demo passes on localnet. With `AGENT_MODE=cached` it cannot run on devnet: the recordings are keyed on a fresh chain's agent ids, and devnet's keep growing |
| Local gate on the new build | backend 52/52, `test:live` all passed, `demo:local` passed, interface 115/115, browsers 303 passed (5 browsers) |

Found while recording the screenshots and fixed: `solana-test-validator`
keeps ~10,000 shreds by default, so a few minutes into `DEMO_HOLD=1` the
program's history was purged and every interface page showed an empty
record (no events, "too little history"). `local-up.ts` now passes
`--limit-ledger-size 50000000`; the pages then show the real record (the farm
agent self-funded → block, job 1's four transactions).

Found on devnet and fixed: the public RPC answers bursts with 429, and
refuses any `getTransaction` batch of 10 or more outright (measured: 5
pass; 10 → 7 of 10 refused; 20 and 50 → all refused), sometimes inside an
HTTP 200. The scripts, the backend and the interface now wait those out and
retry a dropped connection (`patientFetch`, 8 tests, red without the
retry), and read the event history 5 transactions per batch on devnet.

The site on devnet, browser-tested (production build, `FIDORA_CLUSTER=devnet`,
the public RPC), 2026-11-08: the event scan of 184 program transactions did
not answer in 10 minutes. Measured: the public RPC lets ~10 `getTransaction`
calls through, then refuses every call, single or batched, for a while. A
serverless request would time out, taking `/activity`, trust verdicts and job
timelines with it. Fixed: on devnet the scan reads one transaction at a time
within `FIDORA_SCAN_BUDGET_MS` (20 s), keeps what it read for the next
request, and answers incomplete in the meantime (trust readings then block:
no agent is cleared on partial evidence); the RPC retry is bounded to 8 s per
call. After: every call answered in 7–24 s and the history filled in over
calls (10 → 27 → 39 → 50 events); all 21 pages of the sitemap load against
devnet with no console error. On localnet: 50 events, complete, the farm agent
self-funded → block, 21/21 pages. A private devnet RPC (`FIDORA_RPC_URL`, a
free Helius / Alchemy / QuickNode key) reads the history at once.

## Re-verified 2026-11-07 on a fresh checkout (Windows 11 + WSL Ubuntu, Agave 4.0.0)

| Check | Result |
|---|---|
| Program rebuilt from `git archive HEAD` (`cargo build-sbf --tools-version v1.58`) | sha256 `d86e727b…` = `artifacts/SHA256SUMS`: the committed `fidora.so` is this source |
| Program unit tests | 7/7 |
| Backend `npm run check` | typecheck clean, 46/46 |
| Backend `npm run test:live` | `All live checks passed.` (verify-solana 23, x402 smoke, attacks 6/6, sweep check) |
| Backend `npm run demo:local` | `Demo passed.` |
| Interface `pnpm check` | typecheck, lint, format clean; 110/110 |
| Interface browser tests, all 5 browsers (`E2E_ALL_BROWSERS=1 pnpm test:e2e`) | 303 passed, 2 skipped |
| Interface production build on the live local chain | every page 200, every `/api/v1` route returns chain data, I1 holds (vault = owed), no broken internal link |

On Windows: `npm ci` refuses the backend's lockfile under npm 10 (written by
npm 11). `npm install` works; it rewrites the lockfile (drops npm 11's
`libc` fields), so `git checkout -- package-lock.json` afterwards.
