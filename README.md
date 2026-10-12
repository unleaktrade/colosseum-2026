# UnleakTrade · Colosseum 2026

**Competitive OTC trading on Solana, with quotes sealed until reveal and atomic on-chain settlement.**

UnleakTrade helps funds, DAOs, token treasuries, large token holders, and market makers negotiate large or illiquid token trades through a structured request-for-quote (RFQ) auction. Multiple participants compete on price without exposing their quotes to other bidders during the commit window. Bonds make non-performance costly, and a Solana program controls selection, settlement, refunds, and deadline-based exits.

This repository is the **project and evaluation hub** for the UnleakTrade organization. The implementation lives in the linked repositories below.

**[Try the beta](https://app.unleak.trade) · [Request access](https://unleak.trade/app) · [Public documentation](https://unleaktrade.gitbook.io/unleaktrade-docs) · [FAQ](https://unleak.trade/faq) · [GitHub organization](https://github.com/unleaktrade)**

> **Current release: Solana devnet beta.** To trade, register through the website's waitlist and activate your registration. Activated wallets receive 1,000 **Unleak USDC (uUSDC)**, a devnet-only test token with no monetary value. Connect the same wallet in the app. Start with the [judge's walkthrough](docs/judges-guide.md).

## Start here

| You want to… | Read this |
| --- | --- |
| Understand the project and its audience | This overview |
| Try the complete trading flow | [Judge's walkthrough](docs/judges-guide.md) |
| Find the right source repository | [Repository directory](docs/repositories.md) |
| Inspect the system and its trust boundaries | [Architecture](docs/architecture.md) |
| Understand quotes, bonds, fees, and failure paths | [Protocol lifecycle](docs/protocol.md) |
| Run components locally or inspect tests | [Development and testing](docs/development.md) |
| Separate implemented features from future work | [Status and roadmap](docs/status.md) |
| Check the exact source revisions behind this hub | [Source snapshot](docs/sources.md) |

## The problem

Large trades can be expensive to execute against shallow public liquidity. Bilateral OTC negotiations offer another route, but often involve fragmented chats, limited quote comparison, uncertainty about a counterparty's ability to perform, and manual settlement coordination.

UnleakTrade brings competitive quoting and explicit execution rules into one workflow. The maker defines the trade, bidders commit sealed quotes, the maker selects a valid revealed offer, and the selected bidder completes an atomic exchange. The protocol also defines what happens when someone misses a required step.

## Who it is for

| Participant | What they need | What UnleakTrade provides |
| --- | --- | --- |
| Funds, DAOs, and token treasuries | Negotiate meaningful token transfers with clear terms | RFQs with maker-defined amounts, minimum quotes, bonds, fees, and time windows |
| Large holders and holders of illiquid tokens | Find counterparties beyond an exchange's available pool depth | Competitive quoting for supported SPL token pairs, including tokens without an exchange listing |
| Market makers and OTC desks | Compete without exposing a quote for others to copy immediately | Commit–reveal quoting and a defined settlement process |
| Facilitators | Connect trading counterparties | Optional, explicitly designated rewards from completed trades |

These are intended users; this hub does not present them as existing customers. Each RFQ currently exchanges one base token for one quote token with one selected counterparty.

## How a trade works

1. **Create and open an RFQ.** The maker sets the pair, amounts, economics, and deadlines, then posts a bond to open bidding.
2. **Commit competing quotes.** Each taker obtains a signed liquidity attestation, posts a bond, and records a SHA-256 commitment on Solana.
3. **Reveal.** After commits close, takers reveal their quotes. The program checks that each reveal matches its commitment and satisfies the RFQ minimum.
4. **Select and escrow.** The maker selects one valid quote and deposits the base tokens in the same instruction.
5. **Settle atomically.** The selected taker supplies the quote tokens and fee. The program exchanges both assets and refunds the maker's and selected taker's bonds in one transaction.

Valid unselected takers reclaim their bonds after the funding deadline. Expiry, ignored quotes, and a selected taker's failure to fund have explicit exit paths. See the [full lifecycle](docs/protocol.md).

## What makes the design useful

- **Competition with sealed quotes:** bidders cannot read one another's quote amounts from the on-chain commitment before reveal.
- **Capacity checks before commitment:** the Liquidity Guard checks balances and signs the commitment; Solana verifies the authorized signature when a quote is committed.
- **Economic accountability:** maker and taker bonds put a cost on defined forms of non-performance.
- **Capital committed in stages:** the maker deposits the base asset at selection; the taker transfers the quote asset at settlement.
- **Atomic exchange:** a successful settlement exchanges both legs together. Atomicity does not force an unwilling or unfunded participant to complete a trade.
- **Visible accounting:** the app exposes activity, fees, slashed bonds, and eligible facilitator rewards from on-chain accounts.

## Product preview

![UnleakTrade devnet marketplace showing RFQs grouped by state](https://github.com/unleaktrade/docs/blob/b15c6e31711a5b4aaa92274f3d44986fcf02f86d/.gitbook/assets/user-guide/marketplace.png?raw=true)

*Official user-guide screenshot captured using the app's recorded test fixtures. The displayed counts and amounts illustrate the interface; they are not adoption or trading-volume claims.*

## Repository map

| Repository | Clear purpose |
| --- | --- |
| **[app](https://github.com/unleaktrade/app)** | Browser trading application: discover RFQs, create requests, commit/reveal quotes, select, settle, reclaim bonds, and view activity. |
| **[settlement-engine](https://github.com/unleaktrade/settlement-engine)** | Core Solana/Anchor program: accounts, state transitions, signature enforcement, escrow, atomic swaps, bonds, fees, and rewards. |
| **[liquidity-guard](https://github.com/unleaktrade/liquidity-guard)** | Rust HTTP service: check a taker's token balances and issue the Ed25519 attestation required to commit a quote. |
| **[landing-page](https://github.com/unleaktrade/landing-page)** | Public website: product explanation, FAQ, roadmap, and waitlist registration/activation interface. |
| **[waitlist](https://github.com/unleaktrade/waitlist)** | Go onboarding service: registrations, activation emails, and asynchronous devnet uUSDC distribution. |
| **[docs](https://github.com/unleaktrade/docs)** | GitBook source: concepts, protocol lifecycle, economics, and illustrated user guides. |
| **[actions](https://github.com/unleaktrade/actions)** | Shared GitHub Actions and workflows for build and CI tooling, including Solana CLI and runner setup. |
| **[generate_keypair](https://github.com/unleaktrade/generate_keypair)** | Standalone Rust developer utility for generating Solana keypairs and mnemonic-based keys. |
| **[experimental-preflight-sigcheck](https://github.com/unleaktrade/experimental-preflight-sigcheck)** | Experimental proof of concept for off-chain signing and on-chain signature verification. |
| **[colosseum-2026](https://github.com/unleaktrade/colosseum-2026)** | This hub: the submission overview, evaluation walkthrough, and map to implementation evidence. |

The [detailed directory](docs/repositories.md) identifies useful entry points and how the repositories depend on one another.

## Technology

| Layer | Implementation |
| --- | --- |
| Blockchain and settlement | Solana, Rust, Anchor, classic SPL Token program, associated token accounts, program-derived addresses |
| Quoting and verification | SHA-256 commitments, Ed25519 signatures, Solana's native signature-verification instruction |
| Trading interface | React, TypeScript, Vite, Tailwind CSS, Solana Wallet Adapter / Wallet Standard, TanStack Query, RPC and WebSocket subscriptions |
| Liquidity service | Rust, Actix Web, Solana RPC, Docker, Heroku |
| Onboarding | Go, Gin, DynamoDB, email activation, devnet token transfers |
| Engineering | GitHub Actions, Vitest, Playwright, Anchor/TypeScript integration tests, Rust and Go tests, Figma |

## Privacy and current scope

The current privacy protection is **sealed competing quotes during price discovery**. RFQ accounts, including submitted drafts, are public on-chain; reveals and settlement are public too. The Liquidity Guard receives quote details to perform its checks and is a trusted off-chain attester. A balance check is a point-in-time observation, not a reservation of the taker's future funds.

Zero-knowledge solvency verification is **planned**, not part of the current beta. The implemented token path uses the classic SPL Token program; broad Token-2022 extension support, NFTs, multi-asset bundles, and partial fills are not claimed here. See [architecture](docs/architecture.md) and [status](docs/status.md).

## Explore further

- [Judge's walkthrough and access troubleshooting](docs/judges-guide.md)
- [Official website](https://unleak.trade) and [FAQ](https://unleak.trade/faq)
- [Illustrated user guide](https://unleaktrade.gitbook.io/unleaktrade-docs/user-guide)
- [Builder profile](https://unleak.trade/builder)
- [Project updates on X](https://x.com/unleaktrade)

Documentation snapshot prepared on **12 October 2026**. Repository histories remain the authoritative record of development; the creation date of this hub does not imply that every linked component was created during the hackathon.
