# Judge's walkthrough

[Back to the project overview](../README.md)

Use **Solana devnet** throughout this walkthrough. The public beta uses test assets with no monetary value. A browser, a Wallet Standard-compatible Solana wallet, and activated waitlist access are the shortest route to trying the trading flow.

## 1. Activate access and receive test tokens

1. Open [unleak.trade](https://unleak.trade) and choose **Request Access**, or go directly to the [waitlist screen](https://unleak.trade/app).
2. Register the wallet you intend to connect in the app and complete the requested registration details.
3. If you need a sponsor, the [official FAQ](https://unleak.trade/faq#why-request-access) provides this public sponsor address:

   ```text
   GhTppHPoSppnBurco1sPuYWyn3mejhsaRdRbt3dHtFyc
   ```

4. Follow the email activation link.
5. Activation triggers distribution of **1,000 Unleak USDC (uUSDC)** to the registered wallet. Distribution is asynchronous, so a successful activation can precede the token transfer.

uUSDC is a devnet test version of USDC with six decimals. It is not real USDC, has no monetary value, and has no public faucet. The current bond mint is read from the protocol's on-chain configuration; sending an unrelated devnet USDC token does not satisfy the bond requirement.

The waitlist allocation covers the beta's bond token. You also need **devnet SOL** for transaction fees and account creation, and the actual test assets required for your side of a trade. A uUSDC allocation alone does not supply every possible base or quote token.

## 2. Connect the same wallet

Open [app.unleak.trade](https://app.unleak.trade), select your wallet, and approve the sign-in message. The message signature itself does not transfer tokens. Check that the app and wallet are using **Devnet**.

Supported wallets include Phantom, Solflare, Backpack, and other Wallet Standard-compatible wallets. Connect the exact address used for registration so its uUSDC balance is available.

## 3. Explore the interface

| Screen | What to inspect |
| --- | --- |
| Marketplace | RFQs grouped by lifecycle state, pair/amount details, and deadlines |
| RFQ detail | The posted terms, current phase, available actions, and quote information appropriate to the phase |
| My Activity | Requests you posted, quotes you submitted, actions that need attention, and rewards |
| Transparency | On-chain records of protocol fees and slashed bonds |

Historical or seeded RFQs are useful for inspection, but closed requests cannot be reopened for a new test. The marketplace is not a promise that a counterparty will respond during your review session.

## 4. Run a complete trade

For a controlled demonstration, use **two distinct devnet wallets**: one to post a request and one to quote. Use a third wallet if you want to demonstrate competing bids and the unselected bidder's refund. Register and activate each wallet that needs the beta token allocation, and use separate browser profiles or switch wallets carefully.

Prepare the balances first:

- Maker: bond tokens, devnet SOL, and enough of the chosen base token to deposit at selection.
- Each taker: bond tokens, devnet SOL, and enough quote tokens to cover its quote plus the fee if selected.

Choose a pair for which those balances actually exist. If you need a coordinated test counterparty or additional test assets, use the Telegram support link on the [official website](https://unleak.trade) or [FAQ](https://unleak.trade/faq#devnet-usdc).

| Step | Wallet | Action | What to verify |
| --- | --- | --- | --- |
| Create | Maker | Use **Create RFQ**: pair → economics → timing → review | Pair, base amount, minimum quote, bond, fee, and four windows are explicit |
| Open | Maker | Open the draft | Maker bond is posted; commit window begins |
| Commit | Taker(s) | Open the RFQ and submit a quote | Liquidity check and signed attestation precede the on-chain commitment; bond is posted |
| Preserve recovery material | Taker(s) | Save the downloadable reveal ticket | Quote amount and salt can be recovered for reveal |
| Reveal | Taker(s) | Reveal after commits close, before the reveal deadline | The quote matches its commitment and becomes public |
| Select | Maker | Select one valid quote during the selection window | Base tokens move into escrow at selection |
| Settle | Selected taker | Complete settlement before the funding deadline | Base and quote exchange, fee is paid, and both selected participants' bonds return |
| Reclaim | Unselected valid taker | Reclaim after the funding deadline | Its own bond is refunded without selection |

Use time windows long enough to switch wallets and approve transactions. Keep the reveal ticket private until you are ready to reveal; it contains the sealed quote's recovery material. The app can also derive the salt again with the same wallet, but saving the ticket avoids relying on that recovery path alone.

If testing facilitator rewards, specify the **same facilitator address** on the RFQ and winning quote before settlement, then claim the reward with that address after the trade succeeds.

## 5. Inspect failure behavior

The protocol includes explicit expiry and refund paths. You can inspect terminal RFQs in the UI or create small devnet-only examples:

- Open an RFQ with no valid reveals, then close it after the reveal deadline.
- Reveal a quote but make no selection, then reclaim using the permitted post-deadline action.
- Select a quote but do not fund it; after the funding deadline, the maker can recover the escrowed base and its bond.

These paths can slash test-token bonds. Their exact rules are documented in the [protocol guide](protocol.md). Refunds and closure require transactions; they do not happen simply because a timer expires.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Registration rejected for sponsor | Use an activated sponsor or the public address in the FAQ |
| Activation succeeded but uUSDC is missing | Confirm the registered wallet and Devnet; distribution may be pending or the RPC may lag. Use the activation status and official support path. |
| Cannot pay network fees | The wallet needs devnet SOL as well as uUSDC |
| Quote fails the liquidity check | Bond balance and quote-plus-fee balance must satisfy the request; check the required mint and Guard status |
| Quote commit receives an API/authentication error in a local build | See the Guard client-key configuration in the [development guide](development.md) |
| Reveal is unavailable | Check the RFQ's commit/reveal windows and connected wallet; import the correct reveal ticket if needed |
| Settlement cannot proceed | Confirm that this wallet owns the selected quote, the deadline is open, and quote tokens plus fee are available |
| Unselected bond is not yet reclaimable | The program requires the funding deadline to have passed |
| No active counterparty or suitable token pair | Arrange a controlled two-wallet demo or ask for beta test support |

## Further evidence

- [Illustrated public user guide](https://unleaktrade.gitbook.io/unleaktrade-docs/user-guide)
- [App source and test suite](https://github.com/unleaktrade/app)
- [Settlement success/failure tests](https://github.com/unleaktrade/settlement-engine/tree/main/tests)
- [Exact source revisions reviewed for this hub](sources.md)

The screenshot in the overview uses recorded test fixtures. This walkthrough describes how to perform a live devnet evaluation; it is not a claim that a new end-to-end trade was executed while preparing this documentation.
