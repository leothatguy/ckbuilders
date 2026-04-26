# Builder Track Weekly Report - Week 6

**Name:** Divine Oshione Anesi

**Week Ending:** 04-25-2026

---

## Courses Completed

- Completed the "Rust Script API" modules from the Nervos CKB docs:
  - [Rust API Introduction](https://docs.nervos.org/docs/script/rust/rust-api-introduction)
    - Understood the purpose of `ckb-std` in CKB's bare-metal script environment
    - Mapped the API areas: entry/allocator, syscalls, errors, logger, utils, dynamic loading, and spawn
    - Learned when to use high-level wrappers instead of low-level interfaces
  - [Entry & Allocator](https://docs.nervos.org/docs/script/rust/rust-api-entry)
    - Learned why CKB scripts require `#![no_std]` and `#![no_main]`
    - Understood how `ckb_std::entry!` defines the contract entry point
    - Learned `ckb_std::default_alloc!` and heap configuration for `alloc`-based types (`Vec`, `String`)
  - [Syscalls](https://docs.nervos.org/docs/script/rust/rust-api-syscalls)
    - Understood the role of VM syscalls for reading transaction and blockchain context inside scripts
    - Learned common high-level APIs such as `load_tx_hash`, `load_script_hash`, and `QueryIter`
    - Reviewed loaders for cells/inputs/witnesses and how they support script verification logic
  - [Error](https://docs.nervos.org/docs/script/rust/rust-api-error)
    - Learned how `ckb-std` maps syscall failures into `Result<T, SysError>`
    - Understood core error variants such as `IndexOutOfBound`, `ItemMissing`, `LengthNotEnough`, and `Unknown`
    - Reviewed spawn-related error cases (`InvalidFd`, `WaitFailure`, `MaxVmsSpawned`, etc.)
  - [Logger](https://docs.nervos.org/docs/script/rust/rust-api-logger)
    - Learned usage of `debug!` for contract-side logging during development
    - Understood that logging still consumes cycles and release behavior depends on debug assertions
    - Reviewed `ckb_std::logger::init()` for structured log levels (`trace`, `debug`, `info`, `warn`, `error`)
  - [Type ID](https://docs.nervos.org/docs/script/rust/rust-api-type-id)
    - Learned to use `type_id::check_type_id(offset)` for singleton type-script validation
    - Understood minting / transfer / burning validation paths for Type ID cells
    - Reviewed how uniqueness is derived from first input + cell index in minting flow
  - [Spawn](https://docs.nervos.org/docs/script/rust/rust-api-spawn)
    - Learned `spawn_cell` for direct cross-script process invocation
    - Understood inherited file descriptors and process-level APIs (`process_id`, `pipe`, `read`, `write`)
    - Reviewed parent/child communication flow using spawn-related syscalls
  - [IPC](https://docs.nervos.org/docs/script/rust/rust-api-ipc)
    - Learned `ckb-script-ipc` service trait pattern for simpler inter-process communication
    - Understood shared trait definition, server setup (`spawn_server`), and request loop (`run_server`)
    - Reviewed client setup pattern for invoking server methods over IPC pipes

---

## Key Learnings

- Solid understanding of the `ckb-std` API layers and how they map to real script development
- Better understanding of script bootstrapping in CKB: explicit entrypoint + manual allocator setup
- Learned to design script logic around syscall-backed data access from inputs, outputs, and witnesses
- Improved confidence using `high_level` helpers (`QueryIter`, `load_cell_*`) to write cleaner validation code
- Connected API theory to implementation choices in the Loavix invoice contract

---

## Practical Progress
<img width="1384" height="778" alt="image" src="https://www.dropbox.com/scl/fi/afkvww9drgcdunamdtb2x/Screenshot_2026-04-26_07-39-28.png?rlkey=w7xrskvwmq6pla1isogygjbrz&st=31h1ckx4&dl=0" />
### Loavix Invoice Rust Script

This week I started writing the Rust script for **Loavix (Invoice system)** and completed an initial successful build.

#### Current implementation progress

- Added the `invoice-script` contract crate under `loavix-scripts/contracts/invoice-script`
- Structured script modules into `lib.rs`, `types.rs`, `payment.rs`, `cancellation.rs`, and `error.rs`
- Implemented invoice parsing with a fixed 80-byte layout (`invoice_hash`, `amount_shannons`, `payee_lock_hash`, `deadline_block`)
- Implemented unlock-type dispatch from script args:
  - `0x00` → payment path
  - `0x01` → cancellation path
- Implemented payment verification using `QueryIter` + `load_cell_capacity` to ensure payee outputs meet minimum invoice amount
- Implemented cancellation verification requiring a payee-controlled input cell as co-signer proof
- Built the contract successfully for the CKB target

---

## Environment

- Node.js (v22.13.0)
- pnpm (10.29.2)
- Rust (1.94.0) + RISC-V target (`riscv64imac-unknown-none-elf`)
- Docker (29.2.0)
- offckb (0.4.4)
- clang (18.1.8)
- ckb-std (1.1)
- JavaScript SDK: Lumos (0.23.0), CCC (≥1.12.5)
