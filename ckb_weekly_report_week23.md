## Builder Track Weekly Report - Week 23

**Name:** Divine Oshione Anesi

**Week Ending:** 08-25-2026


### Milestones Completed

- Established the initial scope for Plimsoll, a BTC-backed stablecoin CDP protocol on CKB:
  * Selected testnet xBTC as collateral because the available live oracle feed prices BTC rather than native CKB
  * Defined `plUSD` as the debt asset minted against overcollateralized personal vaults
  * Defined the borrower flow around opening, depositing, minting, repaying, withdrawing, and closing vaults
  * Kept the MVP focused on one collateral asset, flat debt, full-vault liquidation, and testnet deployment
- Specified the initial CKB cell model for vault state and collateral custody:
  * Separated vault debt state from xBTC collateral because one cell cannot use both the vault type script and xUDT type script
  * Assigned each vault a type-ID-style identity that survives valid state transitions
  * Proposed a vault-specific collateral lock that binds separate xBTC cells to one vault
  * Required collateral movement and the matching vault transition to be validated atomically in one transaction
- Established the initial Lean Oracle integration requirements:
  * Kept BTC/USD oracle data read-only through `cell_deps` so vault operations do not compete to spend the price cell
  * Required the expected feed identifier and oracle type-script hash before accepting price data
  * Treated oracle freshness as a separate requirement from price authenticity
  * Scoped a price-gate spike to validate `header_dep`, `since`, and stale-price rejection before implementing vault logic
- Started the Plimsoll implementation repository:
  * Selected the Plimsoll product name and `plUSD` token identity
  * Initialized a private monorepo with contract, SDK, keeper, web, deployment, and documentation boundaries
  * Added the initial Rust workspace manifest and package-level implementation placeholders
  * Added dependency-free repository validation and a minimally privileged GitHub Actions workflow
  * Pushed the initial bootstrap and confirmed the first CI run passed
- Continued advanced Rust self-study with a focus on the newtype pattern and unit-safe arithmetic:
  * Studied how tuple structs can give identical integer representations different domain meanings
  * Compared primitive aliases with newtypes that prevent accidental interchange of collateral, debt, price, and ratio values
  * Examined checked arithmetic and explicit conversion boundaries for deterministic fixed-point calculations
  * Connected zero-cost domain types to safer contract code without adding runtime dispatch or heap allocation

### Key Learnings
- CKB cell composition changes how a collateralized vault must be represented:
    - the vault cell can own debt state and transition rules through its type script
    - xBTC remains governed by the xUDT type script and therefore belongs in separate collateral cells
    - a vault-specific lock can connect those collateral cells to one vault identity
    - transaction validation must reconcile the vault and its related collateral cells atomically

- Oracle authenticity and oracle freshness are different security properties:
    - checking the feed identifier prevents another asset's valid price from being accepted as BTC/USD
    - checking the oracle type-script hash ties the data to the expected verification rules
    - neither check proves that the published value is recent enough for a financial decision
    - freshness therefore needs a validated transaction time anchor in addition to authenticated data

- Stablecoin supply must remain tied to validated vault debt:
    - creating `plUSD` should require a matching increase in vault debt
    - repaying debt should require the corresponding amount of `plUSD` to leave circulation
    - merely requiring a vault cell in the transaction would not prove that the supply change matches its debt change
    - unauthorized minting tests need to be completed before secondary vault features

- Rust newtypes and checked arithmetic fit financial contract logic:
    - collateral quantity, price, debt, and ratio should not be interchangeable integers
    - tuple structs can enforce those distinctions without changing their compact integer representation
    - checked `u128` operations make overflow handling explicit during fixed-point calculations
    - rounding direction should be selected deliberately so calculations never overstate vault safety

### Practical Progress
- Completed the initial Plimsoll product specification:
    - Defined borrower, liquidator, keeper, and protocol-integrator responsibilities
    - Set the initial minimum collateral ratio at 150 percent and liquidation threshold at 130 percent
    - Set a 10 percent liquidation incentive and a minimum debt of 100 `plUSD`
    - Excluded governance, interest, partial liquidation, multiple collateral assets, and peg-defense mechanisms from the MVP

- Specified the initial vault state machine:
    - Open creates one uniquely identified zero-debt vault
    - Deposit increases bound xBTC collateral without changing debt
    - Mint increases debt only when the post-state remains above the minimum collateral ratio
    - Repay reduces debt without requiring a fresh price because repayment cannot make the vault less safe
    - Withdraw requires a fresh price and a safe post-state, while close requires zero remaining debt
    - Liquidation closes an unhealthy vault by burning its full debt and transferring discounted collateral

- Defined the implementation questions for the first contract design spike:
    - Confirm the exact Molecule layouts for vault state, script arguments, and operation witnesses
    - Confirm xUDT owner-mode wiring so token supply changes cannot bypass vault debt validation
    - Confirm the owner and permissionless liquidation paths for the vault-bound lock
    - Validate explicit-header access, transaction `since`, and stale oracle-cell behavior in an isolated price-gate script

- Bootstrapped the private implementation repository:
    - Created `contracts`, `packages/sdk`, `apps/keeper`, `apps/web`, `deployment`, and `docs` boundaries
    - Added root workspace metadata, a Rust workspace manifest, editor settings, ignore rules, and an MIT license
    - Added a repository check for required paths and obsolete product identifiers
    - Pinned the CI checkout action to an immutable commit and limited workflow permissions to repository read access
    - Confirmed the initial remote CI run succeeded

- Applied the Rust study to the contract design direction:
    - Mapped separate domain types for xBTC quantity, normalized price, `plUSD` debt, and ratio basis points
    - Kept raw integer decoding at the planned cell-data boundary
    - Selected checked multiplication and division for collateral-value calculations
    - Defined explicit rounding direction instead of relying on incidental integer truncation

### Environment
- Project: Plimsoll BTC-backed stablecoin CDP protocol
- Current phase: initial specification, repository bootstrap, and contract design spike
- Target network: CKB testnet
- Test collateral: xBTC xUDT with 8 decimals
- Stablecoin: `plUSD`
- Price source: Lean Oracle BTC/USD feed backed by Pyth data
- Minimum collateral ratio: 150 percent
- Liquidation threshold: 130 percent
- Liquidation incentive: 10 percent
- Contract direction: Rust with `ckb-std` and `ckb-testtool`
- Application direction: TypeScript SDK, keeper, and CCC-connected web interface
- Rust study focus: newtype pattern, checked arithmetic, and unit-safe fixed-point calculations
