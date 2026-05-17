# Builder Track Weekly Report - Week 9

**Name:** Divine Oshione Anesi

**Week Ending:** 05-16-2026

---

## Courses & Topics Applied

- Applied CCC SDK concepts to replace the remaining Lumos-based development flow:
  - Migrated backend and scripts away from Lumos to CCC for local offckb devnet transactions
  - Reused offckb `system-scripts.json` so CCC clients resolve the correct devnet script outpoints
  - Added PWLock mapping for MetaMask-derived CKB accounts on local devnet
- Applied dApp architecture improvements:
  - Moved the invoice issuance path toward wallet-owned signing
  - Kept the backend as an indexer/API helper instead of the sole transaction signer
  - Added a backend verification step before accepting wallet-issued invoice cells
- Applied API documentation and testing practices:
  - Added Swagger/OpenAPI docs for backend routes
  - Added an end-to-end backend smoke test for invoice lifecycle verification

---

## Key Learnings

- A dApp should not depend on a backend-owned private key for its primary transaction path. The better architecture is for the frontend wallet to sign the CKB transaction, while the backend verifies and indexes chain-derived state.
- CCC client configuration matters. Public testnet/mainnet clients produce the familiar wallet-derived address, while an offckb devnet client can produce a different address because devnet script deployments differ.
- MetaMask integration through CCC uses PWLock, so local devnet support requires the PWLock script info to be present in the CCC script map.
- Backend state should be treated as a cache/index of verifiable chain state. For wallet-issued invoices, the backend now checks the live cell data before marking an invoice as `ISSUED`.
- Swagger is useful for validating the API surface while the frontend is evolving, especially for routes that bridge off-chain invoice records and on-chain CKB cells.

---

## Practical Progress

### Sprint 2 - Backend Invoice Lifecycle, CCC Migration, and dApp UI

This week I moved Loavix from a CLI/backend prototype into a working backend + wallet-facing dApp flow.

#### Backend Invoice Lifecycle

- Implemented backend invoice lifecycle services using NestJS, TypeORM, BullMQ, Redis, and PostgreSQL
- Added API support for:
  - creating draft invoices
  - listing invoices by merchant address
  - issuing invoices on-chain
  - cancelling issued invoice cells
  - settling issued invoices
- Added queue-backed chain workers for:
  - `CREATE_INVOICE_CELL`
  - `UNLOCK_INVOICE_CELL`
  - `CANCEL_INVOICE_CELL`
- Fixed cancellation behavior so issued cells can be cancelled correctly
- Added indexer polling to detect invoice cell creation and consumption on the local devnet

#### Lumos to CCC Migration

- Removed Lumos usage from the active Loavix app flow
- Replaced backend and script transaction clients with CCC
- Shared offckb devnet system scripts through `packages/common/src/system-scripts.json`
- Added CCC script mapping for:
  - Secp256k1 sighash
  - Multisig
  - AnyoneCanPay
  - OmniLock
  - xUDT
  - NervosDAO
  - Type ID
  - PWLock
- Verified the CCC-only devnet transaction path with a test transfer and backend invoice lifecycle

#### Swagger / OpenAPI

- Added `@nestjs/swagger` and `swagger-ui-express`
- Exposed Swagger UI at:

```bash
http://127.0.0.1:3000/docs
```

- Exposed OpenAPI JSON at:

```bash
http://127.0.0.1:3000/docs-json
```

- Documented invoice and chain routes, including:
  - `POST /invoices`
  - `GET /invoices`
  - `GET /invoices/:id`
  - `PATCH /invoices/:id/issue`
  - `POST /invoices/:id/issued`
  - `POST /invoices/:id/cancel`
  - `POST /internal/invoices/:id/settle`
  - `GET /chain/config`

#### Backend Smoke Test

- Added a backend invoice smoke script:

```bash
pnpm run smoke:backend-invoice
```

- The smoke test verifies the full flow:
  - create invoice
  - issue invoice cell
  - wait for `ISSUED`
  - settle invoice
  - wait for `PAID`

- Verified a successful smoke run:

<img width="1361" height="740" alt="image" src="https://github.com/user-attachments/assets/a815df55-9468-4649-9be1-0653f1a5eae0" />


#### Wallet-Signed dApp Flow

- Replaced the starter Next.js page with a Loavix invoice dApp UI
- Added CCC React provider support for wallet connection
- Added env-driven frontend CKB network selection:

```bash
NEXT_PUBLIC_CKB_NETWORK=devnet
NEXT_PUBLIC_CKB_RPC_URL=http://127.0.0.1:28114
NEXT_PUBLIC_API_URL=http://127.0.0.1:3000
```

- Added frontend support to:
  - connect wallet through CCC
  - show wallet balance
  - show connected wallet address in the navbar
  - create a draft invoice
  - build a CKB invoice cell transaction in the browser
  - sign and send the transaction through the connected wallet
  - call the backend to attach the wallet-issued invoice cell after confirmation
- Added backend verification for wallet-issued cells:
  - invoice hash must match
  - amount must match
  - payee lock hash must match
  - outpoint tx hash must match submitted tx hash

#### Frontend UI

- Built the first Loavix dApp interface with:
  - wallet navbar
  - balance display
  - invoice creation form
  - invoice list
  - wallet refresh action

<img width="1361" height="740" alt="image" src="https://github.com/user-attachments/assets/8b015b13-9576-41c7-8f10-112e01d1ba96" />

---

## Verification

Commands run successfully:

```bash
pnpm --filter @loavix/backend run build
pnpm --filter @loavix/backend exec jest --runInBand
pnpm --filter @loavix/frontend run lint
pnpm --filter @loavix/frontend run build
pnpm run smoke:backend-invoice
```


---

## Environment

- Node.js (v22.13.0)
- pnpm (10.29.2)
- Docker (29.2.0)
- PostgreSQL + Redis
- offckb (0.4.4)
- CKB local devnet RPC: `http://127.0.0.1:28114`
- Backend API: `http://127.0.0.1:3000`
- Frontend dApp: `http://127.0.0.1:3001`
- JavaScript SDK: CCC (≥1.12.5)
- Backend framework: NestJS
- Frontend framework: Next.js
