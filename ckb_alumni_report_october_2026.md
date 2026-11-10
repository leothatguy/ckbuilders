## CKBuilders Alumni Report - October 2026

**Name:** Divine Oshione Anesi

**Period Covered:** 09-09-2026 to 11-10-2026


### Overview

- This report documents the Nervos work done through September to October, across four codebases:
  * Plimsoll, the BTC-backed stablecoin CDP protocol on CKB, taken from finished contracts to a live, verified testnet deployment
  * Loavix, the Fiber Network payments backend, hardened and moved to its own hosting
  * `fiber-node-connector`, the standalone connector merchants run beside their Fiber node
  * The hosting platform that runs the Plimsoll keeper and the Loavix stack on a single VPS

### Plimsoll

- Built the off-chain half of the protocol on top of the contracts:
  * `plimsoll-sdk`: byte-exact codecs, transaction builders for all seven operations, the read side, and an oracle refresh path, with typed script errors
  * A devnet end-to-end harness that deploys everything to a local chain and runs the full loop as real transactions, including a permissionless liquidation by a second account
  * A liquidation keeper that polls, refreshes a stale price, scans vaults, and liquidates, with race handling, a dry-run mode, and profit logging
  * A web app with the Plimsoll-mark health gauge, vault management, and an all-vaults table ordered riskiest first
  * An idempotent deployment toolbox with capacity planning, artifact receipts, and a read-only `verify` command
- Deployed the protocol to CKB testnet and released its public surfaces:
  * Deployed the four scripts and the xBTC collateral token, with every transaction recorded in checked-in artifacts
  * Published `plimsoll-sdk` 0.1.0 on npm
  * Hosted the web app at plimsoll-web.vercel.app
- Kept the price feed alive after Pyth began requiring authentication:
  * Added an authenticated Hermes fetcher and threaded the API key through the SDK and the keeper
  * Repointed the testnet deployment at a self-hosted Pyth core oracle feed
  * Added a freshness watchdog that alarms before the feed goes stale
- Packaged the keeper for unattended operation:
  * A hardened systemd service with an installer
  * A distroless container image published by digest, and a hand-build path for when CI cannot run
- Redesigned the web app for readability:
  * Raised every failing text contrast to WCAG AA with auditable colour tokens
  * Reworked the layout, typography, vault card, and vault dialog, and rewrote the copy in plainer language
- Migrated the protocol to Lean Oracle v2:
  * `vault-type` now prices vaults from a committee-signed feed cell and checks the committee cell behind it, rejecting a paused committee or a price signed by a retired key set
  * Added a fifth script, `feed-cell-lock`, so the deployment owns a feed cell anyone may move forward and nobody may burn
  * Extended the SDK to read, attach, and refresh the v2 feed cell from the oracle's mirrors
  * Ran the full vault loop on devnet against the real Lean Oracle v2 binaries
  * Deployed to testnet as a second deployment, reusing the unchanged code cells, and made it the default for the app and the keeper
- Verified the live deployment end to end on testnet:
  * A smoke test runs a vault's whole life (open with debt, mint, repay, close) through the web app's own modules
  * A liquidation test has the real keeper, not in dry-run, liquidate a vault after a 25 percent price fall, with 22 independent assertions on the settlement
  * The live feed cannot be pushed down, so the test prices its vault from a feed cell under a one-key committee it holds, while Lean Oracle's real contracts verify every signature
- Found and fixed a mint-authority vulnerability through that liquidation test:
  * The authority pinned only the `vault-type` and xUDT identities, while a vault's oracle, committee, and collateral token sit in the vault's own args, chosen by whoever opens it
  * A vault priced by a committee of the opener's own could therefore mint the same `plUSD` as an honest vault
  * Fixed by pinning the whole vault config in the authority's args, so only vaults of exactly this deployment are counted
  * Added seven contract tests covering another oracle, another token, another authority, and a foreign vault beside a legitimate one
  * Redeployed the authority on testnet, re-ran the smoke and liquidation tests against it, and reclaimed the retired code cell
- Fixed two further issues surfaced by testnet work:
  * SDK reads now come from the chain; the client library's cache kept returning vaults another party had already liquidated
  * The Rust toolchain is pinned and recorded in deployment artifacts, so a code cell's bytes can be rebuilt
- Brought the documentation up to date:
  * Rewrote the README, including a section on how Plimsoll consumes Lean Oracle safely
  * Updated the handbook, the byte-exact script specification, the SDK notes, and the build plan

### Loavix

- Hardened the Fiber payments backend for production:
  * Pending withdrawals are claimed atomically, so each is paid exactly once
  * The API reports not-ready with a 503 and checks the database, Redis, and Fiber reachability
  * Replaced the replayed schema script with versioned migrations applied per release
  * Images run as a non-root user with production dependencies only
- Moved off AWS onto the self-managed host:
  * Retired the AWS pipelines and deployment path
  * Releases reach production through a restricted deploy key, and the Fiber node keeps running across releases
- Adopted the standalone connector:
  * The backend accepts the `fiber-node-connector` protocol and enforces one active connector per merchant
  * Retired the in-repo connector runner and its image
  * Completed the first end-to-end Fiber payment through the hosted stack

### fiber-node-connector

- Fixed reliability issues in the connector and gateway:
  * Poll timers stay referenced and finished abort listeners are dropped
  * Timed-out commands are removed from the queue
- Added a per-process instance id, so a second connector for the same merchant stands by instead of competing
- Kept the build pipeline's pinned actions current

### Hosting Platform

- Built a reproducible single-host platform for the Nervos backends:
  * A service catalog with policy validation and a release contract
  * Ansible roles for the baseline, SSH, firewall, rootless Docker, and nginx TLS ingress
  * A daily read-only drift check comparing the host against the repository
  * Validated every role on a disposable VM before touching production
- Runs the Plimsoll keeper as a rootless container with a watchdog, keeping the testnet price feed fresh
- Runs the Loavix stack as a rootless compose project behind TLS
- Ships pinned container images to the host without a registry credential on it, each blob verified by digest

### Key Learnings

- A stateless authority must pin everything it trusts:
    - a script's args are chosen by whoever creates the cell, so any identity left out of the authority's args is the attacker's to choose
    - the mint authority has to pin the oracle and the collateral token, not only the code that reads them
    - leaving the authority's own hash out of the pinned config is what keeps the two args layouts acyclic
    - running the real system against a real chain found this; the unit tests could not, because they never varied the vault config

- An oracle consumer needs three separate guarantees:
    - authenticity: the feed cell is pinned by type hash and anchored to a committee the script checks itself
    - freshness: the price is compared against a recent block header, and a superseded cell cannot be replayed because a cell dep must be live
    - liveness: when the oracle publishes no shared cell, the consumer must own one that anyone can update and nobody can destroy

- A client-side cache is not chain state:
    - a cell the client has seen stays visible until that same client spends it
    - any reader that outlives a single transaction must ask the chain, or it will act on cells someone else already spent
    - a settlement check must read through a client that never saw the cells it expects to be gone

- A code cell is named by its bytes, so the build must be reproducible:
    - an unpinned toolchain makes a deployed hash impossible to reproduce later
    - recording the compiler version next to the deployment receipt turns "trust the artifact" into "rebuild and compare"
    - content addressing makes redeployment idempotent: unchanged scripts are found and reused, not paid for again

- Derived values must never be inputs to their own derivation:
    - a recorded token hash is a cross-check of the result
    - deriving with the stale value still declared hid a mismatch after a redeploy, and only the SDK's own validation caught it before anything was sent

### Environment

- Projects: Plimsoll, Loavix, `fiber-node-connector`, and their hosting platform
- Target network: CKB testnet
- Plimsoll scripts: `vault-type`, `vault-lock`, `vault-collateral-lock`, `mint-authority`, `feed-cell-lock`
- Oracle: Lean Oracle v2, `Crypto.BTC/USD`, committee-signed
- Contract framework: `ckb-std` 1.1 and `ckb-testtool`, Rust 1.98.0 pinned
- Protocol parameters: 150 percent minimum ratio, 130 percent liquidation threshold, 10 percent incentive, 100 `plUSD` minimum debt, 30-minute staleness
- Test status: 167 contract tests, 243 SDK tests, 60 keeper tests, 109 web tests, 45 deployment tests, and 80 devnet end-to-end assertions passing
- Live surfaces: testnet deployment, web app on Vercel, `plimsoll-sdk` on npm, keeper on a VPS
