# Development and testing

[Back to the project overview](../README.md) · [Repository directory](repositories.md)

This hub contains documentation. Run builds and tests in the relevant implementation repository. Read that repository's current README and pinned toolchain before changing its code.

## Fastest start: run the app against devnet

The app declares Node.js 22 or newer; use a current Node 22 LTS release compatible with its Vite dependencies.

```bash
git clone https://github.com/unleaktrade/app.git
cd app
npm ci
npm run dev
```

Open `http://localhost:3000`, select Devnet, connect a wallet, and follow the [beta access guide](judges-guide.md). The app ships an IDL, derives its program address from that IDL, and reads the current bond-token mint from on-chain configuration.

The development server proxies Guard requests. **Read-only exploration and quote submission have different requirements:** if the hosted Guard requires a client key, quote submission needs `LG_API_KEY_DEVNET` configured for the local build. See the app's [configuration example](https://github.com/unleaktrade/app/blob/main/.env.example) and [Vite configuration](https://github.com/unleaktrade/app/blob/main/vite.config.ts). A local Guard can be used instead with matching program configuration.

Public browser client keys are revocable identifiers, not a place to store a server secret. Keep Guard signing keys, wallet keypairs, and deployment credentials outside the browser configuration.

## Local validator development

The app's localnet tooling expects these sibling directories:

| Directory | Checkout |
| --- | --- |
| `app` | `https://github.com/unleaktrade/app.git` |
| `settlement-engine` | `https://github.com/unleaktrade/settlement-engine.git` |
| `liquidity-guard` | `https://github.com/unleaktrade/liquidity-guard.git` |

Prerequisites include Rust/Cargo, the Solana CLI/local validator, Anchor, Node/npm, and the engine's Yarn dependencies. The engine's reviewed [Anchor.toml](https://github.com/unleaktrade/settlement-engine/blob/main/Anchor.toml) pins Anchor **0.32.1**.

The app documents this localnet sequence:

```bash
# Run from app/, with sibling repos and toolchains installed.
bash scripts/dev-localnet.sh
npm run copy-idl
# Start and configure liquidity-guard on port 8080 in another terminal.
npm run seed
npm run dev
```

Follow the [app scripts guide](https://github.com/unleaktrade/app/blob/main/scripts/README.md) for validator, fixtures, and wallet preparation. Select Localnet in the UI. The Guard's network, bond-token mint, and signing public key must agree with the local program's configuration. Configure required signing material locally using the Guard's README; no secrets are provided by this hub.

Seeding creates test accounts and transactions. It is not a reset command and should be used on a development network. Localnet creates its own token setup; the public waitlist allocation is specific to devnet.

## Component commands and test boundaries

| Repository | Commands / evidence | Scope |
| --- | --- | --- |
| `app` | `npm run typecheck`, `npm run lint`, `npm test`, `npm run build` | Type checking, lint, Vitest checks, and production build. Build alone does not type-check. |
| `app` | `npm run test:e2e` | Desktop/mobile UI checks using recorded RPC responses and ephemeral wallets |
| `app` | `npm run test:e2e:tx` | Real devnet commit → reveal → select → settle → reward flow; requires funded test wallets, RPC, and Guard access |
| `settlement-engine` | `yarn install`, `anchor build`, `anchor test` | Build and integration tests; the suite obtains Guard attestations and needs the documented test setup |
| `settlement-engine` | `yarn test:unit` | Focused TypeScript tests for supporting utilities and configuration tooling |
| `liquidity-guard` | `cargo test`, `cargo clippy --all-targets -- -D warnings`, `cargo fmt --check` | Rust tests, including in-process mocked RPC/HTTP tests, lint, and formatting |
| `landing-page` | `npm ci`, `npm test`, `npm run coverage`, `npm run build` | Website, registration/activation UI, coverage, and build checks |
| `waitlist` | Go tests and the repository's [CI workflow](https://github.com/unleaktrade/waitlist/blob/main/.github/workflows/test_heroku_deploy.yml) | Unit tests plus credential-dependent DynamoDB/mail tests; devnet distribution tests need separate test credentials and assets |

The [app E2E README](https://github.com/unleaktrade/app/blob/main/e2e/README.md) is the detailed authority for current test modes. Its recorded-RPC suites do not prove live service availability. The transaction suite is separate and opt-in. Install Playwright's browser dependencies as directed by that repository before running browser tests.

The [Guard README](https://github.com/unleaktrade/liquidity-guard/blob/main/README.md) also documents its Postman/Newman collection, health/readiness checks, metrics, rate limiting, and Docker setup.

## Cross-repository contracts worth reviewing

1. **Commitment bytes:** the engine, Guard, and app must hash the same fields in the same order with the same integer encodings.
2. **Fee arithmetic:** off-chain quote funding checks must match the on-chain integer fee formula.
3. **IDL/account layouts:** refresh the app's committed IDL after engine changes; decoder tests check account encoding parity.
4. **Configuration snapshots:** the UI must distinguish current Config values from the values already stored in an existing RFQ.
5. **Network consistency:** the wallet, RPC, program, Guard, token mint, and test fixtures must belong to the intended environment.
6. **Onboarding mint drift:** the waitlist worker verifies its configured campaign mint against on-chain Config and pauses distribution when they differ.

## CI and delivery

The app and public website deploy through GitHub Actions to GitHub Pages. The app has separate preview and transaction-test workflows. The Guard is containerized for Heroku. The waitlist has a manual Heroku deployment path and stores its data in DynamoDB. Shared actions live in the organization's `actions` repository.

Inspect each repository's workflow configuration for the current gates. This hub describes test coverage and entry points; it does not assert that all upstream test suites were rerun or are passing at the time a judge reads it.

## Contributing

File changes in the repository that owns the behavior. UI changes belong in `app`, protocol rules in `settlement-engine`, balance-check behavior in `liquidity-guard`, onboarding in `waitlist`/`landing-page`, and product reference material in `docs`. Update this hub when a change affects the evaluation steps or repository map.

Refer to each repository's own license and attribution. This hub does not apply a blanket license to all linked implementations.
