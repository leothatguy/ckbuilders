# Builder Track Weekly Report - Week 5

**Name:** Divine Oshione Anesi

**Week Ending:** 04-18-2026

---

## Courses Completed

- Completed the "Rust Script Development" modules from the Nervos CKB docs:
  - [Quick Start](https://docs.nervos.org/docs/script/rust/rust-quick-start): Creating and running CKB Rust smart contracts
    - Scaffolding a project using `ckb-script-templates` via `cargo generate`
    - Generating contracts with `make generate`, building with `make build`
    - Running contracts with `ckb-debugger` and `ckb-testtool`
    - Working through the Hello World and Simple Print Args examples
    - Learned to use `dump_tx` for exporting test transactions as JSON
  - [Build](https://docs.nervos.org/docs/script/rust/rust-build): Deep dive into building Rust scripts for CKB
    - Understanding the RISC-V compilation target (`riscv64imac-unknown-none-elf`) and what each flag means
    - Why `--release` is mandatory (debug builds can exceed the 4MB ckb-vm memory limit)
    - History of CKB Rust tooling: Capsule → `ckb-script-templates` + `ckb-testtool`
    - Reproducible builds using Docker containers
  - [Debug](https://docs.nervos.org/docs/script/rust/rust-debug): Debugging CKB scripts with CKB-Debugger
    - Installing and using `ckb-debugger` for off-chain script execution
    - Executing on-chain transactions locally via `ckb-cli mock-tx dump`
    - Debugging failed transactions built with Lumos SDK
    - GDB remote debugging with `--mode gdb` for step-through debugging
    - VSCode integration using NativeDebug and CodeLLDB extensions
    - Flamegraph profiling with `--pprof` and `inferno`
    - Native Simulator for debugging on the host platform without RISC-V

---

## Key Learnings

- Understood the CKB cell model and how invoices can be represented as on-chain cells with type scripts
- Learned the full CKB Rust script development workflow: scaffold → generate → build → test → debug
- Understood the RISC-V cross-compilation pipeline and why clang/LLVM is needed as the C compiler for `ckb-std` dependencies
- Learned the Lumos SDK for building and signing CKB transactions programmatically
- Understood the offckb devnet tooling - pre-funded accounts, system script outpoints, and the RPC proxy on port 28114
- Learned how to use `ckb-debugger` for local transaction execution, GDB debugging, and performance profiling
- Understood the `ckb-testtool` testing framework and how it simulates the on-chain environment for contract testing

---

## Practical Progress

### Rust scripting environtment setup and practice

<img width="1785" height="1005" alt="image" src="https://github.com/user-attachments/assets/9ea6147f-5c43-484c-b3f6-3be22e394746" />
<img width="1384" height="778" alt="image" src="https://github.com/user-attachments/assets/28659bc0-cbc6-4b82-8af9-70c0a39a7699" />

### Environment Setup (Loavix)

Set up the complete local development environment for the Loavix system a pnpm monorepo with NestJS backend, Next.js frontend, shared type package, CLI scripts, and a Rust smart contract scaffold.

#### What was built

- **Monorepo structure**
- **Shared types package **
- **NestJS backend **
- **Next.js frontend **
- **Rust smart contract **
- **Docker Compose**: PostgreSQL 16 + Redis 7 with healthchecks and named volumes
- **Test transaction script (`send-test-tx`)**: Uses Lumos to send 100 CKB between offckb pre-funded accounts on the local devnet, waits for confirmation

#### test script

`pnpm run send-test-tx`: sends a real CKB transaction on the local devnet and prints the confirmed tx hash.

---

## Environment

- Node.js (v22.13.0)
- pnpm (10.29.2)
- Rust (1.94.0) + RISC-V target (`riscv64imac-unknown-none-elf`)
- Docker (29.2.0)
- offckb (0.4.4)
- clang (18.1.8)
- JavaScript SDK: Lumos (0.23.0), CCC (≥1.12.5)
- NestJS (11.x), Next.js (16.x), TypeORM, BullMQ
