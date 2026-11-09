# Fidora — Encode Solana Hackathon 2026 submission

Track: **Best Use of x402 on Solana**.

The Aurora Intents track is not entered: its rules require a live mainnet
integration, which needs an Aurora API key and mainnet funds we do not have
(see §5).

## 1. What it is (description for the form)

Fidora lets AI agents hire each other over HTTP x402 on Solana, without
trusting each other.

- A paid call locks the client's USDC in an on-chain escrow under the
  client's spend caps.
- The worker's delivery is recorded on chain with the hash of the exact
  response body.
- A tool-less judge then approves (the worker is paid) or disputes (an
  arbiter rules). A worker that cannot deliver declines inside the same
  request, and the client is refunded in full.
- Reputation is written only by settlement.
- A trust check reads the program's own events to spot agents that funded
  their own clients, and the x402 client refuses to pay them before
  signing.

The demo runs end to end on a local validator with one command.

## 2. How Solana powers it

- **The escrow is a Solana program** (Anchor 1.2.2): a vault PDA holds every
  token. `create_job`, `accept_job`, `submit_result`, `approve`, `dispute`
  and `decline_job` are its instructions. Invariant I1 (vault balance equals
  what is owed) is checked live.
- **x402 settles on Solana.** Each x402 phase is a signed Solana transaction
  that our facilitator decodes against the IDL, simulates, and relays:
  - lock = `create_job`
  - deliver = `accept_job` + `submit_result(sha256(body))`
  - cancel = `decline_job`
- **Keys and caps on chain.** An agent's wallet keypair can only sign job
  instructions, within per-task and rolling 24 h caps. Only the owner's key
  can withdraw. A stolen agent key's worst case is its caps, whether it
  spends them as payments or as the worker bonds it locks.
- **Discovery and trust from chain state.** Agents are program accounts
  whose `card_uri` points to their x402 endpoint. The trust verdict is
  computed from `Funded` and settlement events, with no database.
- **Network id from the chain.** The CAIP-2 id comes from the node's
  genesis hash, so a local validator is never labelled devnet.

## 3. What existed before the hackathon, and what was built in it

**Existed:**
- The protocol design (escrow states, caps, bonds, the trust rules), by the
  author.
- Agent code written by the author before the hackathon, reused unchanged in
  fidora-solana-backend:
  - `src/models/`: the model providers, cached replay, the tool-less judge,
    and the prompts
  - the trust-verdict rules
  - the worker output schemas

**Built during the hackathon:**
- The Anchor program: the protocol's rules on Solana, with PDAs and an SPL
  vault, plus the new `decline_job`. Live-verified with 23 checks.
- The IDL build without the Anchor CLI.
- The `fidora-escrow` x402 scheme (client, resource server, facilitator) on
  the official x402 v2 packages, using x402's own `escrow` payment flow.
- The self-hosted facilitator.
- The x402 worker and orchestrator.
- The on-chain trust read.
- The demo and the one-command local run.
- The unit tests (52 backend, 7 program), mutation-checked, and a live
  suite of hostile workers (`scripts/x402-attacks.ts`).

Checkpoints 1 and 2 were missed: this is a late entry.

## 4. Why x402 needs an escrow (the use case)

x402's standard `exact` scheme settles after the handler succeeds. A failed
call costs nothing, but a delivered-but-wrong answer is paid in full, and
there is no recourse.

For agents buying from agents, that is the common failure: the answer
arrives, it is bad, and nobody checks before paying. Fidora keeps the money
in escrow until the buyer's judge accepts. It backs delivery with the
worker's bond, and bounds every buyer by on-chain caps.

The demo shows each case:
- delivered and approved
- the model down, refunded in-request
- a farmed counterparty, refused before signing
- an oversized purchase, refused by the x402 spend ceiling
- a stolen key, refused by the program

## 5. Honest limits

- **Devnet: deployed, checked, the demo partly run.** The program is live on
  devnet and `verify-solana` passes 21 checks there (the two that wait out
  10-minute and 1-hour windows run on localnet). The agent demo on devnet
  paid two workers through x402 escrow, then the laptop running the model
  ran out of memory; the full demo passes on localnet. The public devnet RPC
  rate-limits hard; the code waits it out, so a run there is slow. The
  standard `exact` scheme (devnet only, as `@x402/svm` accepts only
  mainnet, devnet and testnet ids) has not been run on devnet yet.
- **Aurora Intents is not integrated.** Aurora has no testnet, and the
  integration needs a Client Portal API key and organisation approval.
- **The web interface is read-only.** It shows agents, trust verdicts, escrow jobs and the event feed from the chain; registering and hiring are done by agents with their own keys (the terminal demo).
- **Agent wallet keypairs live in the agent process.** Keep their SOL to fee
  money.
- **The model recordings are a local llama3 8B.** Figures in its answers are
  model output, not market data.

Found by an adversarial review on 2026-11-07, and NOT fixed (the fixed ones
are in docs/status.md):
- **A client at its full daily cap cannot dispute.** The dispute bond is
  charged to the client's caps like any spend. A client that hired for its
  whole remaining day cannot pay it, and the job auto-approves after the
  review window unless the owner raises the caps in time. Part of the
  protocol design.
- **`initialize` is not tied to the upgrade authority.** Someone watching a
  fresh deployment could initialize it first; `deploy.mjs` refuses an
  initialized program, so the cost is a redeploy, not funds.
- **`direct_pay` writes a review with no action from the worker.** Two owners
  can trade cheap direct payments to build a record. The trust check is the
  defence (`concentrated`, `self-funded`), not the program.
- **`spec_hash` commits to a random nonce, not the request body**, so an
  arbiter cannot tie a dispute to what was asked from the chain alone.
- **The self-hosted facilitator has no authentication** (each call costs it
  one simulation, and it holds no keys or funds), and discovery fetches any
  `card_uri` a registered agent sets.
- **Two clients quoting in the same slot get the same job id**; the second
  lock fails, so that hire is reported failed and nothing is paid.

## 6. Links (fill in at submission)

- Repos (clone side by side):
  - Programs: https://github.com/gopaltalaviya/fidora-solana-programs
  - Backend: https://github.com/gopaltalaviya/fidora-solana-backend
  - Docs: https://github.com/gopaltalaviya/fidora-solana-docs
  - Interface: https://github.com/gopaltalaviya/fidora-solana-interface
- Screenshots (the site reading the chain a `demo:local` run left, 1920×1080):
  [home](images/01-home.png), [marketplace](images/02-agents.png),
  [the farm agent: self-funded → block](images/03-agent-farm.png),
  [an escrow job's timeline](images/04-job-timeline.png),
  [activity](images/05-activity.png), [status: vault = owed](images/06-status.png).
- Live site, reading devnet: https://fidora-solana-interface.vercel.app/
  (status shows the vault = what the program owes, live).
- Video (3:46, captioned): https://www.youtube.com/watch?v=yfxOOvaoiOo
- Presentation deck (9 slides): [docs/Fidora-pitch.pdf](Fidora-pitch.pdf)
  (source `docs/pitch/pitch.html`; https://github.com/gopaltalaviya/fidora-solana-docs/blob/master/docs/Fidora-pitch.pdf)
- Devnet program: [`5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg`](https://explorer.solana.com/address/5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg?cluster=devnet).
