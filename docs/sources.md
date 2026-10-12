# Sources and reviewed revisions

[Back to the project overview](../README.md)

This hub was prepared on **12 October 2026** by reading the public repositories, the official FAQ, and the published product documentation. This table preserves the exact source revisions inspected. Repository root links elsewhere lead to current development; deep source links in this snapshot point to the reviewed commits.

## Repository snapshot

| Repository | Reviewed commit |
| --- | --- |
| [settlement-engine](https://github.com/unleaktrade/settlement-engine) | [`a618b1d88f93`](https://github.com/unleaktrade/settlement-engine/tree/a618b1d88f93e1811e179acf54c02ced2d7f004b) |
| [liquidity-guard](https://github.com/unleaktrade/liquidity-guard) | [`cde005124543`](https://github.com/unleaktrade/liquidity-guard/tree/cde0051245431039c92d96abb6433cf3ac695296) |
| [app](https://github.com/unleaktrade/app) | [`ca05bf35fcb4`](https://github.com/unleaktrade/app/tree/ca05bf35fcb4b96b55dab3db419cca5a4cb1be1d) |
| [landing-page](https://github.com/unleaktrade/landing-page) | [`db74a0da866d`](https://github.com/unleaktrade/landing-page/tree/db74a0da866dce50c66382c752738b9d5d360735) |
| [waitlist](https://github.com/unleaktrade/waitlist) | [`93f9d446645c`](https://github.com/unleaktrade/waitlist/tree/93f9d446645c30bddf316243b9240676974773d6) |
| [docs](https://github.com/unleaktrade/docs) | [`b15c6e31711a`](https://github.com/unleaktrade/docs/tree/b15c6e31711a5b4aaa92274f3d44986fcf02f86d) |
| [actions](https://github.com/unleaktrade/actions) | [`bfe17249c1c5`](https://github.com/unleaktrade/actions/tree/bfe17249c1c5e957302d9b824bfd3e097cf66b77) |
| [generate_keypair](https://github.com/unleaktrade/generate_keypair) | [`1f6338575ece`](https://github.com/unleaktrade/generate_keypair/tree/1f6338575ece9fa7cf8386a0260ddd6cdbee29a2) |
| [experimental-preflight-sigcheck](https://github.com/unleaktrade/experimental-preflight-sigcheck) | [`224b95670dac`](https://github.com/unleaktrade/experimental-preflight-sigcheck/tree/224b95670dacdbef299965f8c6b47c25bd89c65a) |

## Evidence map

| Topic | Primary source |
| --- | --- |
| Lifecycle states and deadlines | [RFQ account](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/state/rfq.rs) |
| Guard signature enforcement and commitment reuse | [Commit instruction](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/instructions/quote/commit_quote.rs) |
| Maker's deposit at selection | [Select instruction](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/instructions/settlement/select_quote.rs) |
| Token exchange and fee/bond transfers | [Complete-settlement instruction](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/instructions/settlement/complete_settlement.rs) |
| Fee arithmetic, including zero-rate behavior | [Settlement account](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/state/settlement.rs) |
| Refund eligibility and ignored RFQs | [Refund instruction](https://github.com/unleaktrade/settlement-engine/blob/a618b1d88f93e1811e179acf54c02ced2d7f004b/programs/settlement-engine/src/instructions/quote/refund_quote_bonds.rs) |
| Guard inputs, balance checks, and test bypass | [Guard implementation](https://github.com/unleaktrade/liquidity-guard/blob/cde0051245431039c92d96abb6433cf3ac695296/src/main.rs) |
| Wallet/RPC/Guard integration | [App chain code](https://github.com/unleaktrade/app/tree/ca05bf35fcb4b96b55dab3db419cca5a4cb1be1d/src/chain) |
| Reveal-ticket storage and recovery | [Reveal-ticket implementation](https://github.com/unleaktrade/app/blob/ca05bf35fcb4b96b55dab3db419cca5a4cb1be1d/src/app/lib/reveal-ticket.ts) |
| Current browser test modes | [E2E guide](https://github.com/unleaktrade/app/blob/ca05bf35fcb4b96b55dab3db419cca5a4cb1be1d/e2e/README.md) |
| Activation and uUSDC distribution | [Waitlist README](https://github.com/unleaktrade/waitlist/blob/93f9d446645c30bddf316243b9240676974773d6/README.md) and [worker](https://github.com/unleaktrade/waitlist/tree/93f9d446645c30bddf316243b9240676974773d6/internal/airdrop) |
| Public sponsor, beta access, and ZK direction | [FAQ source](https://github.com/unleaktrade/landing-page/blob/db74a0da866dce50c66382c752738b9d5d360735/src/components/FAQ.tsx) |
| Illustrated product walkthrough | [User-guide source](https://github.com/unleaktrade/docs/tree/b15c6e31711a5b4aaa92274f3d44986fcf02f86d/user-guide) |
| Marketplace screenshot | [Original asset](https://github.com/unleaktrade/docs/blob/b15c6e31711a5b4aaa92274f3d44986fcf02f86d/.gitbook/assets/user-guide/marketplace.png) and [capture methodology](https://github.com/unleaktrade/app/blob/ca05bf35fcb4b96b55dab3db419cca5a4cb1be1d/e2e/README.md) |

## Public references

- [Website](https://unleak.trade)
- [Trading beta](https://app.unleak.trade)
- [FAQ](https://unleak.trade/faq)
- [Published documentation](https://unleaktrade.gitbook.io/unleaktrade-docs)
- [Roadmap](https://unleak.trade/roadmap)

## Point-in-time service observation

During preparation, the devnet Guard's public health endpoint, at the URL configured in the app, returned `status: healthy`, `network: Devnet`, and `skip_fund_checks: false`. This confirms the reported mode at that check, not continuous uptime, an independent audit of balances, or a completed end-to-end trade. The source configuration and future deployments may change.

## How conflicting descriptions were handled

Executable protocol rules were used for exact fees, deadlines, transfers, and supported token-program behavior. The FAQ and current user guide supplied onboarding and user-facing context. Broad landing-page language was not treated as evidence that zero-knowledge proofs, guaranteed settlement, permanent privacy, or a fixed dollar fee are implemented.

The documentation review did not submit trading transactions, activate new waitlist accounts, or run the upstream product test suites. The hub's verification covers source references, document consistency, and repository publication.
