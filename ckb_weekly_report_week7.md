# Builder Track Weekly Report - Week 7

**Name:** Divine Oshione Anesi

**Week Ending:** 05-02-2026

---

## Courses & Topics Completed

- Explored Payment Channels and Off-chain Scaling on CKB:
  - [Fiber Network Documentation](https://www.fiber.world/docs)
    - Understood the architecture of Fiber Network as a Lightning-compatible P2P payment and swap network on CKB
    - Learned about its highlighted features: multi-asset support, instant swaps, low-cost micropayments, high privacy, and watchtower support
    - Explored the Fiber routing and lifecycle, including the invoice protocol and cross-network asset transfers
  - [Fiber Network Showcase](https://www.fiber.world/showcase)
    - Explored open-source projects built on Fiber, including Fiber Audio Player, Echo, FiberQuest, and BlackBox POS
    - Reviewed the practical applications of Fiber for micropayments, AI agent economies, and decentralized gaming
  - [Perun CKB Contract](https://github.com/perun-network/perun-ckb-contract)
    - Learned about Ethereum-compatible P2P payment channels/swaps on CKB using the Perun network
    - Understood the design of the `perun-channel-lockscript`, `perun-channel-typescript` (state machine), and `perun-funds-lockscript`
    - Reviewed the cross-chain verifiability and interoperability with Ethereum-based Perun backends
- Studied Ecosystem Scripts and Tools:
  - [Ecosystem Scripts Introduction](https://docs.nervos.org/docs/ecosystem-scripts/introduction)
    - Reviewed common ecosystem scripts such as `secp256k1_blake160_sighash_all`, proxy locks, Omnilock, and xUDT
    - Understood the difference between standalone executable scripts and library scripts like CKB Auth and CKB Crypto Service
  - [Rust SDK and Devtools](https://docs.nervos.org/docs/sdk-and-devtool/rust)
    - Set up the `ckb-sdk-rust` client for interacting with CKB nodes via RPC
    - Learned how to assemble and sign CKB transactions using the Rust SDK, including capacity transfers and generating new addresses
    - Explored the `ckb_sdk::rpc::CkbRpcClient` and `SecpSighashUnlocker`

---

## Key Learnings

- Gained a solid understanding of how CKB supports high-throughput payment channels via Fiber and Perun while maintaining base-layer security.
- Learned that Fiber enables seamless cross-network asset swaps and multi-asset micropayments leveraging CKB's unique cell model.
- Discovered that Perun maintains an Ethereum-compatible state representation on CKB, allowing multi-chain channels and unified data models.
- Understood how to leverage the Rust SDK (`ckb-sdk-rust`) to programmatically build, sign, and broadcast transactions using native Rust code.
- Mapped out the various core ecosystem scripts (Omnilock, xUDT, Spore) available for composing advanced CKB dApps.

---

## Practical Progress

### Exploring Payment Channels & Rust SDK

This week I focused on studying off-chain scaling solutions and setting up the Rust SDK for programmatic transaction generation.

#### Current implementation progress

- Set up and explored the CKB Rust SDK for interacting with the local Devnet and Testnet.
- Reviewed and analyzed the Fiber Network open-source showcase projects to conceptualize new potential use cases for micropayments.
- Investigated the `perun-ckb-contract` repository and toolchain to understand how stateful type scripts are used to enforce channel state transitions on-chain.
- Studied the CKB Ecosystem Scripts to evaluate pre-built tools (Omnilock, CKB Auth) for use in upcoming projects.

---

## Environment

- Node.js (v22.13.0)
- pnpm (10.29.2)
- Rust (1.94.0) + RISC-V target (`riscv64imac-unknown-none-elf`)
- ckb-sdk (3.2.0)
- Docker (29.2.0)
- offckb (0.4.4)
- clang (18.1.8)
- JavaScript SDK: Lumos (0.23.0), CCC (≥1.12.5)
