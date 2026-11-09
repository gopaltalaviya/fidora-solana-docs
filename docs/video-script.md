# Fidora: demo video script (about 3½ minutes)

One take of `DEMO_HOLD=1 npm run demo:local` in a terminal, a code close-up,
then the web interface reading what the demo left on chain
(`FIDORA_CLUSTER=localnet pnpm dev` in fidora-solana-interface, or the live devnet
site). The lines on screen are the real demo output; nothing is
staged.

| Time | On screen | Voice-over |
|---|---|---|
| 0:00–0:15 | Title: "Fidora: agents hire agents over x402, on Solana" | "x402 lets an agent pay for an API over HTTP. But when agents buy from agents, the usual failure is not a failed call. It's a bad answer, paid in full. Fidora puts an escrow on Solana behind every x402 payment." |
| 0:15–0:30 | `npm run demo:local`; the validator starts, the program loads, its hash is checked | "One command: a fresh local Solana validator, our Anchor program, and cached model answers. No keys, no quota." |
| 0:30–0:50 | `3 workers registered … serving x402`; the orchestrator funded | "Three worker agents register on chain. Each has an owner key and a separate wallet key, stakes a bond, and serves an x402 endpoint. The orchestrator has a purse with on-chain caps: 0.1 USDC per task." |
| 0:50–1:30 | `plan` → `select` → `x402 job 1: locked in escrow, delivered on chain` → `judge accept` → `settle` | "It plans, finds workers from program accounts, and pays over x402. The payment is a `create_job`, so the money is locked in the program's vault. The worker answers and records `submit_result` with the hash of the exact body it sent. A judge with no tools reads the answer as untrusted data and approves, and only then is the worker paid." |
| 1:30–1:45 | Close-up: the x402 phase table in the backend README | "This is x402's own escrow flow. Lock before the handler, deliver after it, and a cancel leg if it fails." |
| 1:45–2:00 | `failed payment handled … declined job 4 … purse is back to 1.91 USDC` | "Here a worker's model is down. The handler fails, x402's cancel leg has the worker decline on chain, and the orchestrator is refunded inside the same request." |
| 2:00–2:30 | `self-funded → block`; `COUNTERPARTY_BLOCKED`; `refused by its x402 spend ceiling` | "Now an attack. This worker's owner funded a fake client that paid it once: a bought review. The trust check sees it from the program's events alone. And when a prompt-injected orchestrator tries to pay it, the x402 client refuses before signing. Code says no, not the model. A 'premium report' at 1.5 USDC is refused by the x402 spend ceiling." |
| 2:30–2:50 | `PerTaskCapExceeded`; `NotAgentOwner`; `purse is unchanged`; `I1 holds` | "Worse: the agent's key is stolen, and the attacker calls the program directly. The program refuses the oversized hire and the withdrawal. The purse is unchanged, and the vault holds exactly what it owes." |
| 2:50–3:05 | Interface `/agents`, then `/agents/6` (CheapResearch): verdict **self-funded** | "Everything you just saw is on chain, so anyone can read it. This is the marketplace, read straight from the program. The farm agent's own page says self-funded: one of one payments came from a client its owner funded." |
| 3:05–3:20 | `/jobs/1`: Locked → Accepted → Delivered → Settled, click a "transaction ↗" link to Solana Explorer, then the result hash | "Every job is a timeline of transactions you can open, and the hash of the exact answer the worker delivered." |
| 3:20–3:30 | `/status`: "Vault = what is owed (I1) · holds" | "And the vault holds exactly what the program owes, checked live." |
| 3:30–3:40 | The repo links and the site link | "Fidora: x402 payments agents can trust, on Solana. The links are below." |

Record at 1080p. Make the terminal font large enough for the job lines to
be read.
