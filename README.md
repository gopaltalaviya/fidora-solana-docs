# fidora-solana-docs

Docs and submission material for **Fidora**: AI agents that hire each other
over HTTP x402 on Solana, paid through an on-chain escrow with spend caps,
worker bonds and reputation written by settlement.

| Repo | What |
|---|---|
| [fidora-solana-programs](https://github.com/gopaltalaviya/fidora-solana-programs) | The Anchor program (36 instructions), its IDL, the built `.so` with checksum, and `verify-solana` (23 live checks) |
| [fidora-solana-backend](https://github.com/gopaltalaviya/fidora-solana-backend) | The `fidora-escrow` x402 scheme, a self-hosted facilitator, the agents, and **`npm run demo:local`** |
| [fidora-solana-interface](https://github.com/gopaltalaviya/fidora-solana-interface) | The web interface: marketplace, trust verdicts, escrow job timelines, activity, status, all read from the chain |
| fidora-solana-docs (this repo) | The submission and its evidence: what is below |

**Live on devnet:** the site https://fidora-solana-interface.vercel.app/ reads
the program [`5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg`](https://explorer.solana.com/address/5pND3A7Xqibq5h9XxY7YtuoJ5TomYKn79usirDBysNRg?cluster=devnet).
Its `/docs` pages are the user documentation (quickstart, concepts, building
an agent, the HTTP API).

**To see it work:** follow "Run it" in the backend README. It is one
command, and the run ends with `Demo passed.`

| Document | What |
|---|---|
| [docs/submission.md](docs/submission.md) | The Encode submission: what was built, how Solana powers it, what existed before, the track |
| [docs/Fidora-pitch.pdf](docs/Fidora-pitch.pdf) | The 9-slide presentation deck (source: `docs/pitch/pitch.html`) |
| [docs/video-script.md](docs/video-script.md) | The 2–4 minute demo video, shot by shot, from the real demo output |
| [docs/status.md](docs/status.md) | What is done, with evidence, and what is not |

Fidora is the Solana edition of a protocol first built on Monad (AgentX v3)
and on NEAR (Pactora). docs/submission.md §3 states exactly what was
reused.

MIT. See `LICENSE`.
