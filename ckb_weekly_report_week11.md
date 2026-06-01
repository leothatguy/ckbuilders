# Builder Track Weekly Report - Week 11

**Name:** Divine Oshione Anesi

**Week Ending:** 06-01-2026

---

## Courses & Topics Completed

- Reviewed the CKBuilder Handbook again to identify the next advanced topics that are relevant to my current Loavix work.
- Studied Nervos DAO and iCKB at a high level:
  - Nervos DAO as the mechanism for locking CKBytes and receiving compensation from secondary issuance
  - The idea of locked CKB versus liquid/available CKB
  - iCKB as a protocol for making Nervos DAO positions liquid
- Reviewed Script-Sourced Rich Information (SSRI):
  - SSRI as a way to bind useful information and conventions directly to a CKB script
  - How SSRI can help off-chain applications understand a script's behavior
  - Why script-level metadata may be useful for more advanced dApps

---

## Key Learnings

- Nervos DAO and iCKB are especially relevant now because Loavix is moving toward withdrawal and liquidity accounting. I need to clearly distinguish between CKB that is available immediately and CKB that is locked or held in another settlement layer.
- Fiber payments are similar in this sense: when an invoice is paid through Fiber, the merchant has received value, but the value is inside Fiber channel liquidity until it is withdrawn or settled back to L1.
- SSRI could become useful later if Loavix needs richer script descriptions for wallets, indexers, or frontend clients.
- For the current Loavix architecture, the UI must clearly separate:

```txt
Wallet CKB        -> regular on-chain wallet balance
Hosted Fiber CKB  -> paid invoice value held in Loavix/Fiber channels
```

- A Fiber invoice payment should be treated as paid once the Fiber node proves the invoice was paid, but withdrawal needs a separate accounting flow so users cannot withdraw the same balance twice.

---

## Practical Progress

### Hosted Testnet Deployment

This week I moved Loavix from local-only testing to a hosted testnet environment.

- Set up AWS EC2 hosting for the backend and Fiber node.
- Added Docker-based deployment for:
  - NestJS backend
  - PostgreSQL
  - Redis
  - Fiber node
  - Nginx reverse proxy
- Added CI/CD using GitHub Actions, Terraform, Ansible, Docker, and GHCR.
- Configured the hosted backend at:

```txt
https://loavix.duckdns.org
```

- Added and verified a `/ready` endpoint for deployment health checks.
- Fixed production database initialization so the hosted backend creates the invoice schema correctly.
- Cleaned up the hosted testnet environment setup by consolidating runtime variables and separating only the values that need to remain standalone.

Confirmed hosted readiness:

```txt
status: ready
network: testnet
fiber: ok
```

### Fiber Payment Improvements

- Added backend Fiber channel readiness checks before offering Fiber payment.
- Added Fiber payment QR support on the invoice payment page.
- Improved handling for expired and invalid Fiber payment links.
- Removed the confusing "Open URI" action since the browser currently has no native Fiber wallet handler.
- Added better loading states for the invoice detail page.
- Confirmed that real `fibt...` invoices can be paid from a separate payer Fiber node.
- Added helper scripts for repeatable Fiber testing:
  - paying a Fiber invoice from the payer node
  - opening a Fiber channel
  - running a Fiber payment smoke test

### Wallet Auth and Dashboard UX

- Improved wallet sign-in behavior so the app does not repeatedly trigger wallet prompts after reconnecting or refreshing.
- Moved the sign-in action into the navbar and changed auth feedback to toast messages.
- Improved the invoice dashboard with:
  - cleaner row actions
  - share/copy actions
  - a better refresh icon interaction
  - scrolling behavior for larger invoice lists
- Added better invalid invoice handling so wrong invoice ids and pasted Fiber payment URIs do not produce a blank page.

### Settlement Safety

- Improved settlement race handling between on-chain payment and Fiber payment.
- Added protection so Fiber settlement cannot overwrite an invoice already settling on-chain.
- Added protection so on-chain settlement cannot overwrite an invoice already paid through Fiber.
- Added idempotency checks for repeated settlement attachment.
- Added a smoke script to test settlement race behavior.

### Invoice Expiry

- Added backend expiry handling for overdue issued invoices.
- Added Fiber invoice expiry handling so expired Fiber payment requests can be detected and regenerated.
- Kept expired invoices from being treated as active payable invoices.

### Hosted Fiber Balance

- Changed Loavix so Fiber-paid invoices become:

```txt
status: PAID
settlementMethod: FIBER
```

- Added a hosted Fiber balance endpoint:

```txt
GET /invoices/fiber-balance
```

- Added an account page that separates:
  - wallet/on-chain CKB balance
  - hosted Fiber CKB balance
  - paid Fiber invoice count
  - withdrawal placeholder
- Improved wallet balance loading so the UI shows the last known wallet state immediately while refreshing in the background.

### Fiber Payment Test

- Funded the payer Fiber node on CKB testnet.
- Opened additional payer-to-merchant Fiber channels to provide enough single-channel liquidity.
- Paid a real 202 CKB Fiber invoice.
- Confirmed payer-side payment result:

```txt
payment_hash: 0x63b9cacea231a7061c9236d9eca5810f4e3278993990d62713636014ee33024d
status: Success
fee: 0
```

---

## Next Steps

### Phase 2

- Start implementing hosted Fiber withdrawal:
  - withdrawal request endpoint
  - balance reservation
  - withdrawal status tracking
  - account-page withdrawal form
- Add proper ledger-style accounting so Fiber credits and withdrawal debits are tracked safely.
- Continue studying Nervos DAO/iCKB and SSRI to understand how liquidity and script metadata could apply to future Loavix features.

### Phase 3

- Allow merchants to connect their own Fiber node to Loavix.
- Let the site use the merchant's connected Fiber node to generate the Fiber payment URI for each invoice.
- Add merchant-side setup instructions for:
  - running a Fiber node
  - connecting it to Loavix
  - checking channel readiness
  - generating invoices from their own node instead of the hosted Loavix node

---

## Environment

- CKB network: testnet
- CKB RPC: `https://testnet.ckb.dev`
- Hosted backend: `https://loavix.duckdns.org`
- Frontend hosting: Vercel
- Backend hosting: AWS EC2
- Fiber Docker image: `nervos/fiber:0.9.0-rc1`
- JavaScript SDK: CCC
- Backend framework: NestJS
- Frontend framework: Next.js
- Deployment tooling: Docker, Terraform, Ansible, GitHub Actions, GHCR
