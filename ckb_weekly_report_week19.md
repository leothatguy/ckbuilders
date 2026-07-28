## Builder Track Weekly Report - Week 19

**Name:** Divine Oshione Anesi

**Week Ending:** 07-28-2026


### Courses Completed

- Completed the standalone extraction of the Fiber node connector:
  * Moved the connector into an independent project with no application-specific runtime profiles or environment aliases
  * Replaced the previous private contract with a generic, versioned Protocol v1 that any backend can implement
  * Added a framework-neutral reference gateway and backend example for testing integrations independently
- Studied CKB-VM performance and cycle-aware script design:
  * Looked at how instruction count, syscalls, data loading, and repeated serialization work contribute to transaction cycle usage
  * Compared reducing algorithmic work with smaller optimizations such as avoiding repeated cell reads and unnecessary allocations
  * Considered how cycle budgets should be measured with realistic transactions instead of estimated only from source-code complexity
- Continued advanced Rust self-study with a focus on macros and compile-time code generation:
  * Studied the difference between declarative `macro_rules!` macros and procedural macros
  * Reviewed how token streams, pattern matching, repetition, and macro hygiene affect generated Rust code
  * Explored where derives and attribute macros reduce repetitive implementation while still keeping generated behavior reviewable
- Studied public container release integrity:
  * Reviewed multi-architecture image publishing for Linux AMD64 and ARM64
  * Added software bill of materials and provenance generation to the release process
  * Considered dependency update automation, immutable release tags, vulnerability scanning, and image signing as separate supply-chain controls
- Defined the first adoption milestone for the generic connector:
  * Selected Loavix as the first application that will implement Protocol v1 against the standalone connector
  * Defined a reusable conformance suite as the acceptance boundary for future backend integrations
  * Added migration tasks for protocol authentication, node identity pinning, credential rollover, and existing merchant registrations

### Key Learnings
- A standalone connector needs an executable reference implementation on both sides of the protocol:
    - documentation describes the expected request and response shapes
    - a reference gateway demonstrates registration, authentication, command leasing, timeout, and result handling
    - end-to-end tests prove that a backend command reaches Fiber RPC and returns through the same contract
    - future applications can validate compatibility without copying application-specific connector code

- At-least-once command delivery creates an important crash-consistency boundary:
    - a Fiber RPC call may succeed immediately before the connector loses the result response
    - redelivering the same command ID prevents command loss but does not alone prevent repeated side effects after a connector restart
    - process-local result caching is useful during normal retries but durable result journaling is needed across restarts
    - invoice creation therefore needs stronger deduplication than read-only node and invoice queries

- Generic capability control should be enforced by the agent that owns node access:
    - the standalone connector currently exposes only `node_info`, `list_channels`, `new_invoice`, and `get_invoice`
    - an integrating backend can reduce this set for a connector but cannot enable an arbitrary Fiber RPC method
    - separating operational status methods from invoice methods makes granted authority easier for a node owner to understand
    - adding fund movement or channel-management methods later should require a deliberate protocol and security review

- Public image delivery is part of the product's trust model:
    - a working Dockerfile does not prove which source revision produced a public image
    - tagged CI releases, provenance, and an SBOM make the artifact easier to trace and inspect
    - running the container as a non-root user reduces the impact of a runtime compromise
    - signed images, pinned build inputs, and automated vulnerability checks remain necessary release-hardening steps

- CKB script performance should be treated as a transaction-level property:
    - cycle cost depends on the complete script group and the transaction data being loaded
    - repeated syscalls and decoding can dominate code that appears simple at the source level
    - benchmarks should include realistic input groups, witnesses, and cell data sizes
    - optimization should preserve deterministic validation and clear failure behavior before reducing cycles

- Rust macros are most useful when they encode a stable pattern rather than hide control flow:
    - declarative macros can remove repeated syntax while retaining compile-time expansion
    - procedural macros can generate trait implementations and validation scaffolding from structured input
    - macro errors can become difficult to understand when generated code is too broad or implicit
    - testing expanded behavior and keeping macro inputs narrow makes compile-time automation safer to maintain

- Using Loavix as the first Protocol v1 adopter will test whether the extraction is genuinely independent:
    - Loavix should implement the public connector endpoints instead of adding a Loavix mode back into the standalone image
    - existing merchant registrations need a controlled migration path rather than an immediate incompatible switch
    - the same conformance tests should remain usable by Loavix, Fiber Offers, and unrelated future applications
    - integration success should be measured through protocol behavior rather than shared source code

### Practical Progress
- Created the independent Fiber Node Connector project:
    - Added a zero-runtime-dependency Node.js agent with configuration validation and structured logging
    - Removed all Loavix-specific names, endpoint profiles, headers, and environment aliases
    - Kept merchant and application configuration outside the image through `BACKEND_URL`, `CONNECTOR_TOKEN`, `CONNECTOR_ID`, and `FIBER_RPC_URL`
    - Added optional Fiber RPC bearer authentication and configurable timeouts, retry limits, logging, and capability reduction

- Implemented the generic Protocol v1 client:
    - Added `POST /v1/connectors/heartbeat` for node identity, capability, channel, and availability reporting
    - Added `POST /v1/connectors/poll` for outbound command retrieval
    - Added `POST /v1/connectors/commands/:id/result` for structured command completion
    - Added bearer authentication, connector version headers, connector identity validation, and configurable API prefixes

- Implemented a reference backend gateway:
    - Added connector registration with SHA-256 token indexing
    - Added heartbeat freshness tracking and active connector status
    - Added command queues, delivery leases, redelivery, attempt limits, timeouts, and capability validation
    - Added a small HTTP adapter and reference backend for exercising the protocol without an existing application

- Hardened connector command execution:
    - Added command shape validation before any Fiber RPC request
    - Added a hardcoded maximum method allowlist owned by the connector runtime
    - Added result caching for repeated command IDs during the connector process lifetime
    - Added exponential retry backoff for backend failures and graceful `SIGINT` and `SIGTERM` shutdown
    - Added node and ready-channel status normalization across expected Fiber RPC response shapes

- Added standalone container and release automation:
    - Built the runtime from `node:22-alpine` and ran it as the non-root `node` user
    - Added branch and pull-request CI across Node.js 20 and 22
    - Added tagged GHCR publication for Linux AMD64 and ARM64 images
    - Added semantic image tags, BuildKit caching, provenance, an SBOM, and digest reporting
    - Added weekly Dependabot checks for npm and GitHub Actions inputs

- Added integration and operator documentation:
    - Documented Protocol v1 authentication, heartbeats, polling, command results, and backend responsibilities
    - Documented backend integration through direct protocol implementation or the JavaScript reference gateway
    - Documented same-host, Docker-network, LAN, and VPN arrangements without exposing Fiber RPC publicly
    - Documented token handling, capability boundaries, command leases, payment verification, deployment, and tagged releases

- Independent verification results:
    - Passed 12 automated tests covering configuration, command policy, authentication, connector identity, capabilities, leasing, redelivery, and protocol formatting
    - Passed a complete backend-to-connector-to-Fiber-RPC command round trip with independent mock servers
    - Confirmed the production Docker image builds and starts as a non-root user
    - Confirmed the source and documentation contain no Loavix-specific compatibility profile
    - Measured the current local image at approximately 228 MB unpacked

- Added the next cross-project implementation milestone:
    - Implement Protocol v1 endpoints in the Loavix backend while preserving the current merchant flow during migration
    - Add expected Fiber node public-key pinning and explicit connector token rotation
    - Build a reusable conformance suite for authentication, heartbeat expiry, leasing, redelivery, idempotency, capability rejection, and result handling
    - Run the standalone public image against Loavix as the first production integration before removing the legacy connector path

### Environment
- Standalone project: `fiber-node-connector`
- Connector protocol: Protocol v1
- Connector runtime: Node.js 20 or newer
- Container base: `node:22-alpine`
- Fiber RPC transport: private JSON-RPC
- Supported Fiber RPC methods: `node_info`, `list_channels`, `new_invoice`, and `get_invoice`
- Reference gateway storage: process memory for development only
- Release registry: GitHub Container Registry
- Release architectures: Linux AMD64 and ARM64
- Release status: workflow configured; first public image not yet published
- Verification: Node.js test runner and Docker
