# Builder Track Weekly Report - Week 8

**Name:** Divine Oshione Anesi

**Week Ending:** 05-09-2026

---

## Courses Completed

- Applied ckb-testtool for integration testing of custom scripts:
  - [ckb-testtool : builtin contracts](https://docs.rs/ckb-testtool/latest/ckb_testtool/builtin/index.html)
    - Used `builtin::ALWAYS_SUCCESS` to isolate type script testing from lock script logic
    - Learned the difference between `deploy_cell()` (raw bytes) and `deploy_cell_by_name()` (filesystem lookup in `build/release/`)
  - [ckb-testtool : Context](https://docs.rs/ckb-testtool/latest/ckb_testtool/context/struct.Context.html)
    - Used `context.create_cell()` and `context.create_cell_with_out_point()` for deterministic test cell setup
    - Verified both success and failure paths using `context.verify_tx()` with `MAX_CYCLES` limits
- Studied on-chain script deployment and cell dep resolution:
  - [CKB Cell Deps](https://docs.nervos.org/docs/reference/transaction#cell-deps)
    - Understood how `data2` hash type works `code_hash = blake2b(cell_data)` with CKB's `ckb-default-hash` personalization
    - Learned the cell dep resolution flow: CKB VM loads the script binary from the referenced cell dep's data field
    - Resolved `ScriptNotFound` errors by correctly wiring the cell dep outpoint after script deployment
- Learned low-level CKB transaction construction with Lumos:
  - [Lumos TransactionSkeleton](https://lumos-website.vercel.app/api/modules/helpers.html)
    - Discovered that high-level helpers (`commons.common.transfer`, `injectCapacity`) overwrite manually-set output cell data, they are designed for simple CKB transfers, not custom cell creation
    - Built transactions using direct indexer cell collection (`indexer.collector()`) + explicit output/input construction
    - Manually serialized WitnessArgs in molecule format (85-byte buffer) since `@ckb-lumos/codec` is not re-exported from the main `@ckb-lumos/lumos` package

---

## Key Learnings

- CKB type scripts are invoked during **both** cell creation (output) and cell destruction (input). On creation, `Source::GroupInput` is empty the script must detect this case and return success, or it will fail with `IndexOutOfBound`. This was a critical bug discovered during devnet testing.
- Lumos's `commons.common.transfer()` and `injectCapacity()` manage outputs internally and will silently overwrite custom cell data, type scripts, and capacity. For creating cells with data (e.g., deploying script binaries or creating invoice cells), low-level transaction construction via the indexer is required.
- The `secp256k1_blake160` lock script requires a WitnessArgs molecule placeholder (not raw bytes) in the first witness of each lock group. Without it, `prepareSigningEntries` cannot compute the signing message. Multi-party transactions need one placeholder per unique lock group.
- CKB devnet fee rate is 1000 shannons/KW. A deploy transaction carrying a 45KB script binary needs ~46,000 shannons minimum fee significantly more than the typical 10,000 for a simple transfer.
- The `ckb-testtool` `builtin::ALWAYS_SUCCESS` script is an effective isolation tool: by using it as the lock script, type script validation logic can be tested independently without worrying about signature verification.

---

## Practical Progress

### Sprint 1 Completion - Invoice Script Testing & Devnet Lifecycle

This week I completed Sprint 1 of the Loavix CKB on-chain invoice system: comprehensive unit tests, a critical type script fix, full devnet deployment, and CLI tooling for the invoice cell lifecycle.

#### Unit Tests (7/7 passing)

Wrote 7 `ckb-testtool` integration tests covering every branch of the invoice script:

| Test | Scenario | Validates |
|------|----------|-----------|
| `test_valid_payment_unlock` | Output to payee ≥ amount | Payment verification logic |
| `test_underpayment_fails` | Output to payee < amount | `InsufficientPayment` error (code 4) |
| `test_payment_to_wrong_address_fails` | Output to wrong lock hash | Payee lock hash matching |
| `test_valid_cancellation_unlock` | Payee input cell present | Cancellation authorization |
| `test_cancellation_wrong_signer_fails` | No payee input cell | `InvalidSignature` error (code 5) |
| `test_invalid_invoice_data_fails` | Data ≠ 80 bytes | `InvalidInvoiceData` error (code 3) |
| `test_overpayment_succeeds` | Output to payee > amount | Overpayment acceptance |

#### Contract Bug Fix

Fixed a critical bug where the type script crashed with `IndexOutOfBound` (error code 1) during cell creation. The script was unconditionally calling `load_cell_data(0, Source::GroupInput)`, but on creation there are no input cells in the script group. Added a match arm to detect the creation case and return `Ok(())`.

#### On-Chain Devnet Verification

Successfully deployed the invoice script and ran the full lifecycle on the local offckb devnet:

- **Deploy:** Script binary (45KB) deployed as a data cell
- **Create:** 200 CKB invoice cell created with type script + 80-byte invoice data
- **Settle:** Payment unlock transaction executed, 200 CKB transferred to payee

#### CLI Scripts

| Script | Purpose |
|--------|---------|
| `devnet-config.ts` | Shared configuration: accounts, system script outpoints, helpers |
| `create-invoice-cell.ts` | Deploys script binary + creates invoice cell on devnet |
| `settle-invoice-cell.ts` | Reads invoice data, builds payment unlock tx, settles on-chain |

#### Commands

```bash
# Build contract + run unit tests
cd loavix-scripts && make build && cargo test

# Full devnet lifecycle
pnpm run create-invoice-cell
pnpm run settle-invoice-cell -- <txHash> 0x0
```
<img width="569" height="487" alt="image" src="https://github.com/user-attachments/assets/8cb1a9fc-22d4-4bcf-ac4d-6fe2de76a7b6" />
<img width="594" height="456" alt="image" src="https://github.com/user-attachments/assets/b550f54e-b485-49b7-8237-e4444f3668f5" />

<img width="1200" height="486" alt="image" src="https://github.com/user-attachments/assets/72e53d89-b4c5-4ee0-94ca-3ebaed7fee6a" />
<img width="1200" height="433" alt="image" src="https://github.com/user-attachments/assets/07d2b95a-7f13-4a18-8d25-acbacbbe3da3" />

---

## Environment

- Node.js (v22.13.0)
- pnpm (10.29.2)
- Rust (1.94.0) + RISC-V target (`riscv64imac-unknown-none-elf`)
- ckb-testtool (1.1.1)
- Docker (29.2.0)
- offckb (0.4.4)
- clang (18.1.8)
- JavaScript SDK: Lumos (0.23.0), CCC (≥1.12.5)
