## Builder Track Weekly Report — Week 12

**Name:** Divine Oshione Anesi

**Week Ending:** 06-09-2026


### Courses Completed

- Continued the payment-channel track by working on the next Loavix Fiber architecture step:
  * Moving from hosted-only Fiber payments toward merchant-owned Fiber node support
  * Connecting merchant Fiber node status to the backend and account UI
  * Keeping invoice generation and payment-status checks routed through the correct merchant node
- Continued studying advanced CKB payment and liquidity concepts that apply directly to Loavix:
  * Fiber node ownership and hosted payment architecture
  * Merchant-side payment-channel readiness
  * Fiber invoice ownership and node-pubkey matching

### Key Learnings
- For Loavix, merchant-owned Fiber nodes need a different architecture from the hosted node model:
    - the backend should not expose a merchant's FNN RPC publicly
    - the merchant connector should poll the backend for commands
    - the backend should track node availability, ready channels, and node pubkey
    - each Fiber invoice intent should remember the node pubkey that created it

- Storing the Fiber node pubkey on each invoice is important because an existing Fiber payment intent should not be checked or reused through a different node.

### Practical Progress
- Added the first merchant Fiber connector flow for Loavix:
    - Added backend connector endpoints for heartbeat, command polling, and command result submission
    - Added connector token protection with `FIBER_CONNECTOR_TOKEN`
    - Added heartbeat TTL and command timeout configuration
    - Added merchant-address scoped connector state
    - Added support for connector-driven `new_invoice` and `get_invoice` Fiber RPC commands
    - Added node-pubkey matching checks with `FIBER_EXPECTED_NODE_PUBKEY`

- Improved Fiber invoice ownership tracking:
    - Added `fiberNodePubkey` to invoice data
    - Returned node pubkey in Fiber payment options and intent responses
    - Prevented an existing Fiber intent from being reused when the connected merchant node does not match the original node
    - Updated Fiber sync so it can check invoice status through either the hosted Fiber node or the active merchant connector

- Improved the account page:
    - Added connected Fiber node status
    - Showed whether the merchant node is ready or offline
    - Displayed the connected node pubkey when available
    - Kept hosted Fiber balance separate from wallet CKB balance

- Added a connector runner script:
    - `pnpm run fiber-connector`
    - The script sends heartbeats to Loavix, polls for backend commands, executes them against local FNN RPC, and posts results back to the backend

- Verification completed:
    - `pnpm --filter @loavix/backend exec jest --runInBand`
    - `pnpm --filter @loavix/backend build`
    - `pnpm --filter @loavix/scripts build`
    - `pnpm --filter @loavix/frontend build`

### Environment
- CKB network: testnet
- CKB RPC: `https://testnet.ckb.dev`
- Hosted backend: `https://loavix.duckdns.org`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- JavaScript SDK: CCC
- Backend framework: NestJS
- Frontend framework: Next.js
- Database: PostgreSQL
- Cache/queue: Redis and BullMQ

