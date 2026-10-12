# Current status and direction

[Back to the project overview](../README.md)

This page separates the implementation reviewed on **12 October 2026** from the project's longer-term direction. Source revisions are recorded in the [snapshot](sources.md).

## Implemented in the reviewed repositories

| Area | Current capability |
| --- | --- |
| RFQ lifecycle | Draft, open, commit, reveal, select, settle, and deadline-based exit instructions |
| Competition | Multiple takers can commit; maker selects one valid revealed quote |
| Quote confidentiality | SHA-256 commitment followed by reveal |
| Liquidity verification | Off-chain balance checks with Ed25519 attestations, enforced at commitment |
| Settlement | Maker base escrow at selection; atomic exchange when selected taker funds |
| Economics | Maker/taker bonds, quote-token fees, treasury accounting, optional facilitator reward claims |
| App | Marketplace, RFQ wizard, quote/reveal flow, wallet-derived actions, activity, and transparency screens |
| Recovery | Reveal-ticket backup plus eligible bond refund and close actions |
| Access | Website waitlist, email activation, and asynchronous devnet uUSDC distribution |
| Developer experience | Localnet scripts, seed fixtures, component tests, recorded-RPC UI tests, and separate live transaction tests |

The public product is presented as a **Solana devnet beta**. Source support for a network selector or environment variable is not evidence of a reviewed production release on that network.

## Current scope

- One base-token amount, one quote-token denomination, and one selected taker per RFQ.
- Classic SPL Token transfers; a token need not be exchange-listed to be used, provided it is compatible and participants have the required balances.
- Public RFQ accounts and public revealed quotes/settlement.
- Signed checks by a trusted Guard that receives quote details.
- Participant-submitted transactions for progress and cleanup; deadlines do not execute actions by themselves.

The source review documents implementation, not an independent audit or a guarantee of production safety. It does not claim live trading volume, customers, guaranteed execution prices, or permanent transaction privacy.

## Planned direction

The [FAQ's zero-knowledge section](https://unleak.trade/faq#zk-migration) describes the central next step: build a usable two-sided market, then reduce reliance on the off-chain attester through zero-knowledge solvency verification. The current beta does not implement that upgrade.

The broader [public roadmap](https://unleak.trade/roadmap) explores multi-asset RFQs, fragmented/partial fills, reputation-based incentives, integrations and SDKs, and institutional workflows. Those are future ambitions, not capabilities implied by today's one-pair/one-winner implementation. This hub assigns no new delivery dates or launch commitments.

## Why build now?

Solana supplies the primitives needed for this particular workflow: token transfers, program-controlled escrow, explicit account state, native Ed25519 verification, and transaction atomicity. That enables a usable commit–reveal venue without waiting for a fully private execution environment. The current beta lets participants evaluate the workflow and its incentives while deeper privacy remains an explicit next step.

## Submission provenance

This repository organizes the project for Colosseum 2026. The linked implementation repositories have their own development histories. Their commit dates, contributions, and dependencies should be evaluated from those histories; this hub does not label all pre-existing work as hackathon-period work.

See the [repository directory](repositories.md) for ownership of each component and the [judge's walkthrough](judges-guide.md) for what can be evaluated directly.
