## Builder Track Weekly Report - Week 14

**Name:** Divine Oshione Anesi

**Week Ending:** 06-22-2026


### Courses Completed

- Continued studying RGB++ and Bitcoin-connected asset flows:
  * Reviewed RGB++ as a direction for moving Bitcoin-connected assets through CKB
  * Started comparing RGB++ with the current Loavix payment architecture
  * Considered where Bitcoin-connected asset settlement could fit into future invoice flows
- Started studying Molecule and CKB serialization:
  * Reviewed why CKB applications need strict binary serialization
  * Connected serialization concepts to cell data, script args, and transaction structures
  * Started mapping how structured invoice data should be encoded and decoded safely
- Continued self-study from The Rust Programming Language:
  * Reviewed package, crate, and module organization for larger Rust projects
  * Continued practicing Rust test structure and error-driven development
  * Revisited iterators and closures as tools for writing clearer Rust data-processing code
- Completed merchant Fiber setup documentation:
  * Documented how a merchant connects their own Fiber node to Loavix
  * Documented connector-token generation and regeneration behavior
  * Documented the connector runner environment variables and local checks
- Implemented the two Fiber receiving flows in Loavix:
  * Separated hosted Fiber payments from merchant-owned connector payments
  * Added accounting rules so hosted Fiber balance only counts hosted receipts
  * Updated invoice and dashboard labels to show whether Fiber settlement used hosted Fiber or merchant Fiber

### Key Learnings
- Molecule-style serialization matters because CKB scripts and applications cannot rely on loose JSON-like data when validating state on-chain:
    - cell data must be structured predictably
    - script args need stable encoding
    - off-chain services and on-chain scripts must agree on the exact bytes being interpreted

- Merchant Fiber setup needs to be documented separately from general deployment:
    - the hosted backend setup is different from a merchant-owned Fiber node setup
    - the connector token is a merchant authentication secret, not a payer-facing value
    - the merchant runner should stay beside the private Fiber RPC
    - the payer should only receive the final payment URI or QR code

- Hosted Fiber and merchant-owned Fiber need different accounting treatment:
    - hosted Fiber payments are received by Loavix and should increase the merchant hosted balance
    - merchant-owned connector payments are received by the merchant's own Fiber node
    - merchant-owned payments should mark the invoice as paid without increasing Loavix-held hosted balance
    - Fiber status polling must check the same receiver that created the payment intent

- RGB++ is still a separate research path from the current Fiber work, but it is useful to study because it connects CKB application development with Bitcoin asset flows.

- The Rust Book sections on modules, tests, iterators, and closures helped reinforce how Rust projects stay organized as the codebase grows.

### Practical Progress
- Added dedicated merchant Fiber connector documentation to Loavix:
    - Created `docs/merchant-fiber-connector.md`
    - Explained the connector flow from merchant Fiber node to Loavix backend to payer invoice page
    - Listed the requirements for running a merchant connector
    - Added local and hosted connector command examples
    - Documented optional connector timing and Fiber RPC bearer-token settings

- Clarified connector-token behavior:
    - Explained that connector tokens are shown once
    - Explained that token regeneration invalidates the old token immediately
    - Clarified that regenerating a token does not move funds or change on-chain balances
    - Documented that the connector must be restarted with the new token after regeneration

- Documented merchant and payer responsibilities:
    - Merchant runs the Fiber node and connector
    - Loavix uses the connector to create Fiber payment intents
    - Payer sees the generated Fiber URI or QR code on the invoice page
    - Payer does not need the connector token or direct access to the merchant node

- Updated deployment documentation:
    - Linked the merchant Fiber connector guide from `docs/deployment-environments.md`
    - Kept the existing local Fiber testnet node instructions in place
    - Separated merchant-owned connector setup from general backend Fiber configuration

- Implemented hosted and merchant-owned Fiber receiving flows:
    - Added explicit hosted and merchant connector Fiber settlement markers
    - Ensured hosted Fiber balance only includes hosted Fiber receipts
    - Updated Fiber settlement sync so it checks the receiver that created the payment intent
    - Kept merchant-owned Fiber payments out of Loavix hosted-balance accounting
    - Added payer and dashboard labels for `Hosted Fiber` and `Merchant Fiber`

- Added merchant setup guidance inside the Loavix app:
    - Added a dedicated `/account/fiber-setup` route
    - Added a quick setup dialog from the account Fiber panel
    - Explained how a merchant can connect a Fiber node running on another PC or server
    - Clarified that production merchants should eventually use a packaged connector instead of cloning the source repo

- Verification completed:
    - Checked the new documentation for long dash characters
    - Ran `git diff --check` on the updated documentation files
    - Ran focused backend Fiber and invoice service tests
    - Built the shared package, backend, and frontend successfully
    - Pushed the latest Loavix implementation commits to GitHub

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
