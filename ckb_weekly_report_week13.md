## Builder Track Weekly Report - Week 13

**Name:** Divine Oshione Anesi

**Week Ending:** 06-18-2026


### Courses Completed

- Continued the payment-channel track by improving the merchant-owned Fiber node flow in Loavix:
  * Made merchant Fiber connection state clearer in the dashboard and account page
  * Improved connector-token handling and token-regeneration safety
  * Moved payer-facing Fiber payment setup closer to the final user experience
- Began studying RGB++ and Bitcoin-connected asset flows:
  * Reviewed RGB++ as a direction for Bitcoin asset issuance and movement through CKB
  * Started thinking through whether Bitcoin-connected asset flows could apply to future Loavix invoice/payment use cases
- Continued self-study from The Rust Programming Language:
  * Reviewed enums and pattern matching with `match`, `if let`, and exhaustive handling
  * Worked through error-handling patterns using `Result`, `Option`, `panic!`, and recoverable errors
  * Revisited generics, traits, and lifetime annotations to better understand how Rust expresses reusable and memory-safe code
- Began merchant Fiber setup documentation:
  * Clarified the local pieces needed to run Loavix with a merchant Fiber node
  * Identified the connector environment variables a merchant needs to run the connector script
  * Documented the difference between connecting a Fiber node and generating a per-invoice Fiber payment URI

### Key Learnings
- Merchant-owned Fiber support needs to be presented as a merchant setup flow, not as something the payer has to understand:
    - the merchant connects their Fiber node once
    - the connector authenticates to Loavix with a connector token
    - Loavix uses the merchant connector to create Fiber invoices
    - the payer only sees the final Fiber payment URI/QR

- Connector-token rotation needs clear warnings:
    - regenerating the token does not move funds
    - the old token stops working immediately
    - the running connector must be restarted with the new token
    - Fiber invoice generation and status checks are unavailable until the connector is updated

- A registered but offline merchant connector should be shown as offline instead of silently falling back to the hosted Fiber node.

- The payer-facing invoice page should not show a "Prepare Fiber" action:
    - Fiber preparation is a merchant-side invoice-generation step
    - when merchant Fiber is available, the app should create the Fiber payment intent automatically
    - the payer should receive a direct payment page with CKB Action and Fiber payment options

- RGB++ is a separate direction from the current Fiber work, but it may become relevant later if Loavix expands into Bitcoin-connected assets or invoice settlement for non-CKB assets.

- The Rust Book sections on pattern matching, recoverable errors, traits, and lifetimes helped reinforce how Rust encourages explicit control flow and correctness before runtime.

### Practical Progress
- Improved merchant Fiber connector status in the backend:
    - Added registered/active connector status fields for clearer frontend state
    - Returned frontend-friendly connector status aliases
    - Changed merchant Fiber status so a registered but offline connector is visible as offline
    - Added tests for registered/offline connector behavior

- Improved merchant Fiber UX in the account page:
    - Changed the first-time action to generate a connector token
    - Changed the existing-connector action to regenerate the token instead of looking like a first-time connection
    - Added a confirmation dialog before token regeneration
    - Added clearer warnings explaining what token regeneration does
    - Displayed connector status, ready channel count, and node pubkey more clearly

- Improved dashboard Fiber visibility:
    - Added a Fiber status indicator in the dashboard header
    - Showed whether Fiber is locked, connected, ready, offline, or not connected
    - Linked the status indicator to the account page so merchants can find the connector setup flow

- Improved the payer-facing invoice page:
    - Removed the confusing "Prepare Fiber" action from the payer flow
    - Added automatic Fiber payment intent creation when merchant Fiber is available
    - Used the existing Fiber payment URI from payment options when available
    - Renamed "Check Fiber" to "Refresh Fiber payment"
    - Kept the CKB Action/on-chain payment option available beside Fiber

- Tested the local Loavix environment:
    - Started PostgreSQL and Redis through Docker Compose
    - Started the backend locally on `127.0.0.1:3003`
    - Started the frontend locally on `localhost:3000`
    - Started local Fiber merchant and payer nodes
    - Confirmed backend readiness after Fiber started
    - Confirmed merchant Fiber RPC and payer Fiber RPC were reachable locally

- Verification completed:
    - `pnpm --filter @loavix/backend exec jest src/fiber/fiber-connector.service.spec.ts --runInBand`
    - `pnpm --filter @loavix/backend exec jest src/fiber/fiber.service.spec.ts src/invoice/invoice.service.spec.ts --runInBand`
    - `pnpm --filter @loavix/backend build`
    - `pnpm --filter @loavix/frontend build`
    - `git diff --check`

### Environment
- CKB network: testnet
- CKB RPC: `https://testnet.ckb.dev`
- Local frontend: `http://localhost:3000`
- Local backend: `http://127.0.0.1:3003`
- Merchant Fiber RPC: `http://127.0.0.1:8227`
- Payer Fiber RPC: `http://127.0.0.1:8229`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- JavaScript SDK: CCC
- Backend framework: NestJS
- Frontend framework: Next.js
- Database: PostgreSQL
- Cache/queue: Redis and BullMQ
