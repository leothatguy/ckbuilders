## Builder Track Weekly Report - Week 17

**Name:** Divine Oshione Anesi

**Week Ending:** 07-14-2026


### Courses Completed

- Studied reusable payment offers as a protocol layer above Fiber invoices:
  * Distinguished a stable signed offer from the fresh one-time Fiber invoice created for each payment attempt
  * Reviewed how fixed amounts, payer-entered amounts, single-use rules, reusable offers, expiry, and recurrence policies affect validation
  * Learned how payment links, Fiber Addresses, encoded `fbroffer1...` payloads, and `fibt...` invoices serve different parts of the payment flow
- Continued Rust self-study with a focus on asynchronous programming and task lifecycle design:
  * Reviewed how `Future`, `async`, and `.await` represent work that can pause without blocking a thread
  * Looked at the difference between concurrency, parallelism, and asynchronous I/O
  * Considered how ownership, `Send` bounds, cancellation, and explicit task state help prevent background work from being lost or duplicated
- Built Fiber Offers as an independent hackathon project:
  * Implemented a signed offer protocol, resolver, CLI, SDK, merchant dashboard, payer pages, and integrated documentation
  * Connected the resolver to real Fiber node RPC methods for fresh invoice creation and settlement lookup
  * Supported CKB, UDT, and RGB++ asset descriptors at the protocol level
- Studied production resolver architecture for payment systems:
  * Used PostgreSQL as authoritative shared storage for offers, resolutions, idempotency records, webhooks, and settlement state
  * Used Redis for distributed rate limiting and BullMQ queues
  * Separated HTTP resolver replicas from settlement and webhook background workers
- Deployed and tested the project on public infrastructure:
  * Provisioned a dedicated AWS host with Terraform
  * Deployed PostgreSQL, Redis, resolver replicas, a worker, Nginx, and a live Fiber node through Docker Compose
  * Configured DNS and HTTPS for the public hackathon deployment

### Key Learnings
- A reusable Fiber offer should not be treated as a reusable Fiber invoice:
    - a Fiber invoice is a one-time payment request
    - a stable offer describes the rules under which new invoices may be created
    - every offer resolution should create a fresh invoice and payment hash
    - the resolver tracks the relationship between the stable offer, resolution attempt, invoice, settlement, receipt, and webhook event

- Offer verification and Fiber node identity require separate proofs:
    - the Fiber node RPC does not expose arbitrary application-message signing for offer documents
    - an Ed25519 lifecycle key can sign offers and revocations independently
    - the signed offer can bind the expected Fiber node public key
    - a live `node_info` check proves that the resolver is connected to the expected merchant node before creating invoices

- Production payment coordination needs database-level concurrency protection:
    - two resolver replicas must not consume the same single-use offer twice
    - idempotency keys need authoritative uniqueness rather than process-local checks
    - row locks and transactions protect recurrence limits and single-use state
    - background settlement and webhook work should survive resolver restarts and replica changes

- Public Fiber reachability is different from payment readiness:
    - a connected peer does not guarantee a route
    - a route does not guarantee enough capacity
    - total channel capacity does not equal inbound liquidity
    - reserves reduce immediately usable liquidity
    - durable production payment acceptance requires an always-on liquidity counterparty

- Rust asynchronous programming reinforces the same worker-design principles:
    - every task should own or borrow its state deliberately
    - cancellation should leave persistent state recoverable
    - retries should be driven by durable state rather than assumptions about an earlier task
    - task boundaries should make duplicate settlement or webhook delivery difficult

- Fiber Offers and Loavix solve related but different payment problems:
    - Loavix creates and tracks individual merchant invoices with on-chain and Fiber settlement options
    - Fiber Offers creates stable reusable payment identities that resolve into new Fiber invoices
    - the reusable offer model could inform a future Loavix product or donation-link feature without replacing Loavix invoice accounting
    - keeping the two models separate makes their custody, settlement, and lifecycle responsibilities clearer

### Practical Progress
- Implemented the Fiber Offers protocol package:
    - Added canonical serialization and deterministic offer identifiers
    - Added Ed25519 signing and verification for offers and revocations
    - Added Bech32m `fbroffer1...` encoding and decoding
    - Added fixed and flexible amount validation using integer base units
    - Added reusable, single-use, expiry, recurrence, revocation, CKB, UDT, and RGB++ rules

- Built the resolver and real Fiber integration:
    - Added offer registration, lookup, resolution, revocation, receipts, and payment history APIs
    - Added Fiber Address discovery through `/.well-known/fiberoffer/<username>`
    - Added strict Fiber RPC method allowlisting
    - Added `node_info` identity checks before using a merchant Fiber node
    - Added real `new_invoice` and `get_invoice` calls for invoice creation and settlement synchronization
    - Added signed webhooks with encrypted secrets, queued delivery, retries, and event logs

- Built merchant and developer tooling:
    - Added CLI initialization, lifecycle-key generation, diagnostics, offer creation, registration, listing, inspection, and revocation
    - Added a `doctor` command for resolver reachability and Fiber node identity checks
    - Preserved generated identity and offer data when registration failed so merchants could retry safely
    - Added SDK clients for merchant workflows, payer workflows, direct Fiber payment adapters, recurrence, diagnostics, and payment readiness
    - Added browser, React, React Native, and Node-specific exports with TypeScript declarations

- Added horizontally safe production storage and workers:
    - Added PostgreSQL migrations and transaction-aware storage
    - Added uniqueness constraints, row locks, and atomic single-use offer consumption
    - Added Redis-backed rate limiting shared across resolver replicas
    - Added BullMQ settlement and webhook queues
    - Ran two resolver replicas behind Nginx with a separate queue worker

- Built the merchant dashboard, payer flow, and documentation:
    - Added offer creation, inventory, revocation, payments, receipts, webhooks, integrations, and diagnostics views
    - Added stable payment links, QR codes, encoded offers, and Fiber Address presentation
    - Displayed CKB to users while keeping protocol and API calculations in shannons
    - Added public payer pages that resolve offers into fresh copyable Fiber invoices
    - Added responsive documentation covering concepts, CLI, SDK, Fiber connectivity, APIs, production deployment, and self-hosting
    - Fixed QR overflow, toast layering above modals, mobile documentation navigation, and a false hosted `Offline` state caused by authentication startup order

- Deployed the hackathon project independently:
    - Added project-scoped Terraform for AWS networking, compute, and security rules
    - Added a local Terraform workflow that keeps state, plans, credentials, SSH keys, and environment files out of Git
    - Deployed PostgreSQL, Redis, migrations, two resolvers, a worker, a gateway, and a live Fiber node
    - Configured `fiber-offers.leothatguy.me` with Nginx and Let's Encrypt HTTPS
    - Kept the Fiber RPC private while exposing the required Fiber peer-to-peer port

- Reused and extended Loavix groundwork without coupling the deployments:
    - Used the Loavix local merchant and payer Fiber nodes for the first real resolver integration
    - Reused lessons from the Loavix Terraform and server deployment structure while creating separate Fiber Offers infrastructure
    - Kept the Loavix and Fiber Offers production servers independent
    - Used the contrast between Loavix invoices and Fiber Offers to clarify one-time invoice lifecycle versus reusable payment intent lifecycle

- Proved real public Fiber payment routing:
    - Opened an additional public channel with 1,000 CKB allocated toward the hosted merchant
    - Confirmed approximately 901 CKB of initial usable inbound liquidity after reserves
    - Paid the hosted merchant from an unrelated payer with no direct merchant channel
    - Confirmed a 1 CKB public multi-hop payment with a 0.001 CKB fee
    - Confirmed merchant invoice status, resolver settlement state, receipt history, and dashboard activity
    - Paid a fresh invoice resolved from the public `coffee@fiber-offers.leothatguy.me` Fiber Address offer

- Verification completed:
    - Ran 111 automated tests with 107 passing and 4 optional infrastructure tests skipped
    - Tested protocol signatures, encoding, validation, recurrence, and revocation
    - Tested SDK flows, resolver authentication, idempotency, single-use races, receipts, webhooks, and CLI recovery
    - Verified PostgreSQL, Redis, resolver replicas, worker startup, DNS, HTTPS, Fiber node identity, fresh invoice creation, and settlement synchronization
    - Checked dashboard and documentation behavior at 390-pixel mobile and 1440-pixel desktop widths
    - Verified a real public multi-hop Fiber payment against the hosted deployment

### Environment
- CKB network: testnet
- Production URL: `https://fiber-offers.leothatguy.me`
- Public Fiber Address: `coffee@fiber-offers.leothatguy.me`
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- Runtime: Node.js 20 or newer
- Protocol signatures: Ed25519
- Portable offer encoding: Bech32m `fbroffer1...`
- Fiber invoice format: `fibt...`
- Database: PostgreSQL 16
- Cache/queue: Redis 7 and BullMQ
- Deployment: AWS EC2, Terraform, Docker Compose, and Nginx
- TLS: Let's Encrypt
- SDK targets: Node.js, browser, React, and React Native
