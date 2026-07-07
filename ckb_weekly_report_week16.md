## Builder Track Weekly Report - Week 16

**Name:** Divine Oshione Anesi

**Week Ending:** 07-06-2026


### Courses Completed

- Studied xUDT extensibility and token policy design:
  * Compared sUDT's minimal fungible-token validation model with xUDT's extension-script approach
  * Looked at how custom validation can support minting rules, governance constraints, or application-specific transfer policy
  * Connected this to future invoice assets where a payment token may need rules beyond a simple balance transfer
- Continued Rust self-study with a focus on concurrency and worker boundaries:
  * Reviewed thread spawning, message passing with channels, and shared state with `Mutex` and `Arc`
  * Looked at how Rust's ownership model prevents accidental sharing of mutable worker state
  * Connected those patterns to payout workers, retry loops, and settlement jobs that must avoid duplicate state changes
- Finalized automatic hosted Fiber payout behavior for Loavix:
  * Added a hosted Fiber payout worker for turning paid hosted Fiber invoices into on-chain CKB payouts
  * Added payout fee, minimum payout, batch size, interval, and payout-key configuration
  * Added hosted Fiber treasury readiness reporting for checking whether automatic payouts can run safely
- Added merchant reclaim support for Fiber-paid invoice deposits:
  * Added backend tracking for invoice deposit reclaim transactions
  * Added merchant-facing reclaim actions in the dashboard and invoice detail page
  * Added the next implementation plan for payment-method selection and invoices that do not always need an on-chain cell
- Hardened merchant Fiber and hosted Fiber operations:
  * Exposed hosted node public routing readiness metadata
  * Raised connector proxy limits and returned clearer throttling status
  * Reworked account setup to present a Docker-only merchant connector path

### Key Learnings
- Hosted Fiber payouts need more than a simple balance update:
    - a paid Fiber invoice should queue a payout record with a clear destination wallet
    - payout fee calculation needs to happen before the amount is presented as net received
    - a payout worker should mark broadcast and confirmed states carefully so a retry does not double-pay
    - treasury readiness should be visible before relying on automatic payouts in a hosted environment

- Fiber payments can leave the on-chain invoice cell deposit locked if the invoice was created with both payment paths:
    - the Fiber payment can settle the invoice without spending the on-chain invoice cell
    - the merchant still needs a wallet-signed reclaim transaction to unlock that cell capacity
    - reclaiming must remain a merchant-confirmed wallet action because the server should not silently spend the merchant's cell
    - payment-method selection can later reduce this by avoiding on-chain cells for Fiber-only invoices

- Payment-channel UX depends heavily on operational readiness:
    - the hosted node must be publicly reachable for route discovery
    - the payer needs enough outbound liquidity before a Fiber payment can succeed
    - connector throttling should return a clear status instead of looking like a broken payment flow
    - merchant setup should guide users toward the production connector path, not a source-code workflow

- xUDT is useful to study because it shows how token behavior can become script-composed instead of fixed:
    - simple fungible token transfers are only one model
    - extension scripts can define additional policy
    - richer invoice assets may eventually need rules around minting, redemption, or transfer restrictions

- Rust concurrency reinforces the same ideas needed in payment infrastructure:
    - worker ownership should be explicit
    - shared state should have narrow boundaries
    - message passing is useful when a background task should process one job at a time
    - payout and settlement code should make duplicate processing difficult by design

### Practical Progress
- Added automatic hosted Fiber payout infrastructure:
    - Added hosted Fiber payout configuration for enabling the worker, choosing a payout key, setting fees, and setting minimum payout amounts
    - Added a hosted Fiber payout worker that groups pending payouts by destination address
    - Added broadcast and paid handling around payout transaction hashes
    - Added retry support for failed payout records that do not already have a payout transaction
    - Added hosted Fiber treasury status reporting for backend readiness checks

- Added hosted Fiber payout testing support:
    - Added `smoke-hosted-fiber-payout.ts`
    - Covered the hosted Fiber invoice to payout flow from authenticated merchant invoice creation through Fiber payment and payout confirmation
    - Added backend tests around payout states, treasury readiness, retry behavior, and payout-key fallback behavior
    - Kept the payout logic separate from merchant-owned Fiber receipts

- Added merchant reclaim flow for Fiber invoice deposits:
    - Added `depositReclaimTxHash` and `depositReclaimedAt` to invoice storage
    - Added `/invoices/:id/deposit-reclaimed` for attaching a confirmed reclaim transaction
    - Verified that the reclaim transaction spends the invoice cell
    - Verified that the reclaim transaction returns the invoice cell capacity to the merchant lock
    - Added idempotent handling for repeated reclaim attachment with the same transaction hash

- Updated payer and merchant UI around reclaim:
    - Added a merchant invoice-detail panel for reclaiming on-chain invoice cell capacity
    - Added dashboard reclaim actions for eligible Fiber-paid invoices
    - Displayed deposit reclaim state so reclaimed invoices do not keep showing as locked
    - Added the next implementation plan for payment-method selection after the reclaim MVP is stable

- Improved hosted Fiber readiness and connector behavior:
    - Exposed hosted node public routing readiness metadata through backend readiness output
    - Fixed hosted Fiber route visibility issues during testnet payment testing
    - Raised connector proxy request limits
    - Returned clearer throttling status for connector-heavy polling

- Refined the account and merchant setup UI:
    - Added a shared `AppHeader` component and reused it across dashboard, account, setup, and invoice pages
    - Redesigned the account page into a compact settings-style layout
    - Removed the manual hosted Fiber withdrawal action because payouts now queue automatically
    - Kept automatic payout history visible even when there are no records yet
    - Changed the Fiber setup guide into a modal
    - Made the setup flow Docker-only for production merchants
    - Removed development-only setup instructions from the frontend
    - Removed glow effects from status dots so status indicators are plain and less visually noisy

- Hosted Fiber payment testing:
    - Paid hosted Fiber invoices from a local payer Fiber node against the hosted Loavix frontend and API
    - Confirmed the role of payer outbound liquidity before a larger payment could succeed
    - Confirmed that the hosted invoice page could present the Fiber payment URI and track the paid state
    - Used the payment tests to identify the need for reclaiming on-chain invoice cell capacity after Fiber settlement

- Verification completed:
    - `pnpm --filter @loavix/frontend lint`
    - `git diff --check`
    - Hosted backend readiness checked after Fiber routing updates
    - Hosted Fiber invoice payment tested from a local payer Fiber node
    - Latest Loavix frontend changes pushed to GitHub

### Environment
- CKB network: testnet
- CKB RPC: `https://testnet.ckb.dev`
- Hosted frontend: `https://loavix-frontend.vercel.app`
- Hosted backend: `https://loavix.duckdns.org`
- Local merchant Fiber RPC: `http://127.0.0.1:8227`
- Local payer Fiber RPC: `http://127.0.0.1:8229`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- Merchant connector image: `ghcr.io/leothatguy/loavix-fiber-connector:latest`
- JavaScript SDK: CCC
- Backend framework: NestJS
- Frontend framework: Next.js
- Database: PostgreSQL
- Cache/queue: Redis and BullMQ
