## Builder Track Weekly Report - Week 24

**Name:** Divine Oshione Anesi

**Week Ending:** 09-01-2026


### Milestones Completed

- Completed the critical Plimsoll cell-architecture spike:
  * Verified the exact 152-byte Lean Oracle price-cell layout against the contract source
  * Defined how the protocol identifies the expected BTC/USD feed through its type script, feed ID, and canonical lock
  * Confirmed that the live oracle outpoint changes whenever its price cell is updated
  * Moved oracle identities into network deployment configuration instead of hardcoding them in the contract
- Resolved the initial oracle freshness model for price-dependent vault operations:
  * Confirmed that an oracle cell used as a dependency must still be live when the transaction is accepted
  * Combined dependency liveness with an explicit header dependency and a 30-minute staleness policy
  * Removed the proposed input `since` anchor after confirming it only lower-bounds confirmation time and cannot expire a transaction
  * Documented permissionless oracle refresh as the defense when the public feed stops updating
- Resolved the plUSD mint-authority and vault-lock design boundaries:
  * Ruled out xUDT owner mode by vault type hash because every vault has a unique type-ID argument
  * Selected a stateless mint-authority lock whose script hash becomes the plUSD xUDT owner identity
  * Defined the supply invariant requiring net plUSD inflation to equal the matching increase in vault debt
  * Separated owner-authorized vault operations from permissionless liquidation through a dedicated vault lock
- Implemented the first shared Rust protocol foundation:
  * Added a `no_std` Lean Oracle cell parser using the verified byte offsets and little-endian field formats
  * Added consumer validation for feed mismatch, zero publish time, non-positive price, and stale data
  * Added checked fixed-point price calculations for collateral value, collateral ratios, and liquidation amounts
  * Added focused host-side tests for parsing, freshness boundaries, overflow, threshold equality, and rounding direction
- Continued advanced Rust self-study with a focus on explicit enum representation:
  * Studied how `#[repr(i8)]` gives protocol errors a stable integer representation
  * Used explicit discriminants to keep script exit codes consistent across contract modules
  * Kept conversion from domain errors to CKB exit codes deliberate through a small `From` implementation
  * Compared stable protocol-facing error values with ordinary enum variants whose numeric layout is not part of an interface

### Key Learnings
- A valid oracle type script does not by itself identify the canonical public feed instance:
    - matching the feed ID prevents another asset's price from being used as BTC/USD
    - matching the canonical lock separates the intended public feed from a separately created cell for the same feed
    - rejecting a zero publish time prevents a new cell that has never accepted an authenticated update from qualifying
    - keeping these identities in deployment configuration allows the protocol to follow verified oracle upgrades

- CKB dependency liveness provides an important oracle-consumer guarantee:
    - updating the oracle consumes its previous cell and creates a new live cell
    - a transaction referring to the consumed outpoint cannot be accepted afterward
    - an off-chain transaction builder must rediscover the current oracle cell for every price-dependent transaction
    - an update racing a vault transaction should be handled as a normal retry condition

- A `since` value cannot act as transaction expiry:
    - `since` defines the earliest point at which an input may be consumed
    - it cannot set a latest acceptable confirmation time
    - using it would not prevent a previously constructed transaction from being submitted later
    - the freshness model must state this limitation instead of claiming a stronger time guarantee than CKB provides

- xUDT mint authority must have one stable identity across every vault:
    - including a unique type ID in each vault script produces a different full type-script hash per vault
    - those per-vault hashes cannot serve as one common plUSD owner identity
    - a stateless mint-authority lock provides one stable script hash without creating a protocol-owned balance
    - per-user authority cells avoid making all mint operations compete for one singleton cell

- Explicit Rust error discriminants improve the contract boundary:
    - CKB callers observe integer exit codes rather than Rust enum names
    - fixed discriminants let scripts, tests, client error mappings, and documentation refer to the same failure
    - grouping codes by subsystem makes failures easier to locate without changing their meaning
    - published discriminants should remain stable after deployment because changing them would break integrations

### Practical Progress
- Completed the on-chain architecture document:
    - Recorded the oracle data layout and consumer checks
    - Documented dependency liveness, header attestation, and the limits of transaction freshness
    - Defined the plUSD owner-mode wiring and mint-authority responsibility
    - Defined the vault type, owner lock, collateral lock, and liquidation authorization boundaries

- Added the initial byte-exact protocol specification:
    - Defined the vault state and script-argument layouts
    - Assigned one-byte witness operation values for the seven planned vault transitions
    - Defined the Type ID rule used to create a unique vault identity
    - Grouped stable error codes by oracle, arithmetic, vault, lock, and mint-authority failures

- Implemented and tested Lean Oracle parsing in `plimsoll-common`:
    - Decoded all fixed-width fields from the 152-byte oracle state
    - Rejected data with an incorrect length
    - Validated feed identity, publish time, positive price, and maximum staleness
    - Kept the parser independent of CKB syscalls so it can be tested directly on the host

- Implemented the initial collateral price-math module:
    - Normalized BTC/USD prices using the Pyth exponent
    - Used checked `u128` multiplication to reject overflow
    - Implemented minimum-collateral and liquidation-threshold comparisons without division
    - Applied explicit floor rounding against the party receiving value

- Added testnet integration configuration:
    - Recorded the Lean Oracle type-script identity and code dependency
    - Recorded the BTC/USD feed ID, guardian-set type hash, and canonical oracle lock
    - Recorded the canonical testnet xUDT deployment information
    - Left Plimsoll script and asset outpoints empty until an actual testnet deployment is completed

### Environment
- Report period ending: 09-01-2026
- Project: Plimsoll BTC-backed stablecoin CDP protocol
- Current phase: Phase 1 architecture and shared protocol foundation
- Target network: CKB testnet
- Oracle source: Lean Oracle BTC/USD feed backed by Pyth data
- Oracle state format: 152-byte fixed layout
- Oracle staleness policy: 30 minutes
- Shared protocol crate: Rust `no_std`
- Arithmetic representation: checked `u128` fixed-point values
- Contract framework direction: `ckb-std` and `ckb-testtool`
- Rust study focus: explicit enum representation and stable script exit codes
