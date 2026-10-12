# Repository directory

[Back to the project overview](../README.md)

The organization separates the on-chain protocol, trading interface, off-chain verification, onboarding, and documentation. Use this page to choose where to start reading or contributing.

## Trading system

### `settlement-engine` — protocol rules and asset movement

[Repository](https://github.com/unleaktrade/settlement-engine) · [README](https://github.com/unleaktrade/settlement-engine/blob/main/README.md)

The Rust/Anchor program is the authority for accepted actions, deadlines, account ownership, token transfers, and economic outcomes. It defines the RFQ and Quote accounts, verifies the Liquidity Guard's attestation, selects a counterparty, escrows the maker's base tokens, and completes settlement.

Start with:

- [Program entry points](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/lib.rs).
- [Account definitions](https://github.com/unleaktrade/settlement-engine/tree/main/programs/settlement-engine/src/state), especially `rfq.rs` for lifecycle states and deadline calculations.
- [Instruction handlers](https://github.com/unleaktrade/settlement-engine/tree/main/programs/settlement-engine/src/instructions) for the rules behind each user action.
- [Integration and unit tests](https://github.com/unleaktrade/settlement-engine/tree/main/tests), including expired, ignored, and incomplete RFQs.

**Dependencies:** Solana, Anchor, SPL Token/ATA programs, and a correctly configured Liquidity Guard signing key. **Consumed by:** `app`; its commitment format must agree with `liquidity-guard`.

### `liquidity-guard` — signed checks before a quote enters the auction

[Repository](https://github.com/unleaktrade/liquidity-guard) · [Service implementation](https://github.com/unleaktrade/liquidity-guard/blob/main/src/main.rs)

An off-chain Rust/Actix Web service reads balances through Solana RPC, checks the bond and quote-plus-fee requirements, constructs the commitment, and signs it with Ed25519. The service exposes `/check`, `/health`, `/ready`, and `/metrics`.

Its signature authorizes a commitment; it neither selects winners nor moves trading assets. It sees the request's quote details and remains a trust assumption of the current design.

Start with the [README](https://github.com/unleaktrade/liquidity-guard/blob/main/README.md) for the request schema, 178-byte hash preimage, configuration, API authentication, and tests. The [Postman collection](https://github.com/unleaktrade/liquidity-guard/tree/main/postman) exercises its HTTP interface.

**Dependencies:** Solana RPC, the configured bond-token mint, and a service signing key. **Consumed by:** `app` and protocol integration tests; attestations are enforced by `settlement-engine`.

### `app` — the trader's workspace

[Repository](https://github.com/unleaktrade/app) · [Live beta](https://app.unleak.trade) · [README](https://github.com/unleaktrade/app/blob/main/README.md)

The React/TypeScript trading application reads program accounts directly over RPC and WebSocket subscriptions. A connected wallet signs messages and transactions. The UI covers the marketplace, RFQ creation, quote commitments and reveals, selection, settlement, bond recovery, rewards, and transparency views.

Start with:

- [`src/chain`](https://github.com/unleaktrade/app/tree/main/src/chain): decoders, PDAs, deadline/fee math, transaction builders, commitment hashing, and the Guard client.
- [`src/app`](https://github.com/unleaktrade/app/tree/main/src/app): screens, providers, components, and user flows.
- [Reveal tickets](https://github.com/unleaktrade/app/blob/main/src/app/lib/reveal-ticket.ts): local persistence and downloadable recovery material for sealed quotes.
- [End-to-end guide](https://github.com/unleaktrade/app/blob/main/e2e/README.md): recorded-RPC UI checks and the separate live transaction suite.

**Dependencies:** `settlement-engine`, `liquidity-guard`, RPC, and a compatible wallet. The app uses Jupiter for mainnet token/price metadata, with static catalogs for devnet/localnet. **Trades settle through UnleakTrade's program**, not through a Jupiter swap route. The deployed interface is served through GitHub Pages; internal PR previews use Vercel.

## Discovery, onboarding, and documentation

### `landing-page` — explain the product and request access

[Repository](https://github.com/unleaktrade/landing-page) · [Website](https://unleak.trade)

The public React/TypeScript website contains product sections, FAQ, roadmap, builder information, and waitlist registration/activation screens. It calls the `waitlist` API. It is separate from the trading application and does not execute the RFQ lifecycle.

Useful entry points: [FAQ source](https://github.com/unleaktrade/landing-page/blob/main/src/components/FAQ.tsx), [waitlist screen](https://github.com/unleaktrade/landing-page/blob/main/src/components/WaitlistPage.tsx), and [activation screen](https://github.com/unleaktrade/landing-page/blob/main/src/components/ActivateWaitlist.tsx).

### `waitlist` — activate participants and distribute test tokens

[Repository](https://github.com/unleaktrade/waitlist) · [README](https://github.com/unleaktrade/waitlist/blob/main/README.md)

A Go service handles registration, email activation, user data, and asynchronous distribution of 1,000 uUSDC per activated wallet. DynamoDB stores registration and distribution state. The worker transfers existing devnet tokens from a funded distribution account; it does not mint new uUSDC during activation.

Useful entry points: [API](https://github.com/unleaktrade/waitlist/tree/main/cmd/api), [mailer](https://github.com/unleaktrade/waitlist/tree/main/internal/mailer), and [airdrop worker](https://github.com/unleaktrade/waitlist/tree/main/internal/airdrop). The service was adapted from `fairhive-labs/preregister`, as its README records.

**Boundary:** waitlist activation supplies beta onboarding and test tokens. It is not the on-chain quote verifier or settlement engine. Distribution can remain pending after activation succeeds.

### `docs` — public product and protocol documentation

[Repository](https://github.com/unleaktrade/docs) · [Published GitBook](https://unleaktrade.gitbook.io/unleaktrade-docs)

GitBook source for the mental model, lifecycle, participant perspectives, costs, and illustrated app instructions. [SUMMARY.md](https://github.com/unleaktrade/docs/blob/main/SUMMARY.md) is the navigation index; [user-guide](https://github.com/unleaktrade/docs/tree/main/user-guide) is the best starting point for a manual demo.

This Colosseum hub summarizes and connects those materials. Detailed product documentation continues to live in `docs`.

## Engineering support and experiments

| Repository | Purpose | Relationship to the product |
| --- | --- | --- |
| [actions](https://github.com/unleaktrade/actions) | Reusable GitHub Actions and workflows, including Solana CLI installation and Ubuntu runner setup | Development and CI infrastructure; not a runtime service |
| [generate_keypair](https://github.com/unleaktrade/generate_keypair) | Rust CLI for generating Solana Ed25519 keypairs and mnemonic-based keys | Developer utility; not required for judges using a normal wallet. Its output contains private key material and should stay local. |
| [experimental-preflight-sigcheck](https://github.com/unleaktrade/experimental-preflight-sigcheck) | Exploratory off-chain signing / on-chain verification proof of concept | Research reference; production protocol behavior is defined in `settlement-engine` |
| [colosseum-2026](https://github.com/unleaktrade/colosseum-2026) | Project overview, evaluation guide, repository map, and source snapshot | Documentation hub; no separate trading deployment |

## Suggested code-review order

1. Read the [protocol lifecycle](protocol.md) to understand the intended flow.
2. Inspect `settlement-engine`'s RFQ state and instruction handlers.
3. Compare the commit hash and fee calculations across the engine, Guard, and app.
4. Read the app's instruction builders and reveal-ticket flow.
5. Inspect success and failure tests, then review onboarding if testing the public beta.

The [source snapshot](sources.md) records the exact revisions used when preparing this guide.
