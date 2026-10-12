# RFQ lifecycle, bonds, and settlement

[Back to the project overview](../README.md) · [Architecture](architecture.md)

An RFQ specifies a fixed amount of a base token offered by the maker and a quote token in which takers bid. Higher valid quote amounts offer the maker more quote tokens for that base amount. The maker selects one valid revealed quote; there is no automatic matching or partial fill in the current flow.

## Roles

| Protocol term | Meaning |
| --- | --- |
| Maker | Creates and opens the RFQ; selects the counterparty and deposits the base asset |
| Taker | Commits and reveals a quote; completes settlement if selected |
| Facilitator | Optional address designated to receive a share of the successful trade's fee when both sides name the same address |

The app derives available actions from the connected wallet and RFQ state. Users do not need separate role-specific accounts.

## Successful path

| Phase | Instruction | What happens |
| --- | --- | --- |
| Draft | `init_rfq` | Creates the request and its parameters. The maker can edit or cancel before opening. |
| Open | `open_rfq` | Posts the maker bond and starts the commit window. |
| Commit | `commit_quote` | Verifies the Guard attestation, records the commitment, and takes the taker bond. More takers can commit before the deadline. |
| Reveal | `reveal_quote` | After commits close, checks the salt and amount against the commitment and minimum quote. |
| Select | `select_quote` | After the reveal window, selects one valid quote, transfers the maker's base tokens into escrow, and creates the settlement snapshot. |
| Settle | `complete_settlement` | The selected taker transfers quote tokens plus the fee; the program delivers the base and returns both selected participants' bonds atomically. |

States are `Draft`, `Open`, `Committed`, `Revealed`, `Selected`, `Settled`, `Expired`, `Ignored`, and `Incomplete`. `Committed` and `Revealed` also indicate that at least one participant has performed that action; the remaining participants still act within the relevant time window.

The core code is in [RFQ instructions](https://github.com/unleaktrade/settlement-engine/tree/main/programs/settlement-engine/src/instructions/rfq), [quote instructions](https://github.com/unleaktrade/settlement-engine/tree/main/programs/settlement-engine/src/instructions/quote), and [settlement instructions](https://github.com/unleaktrade/settlement-engine/tree/main/programs/settlement-engine/src/instructions/settlement).

## Deadlines

The maker defines four positive durations. The [RFQ account implementation](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/state/rfq.rs) calculates:

| Deadline | Calculation |
| --- | --- |
| Commit | `opened_at + commit_ttl` |
| Reveal | `opened_at + commit_ttl + reveal_ttl` |
| Selection | `opened_at + commit_ttl + reveal_ttl + selection_ttl` |
| Funding, once selected | `selected_at + fund_ttl` |
| Funding horizon, if never selected | `opened_at + commit_ttl + reveal_ttl + selection_ttl + fund_ttl` |

Selection therefore starts the winner's funding window. A timer reaching zero does not itself send a transaction: state changes and refunds require a valid instruction from an authorized participant.

## Bonds and fees are different

| Item | Denomination | Rule |
| --- | --- | --- |
| Participation bond | RFQ's configured USDC mint; uUSDC in the public devnet beta | One maker-defined amount posted by the maker and by each committing taker |
| Protocol fee | RFQ's quote token | Paid on top of the quote by the selected taker, only on successful settlement |
| Facilitator reward | Quote token | A share of the protocol fee when RFQ and selected quote specify the same nonempty facilitator address |
| Network/account costs | SOL | Transaction fees and account creation deposits, separate from bonds and protocol fees |

There is no protocol-enforced USD trade-size minimum or universal dollar-denominated fee floor. The maker sets the RFQ's bond and fee rate. The code allows a zero fee rate; for a positive rate the fee is:

```text
fee = max(1, floor(quote_amount × taker_fee_bps / 10,000))
```

Amounts in that formula are token base units. The minimum of one means **one smallest quote-token unit**, not one dollar. The maker receives the full quote amount. A qualifying facilitator's share is deducted from the total fee and held for a later `withdraw_reward` claim; the rest goes to treasury.

**Illustrative devnet example:** a selected quote of 100 uUSDC at a maker-set 50 bps fee costs the taker 100.5 uUSDC for the quote plus fee. If the RFQ bond is 10 uUSDC, each participant posted that bond earlier. Successful settlement returns the maker's and winner's own 10 uUSDC bonds; it does not pay one participant the other's bond. This example is not a default or pricing promise.

Fee source: [settlement calculations](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/state/settlement.rs).

## When a trade does not complete

| Situation | Exit/action | Economic outcome |
| --- | --- | --- |
| Maker cancels an unopened draft | `cancel_rfq` | Draft account closes; no participation bond was posted |
| No valid reveals after the reveal deadline | Maker calls `close_expired` | Maker bond returns; unrevealed taker bonds are slashed; RFQ becomes `Expired` |
| Valid quotes exist but maker selects none | An eligible revealed taker calls `refund_quote_bonds` after the funding horizon | That taker's bond returns; maker and unrevealed bonds are subject to slashing; RFQ becomes `Ignored` |
| Selected taker misses funding | Maker calls `close_incomplete` after the funding deadline | Maker's base asset and bond return; selected taker and unrevealed bonds are subject to slashing; RFQ becomes `Incomplete` |
| Valid revealed quote is not selected | Taker calls `refund_quote_bonds` after the funding deadline | The unselected taker reclaims its own bond |

Slashed bonds go **100% to the protocol treasury**, not to the other trader. Resolution trackers prevent repeated seizure in overlapping cleanup paths. The selected taker's bond is not recoverable through the ordinary unselected-quote refund instruction.

Code references: [expiry](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/instructions/rfq/close_expired.rs), [incomplete settlement](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/instructions/rfq/close_incomplete.rs), [refunds and ignored RFQs](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/instructions/quote/refund_quote_bonds.rs), and [slashing calculation](https://github.com/unleaktrade/settlement-engine/blob/main/programs/settlement-engine/src/slashing.rs).

## Facilitators

Facilitators can help source counterparties but have no special power to select quotes, bypass deadlines, or move trading assets. A reward requires successful settlement and matching facilitator addresses on the RFQ and winning quote. Reward calculation uses the RFQ's snapshotted facilitator fee rate, and the recipient claims the allocated amount after settlement.

## Practical interpretation

The mechanism provides a verifiable process with consequences for missed steps. It does not guarantee a counterparty will participate, that a trade will settle, or that the selected price is the best available anywhere. The current [trust and privacy boundaries](architecture.md) remain part of evaluating the product.
