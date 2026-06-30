## Builder Track Weekly Report - Week 15

**Name:** Divine Oshione Anesi

**Week Ending:** 06-29-2026


### Courses Completed

- Studied production-style script interface patterns through Pausable UDT and SSRI examples:
  * Looked beyond SSRI as a general idea and focused on how a real script can expose behavior that off-chain tools can understand
  * Considered how pause, resume, and metadata-style methods can make a script easier for wallets, indexers, and applications to integrate safely
  * Started connecting this to future invoice assets where the app may need to explain script behavior before asking a user to sign
- Continued Rust self-study with a focus on shared state and controlled mutation:
  * Reviewed how Rust separates ownership, borrowing, and mutation in larger programs
  * Looked at shared ownership and interior mutability as design tools, not just language syntax
  * Used the Rust model to think more clearly about ledger state, pending requests, and avoiding accidental double-use of a balance
- Turned the merchant Fiber connector into a distributable runtime:
  * Added a dedicated Docker image for the merchant connector
  * Added CI publishing for the connector image through GHCR
  * Updated the merchant setup flow so production merchants do not need to clone the Loavix source repo or install pnpm
- Implemented hosted Fiber withdrawal request accounting in Loavix:
  * Added a withdrawal ledger for hosted Fiber balances
  * Added pending, processing, paid, failed, and cancelled withdrawal states
  * Updated hosted Fiber balance calculation to separate total received, pending withdrawals, paid-out withdrawals, and available balance
- Tested the hosted Fiber withdrawal flow locally:
  * Started local PostgreSQL and Redis
  * Started the backend and frontend against the local API
  * Seeded hosted Fiber receipt data in the local database
  * Tested request, reservation, over-withdrawal rejection, processing, and paid completion over HTTP

### Key Learnings
- A packaged connector changes the merchant setup model:
    - the connector becomes an operational tool instead of a developer script
    - merchants only need a connector token, their merchant address, and a reachable Fiber RPC URL
    - the hosted Loavix app can stay online while the merchant connector runs beside a private Fiber node
    - GHCR visibility matters because a public image lets merchants pull the connector without a GitHub login

- Hosted Fiber withdrawal needs ledger-style accounting instead of a single balance number:
    - hosted Fiber receipts increase the total received amount
    - pending and processing withdrawals reserve part of the balance
    - paid withdrawals must continue reducing the available amount after payout
    - failed or cancelled withdrawals can release reserved balance back to available balance

- The withdrawal feature should not pretend that Fiber funds were paid out before a payout transaction exists:
    - the request can be created immediately
    - the available balance should be reduced immediately to prevent double withdrawal
    - the withdrawal should only become paid when a real payout transaction hash is attached
    - automatic channel-close or payout execution can be added later behind the same state model

- Pausable UDT made SSRI feel more practical because it shows script information as an integration surface, not only documentation:
    - off-chain software can discover what a script supports
    - wallet and indexer behavior can be guided by script-provided methods
    - user-facing applications can give better explanations before signing complex transactions

- The Rust shared-state model is useful for application architecture because it encourages clear ownership of mutable state:
    - ledger totals should be derived from stored events and records
    - state transitions should be explicit
    - shared mutable state needs careful boundaries
    - this maps well to payment accounting, withdrawal status, and settlement workflows

### Practical Progress
- Added a merchant Fiber connector Docker image:
    - Created `Dockerfile.fiber-connector`
    - Added a root `docker:build:fiber-connector` script
    - Built and smoke-tested the local connector image
    - Confirmed the container starts the connector entrypoint and fails fast when `LOAVIX_CONNECTOR_TOKEN` is missing

- Updated CI to publish the connector image:
    - Added `ghcr.io/leothatguy/loavix-fiber-connector`
    - Built and pushed connector tags alongside the backend image
    - Added separate Docker build cache scopes for backend and connector images
    - Added pushed-image verification for the connector image
    - Updated merchant-facing docs and setup UI to reference the public connector image

- Added hosted Fiber withdrawal storage:
    - Added `HostedFiberWithdrawalStatus` to the shared package
    - Added the `hosted_fiber_withdrawals` database table to the deploy schema
    - Added the backend `HostedFiberWithdrawalEntity`
    - Registered the withdrawal entity in the backend modules

- Added hosted Fiber withdrawal APIs:
    - Added `/invoices/fiber-account` for hosted balance plus recent withdrawal history
    - Added `/invoices/fiber-withdrawals` for authenticated merchant withdrawal requests
    - Added internal endpoints for processing, paid, failed, and cancelled withdrawal transitions
    - Added validation so withdrawal requests cannot exceed the available hosted Fiber balance

- Updated hosted Fiber account UI:
    - Replaced the withdrawal placeholder with a real withdrawal request action
    - Displayed available hosted Fiber CKB
    - Displayed total hosted Fiber received
    - Displayed paid-out hosted Fiber amount
    - Displayed pending withdrawal amount and recent withdrawal requests

- Local withdrawal flow tested:
    - Applied the local database schema
    - Seeded two hosted Fiber paid invoices worth `350 CKB`
    - Confirmed merchant-owned Fiber and on-chain rows did not increase hosted balance
    - Requested a `100 CKB` hosted Fiber withdrawal
    - Confirmed available balance dropped from `350 CKB` to `250 CKB`
    - Confirmed over-withdrawing returned a `400` response
    - Marked the withdrawal as processing and then paid with a payout transaction hash
    - Confirmed final accounting showed `100 CKB` paid out and `250 CKB` still available
    - Removed the local test rows after verification

- Verification completed:
    - `pnpm --filter @loavix/common build`
    - `pnpm --filter @loavix/backend build`
    - `pnpm --filter @loavix/backend exec jest --runInBand`
    - `pnpm --filter @loavix/scripts build`
    - `pnpm --filter @loavix/frontend build`
    - `pnpm docker:build:fiber-connector`
    - Local backend API withdrawal flow tested with PostgreSQL and Redis
    - Local frontend account route rendered successfully on `http://localhost:3004/account`

### Environment
- CKB network: testnet
- CKB RPC: `https://testnet.ckb.dev`
- Local frontend: `http://localhost:3004`
- Local backend: `http://127.0.0.1:3003`
- Merchant Fiber RPC: `http://127.0.0.1:8227`
- Payer Fiber RPC: `http://127.0.0.1:8229`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- Merchant connector image: `ghcr.io/leothatguy/loavix-fiber-connector`
- JavaScript SDK: CCC
- Backend framework: NestJS
- Frontend framework: Next.js
- Database: PostgreSQL
- Cache/queue: Redis and BullMQ
