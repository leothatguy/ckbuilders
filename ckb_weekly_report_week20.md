## Builder Track Weekly Report - Week 20

**Name:** Divine Oshione Anesi

**Week Ending:** 08-04-2026


### Courses Completed

- Studied CKB transaction time-lock semantics through the `since` field:
  * Reviewed the difference between absolute and relative transaction constraints
  * Compared block-number, epoch-based, and timestamp-based lock measurements
  * Examined how input maturity and transaction validity depend on the referenced cell's creation context
- Continued advanced Rust self-study with a focus on atomics and memory ordering:
  * Studied `Relaxed`, `Acquire`, `Release`, `AcqRel`, and `SeqCst` ordering guarantees
  * Reviewed compare-and-exchange loops and the possibility of another thread changing shared state between a load and an update
  * Compared atomic state transitions with mutex-protected compound operations
- Designed a one-time enrollment flow for standalone Fiber connectors:
  * Replaced manual long-lived token copying in the proposed onboarding path with a short-lived, single-use enrollment code
  * Defined how the connector exchanges that code for its permanent connector identity and credential over HTTPS
  * Planned local credential persistence so a restarted container can reconnect without repeating merchant enrollment
- Studied compatibility negotiation for independently released agents and backends:
  * Distinguished connector software version from connector protocol version
  * Considered minimum and maximum supported protocol ranges instead of relying only on endpoint names
  * Defined fail-closed behavior for incompatible versions before any Fiber command is accepted
- Converted the enrollment design into a cross-project implementation milestone:
  * Added backend, connector runtime, merchant interface, credential storage, and migration tasks
  * Defined security requirements for expiration, single use, node binding, revocation, and redacted diagnostics
  * Kept the enrollment extension compatible with applications that still provision connector credentials directly

### Key Learnings
- CKB time locks express when an input may be consumed rather than scheduling automatic execution:
    - an eligible transaction still needs to be constructed and submitted by a user or service
    - absolute constraints refer to a chain position or time measurement directly
    - relative constraints measure from the referenced input cell's confirmed origin
    - choosing epochs can better match CKB consensus time progression than assuming a fixed block duration

- Time-lock interpretation belongs in both transaction construction and user-facing state:
    - a valid script alone does not make an immature input spendable
    - builders should calculate the earliest valid submission point before requesting a wallet signature
    - interfaces should distinguish a transaction that is locked by consensus from one rejected by script validation
    - tests need boundary cases immediately before, at, and after the maturity condition

- Atomic operations prevent data races but do not automatically make a multi-step workflow correct:
    - `Relaxed` ordering can protect an individual counter without establishing ordering for surrounding data
    - release and acquire operations can publish and observe related state across threads
    - compare-and-exchange is useful when a state update must succeed only from an expected previous value
    - durable payment state still requires database transactions even when an in-process worker uses atomics correctly

- Enrollment credentials and runtime credentials should have different risk profiles:
    - an enrollment code should expire quickly and become invalid after one successful exchange
    - the permanent connector token should never be returned to the browser after enrollment
    - a stolen unused enrollment code has a limited window, while a stolen runtime token requires explicit revocation
    - separating the two credentials makes merchant onboarding simpler without weakening long-running authentication

- First-contact node binding needs an explicit ownership decision:
    - the backend should record the Fiber node public key observed during enrollment
    - later heartbeats from a different node key should not silently replace the trusted identity
    - changing nodes should require a merchant-authorized reset or a new connector registration
    - a one-time code must not be able to claim an already bound connector after it has expired or been consumed

- Protocol compatibility should be negotiated before command delivery:
    - software releases can change without changing the wire contract
    - protocol changes can become incompatible even when the connector process still starts normally
    - heartbeat responses can communicate accepted protocol ranges and required upgrade information
    - rejecting an unsupported version is safer than allowing a partially understood payment command

- Better onboarding can preserve the outbound-only trust boundary:
    - the connector still initiates enrollment and all later communication
    - the application does not need an inbound route to the merchant machine
    - Fiber RPC remains private while setup is reduced to a backend URL, enrollment code, and node RPC URL
    - direct token provisioning remains useful for automation and advanced operators

### Practical Progress
- Defined the connector enrollment sequence:
    - The signed-in merchant requests a new enrollment code from the application
    - The backend stores only a hash of the code together with merchant ownership, expiration, and requested capabilities
    - The connector submits the code, connector version, protocol version, and observed Fiber node identity over HTTPS
    - The backend consumes the code atomically and returns a connector ID plus a newly generated runtime token
    - The connector stores the issued credential in its mounted data directory and begins normal Protocol v1 heartbeats

- Defined enrollment replay and concurrency protection:
    - Added a single-use state transition so two connector processes cannot redeem the same code successfully
    - Required database-level uniqueness and atomic consumption rather than process-local checks
    - Added explicit expired, consumed, revoked, and ownership-mismatch outcomes
    - Required rate limits and audit events for code creation and redemption attempts

- Planned secure local credential handling for the standalone image:
    - Use a mounted connector data directory instead of baking credentials into the image
    - Restrict credential-file permissions and avoid printing token values in logs or diagnostics
    - Load the stored credential automatically after container restarts
    - Support explicit credential deletion when a connector is revoked or moved to another node
    - Preserve direct `CONNECTOR_TOKEN` configuration for non-interactive deployment systems

- Planned Loavix merchant enrollment UX:
    - Replace the primary manual token-copy flow with a short-lived setup code and copyable Docker command
    - Show code expiration and whether redemption has completed
    - Move the account state from waiting for enrollment to verifying node identity and then connected
    - Require wallet confirmation before resetting a bound node or issuing a replacement enrollment code
    - Keep advanced manual configuration available without placing development commands in the merchant interface

- Defined version-negotiation behavior:
    - Include software and protocol versions during enrollment and heartbeat requests
    - Return the backend's accepted protocol range and upgrade requirement
    - Stop polling when the backend reports an incompatible protocol instead of repeatedly submitting failing requests
    - Surface an actionable incompatible-version state to the merchant account interface

- Added enrollment verification requirements:
    - Test expiration at the exact validity boundary
    - Test simultaneous redemption attempts against one code
    - Test merchant ownership mismatch and attempted reuse after successful redemption
    - Test node public-key mismatch after the connector has been bound
    - Test restart recovery using a persisted credential volume
    - Test migration between direct token provisioning and enrollment-based provisioning

- Updated the implementation direction:
    - Added one-time connector enrollment as a planned Loavix and standalone connector milestone
    - Kept Protocol v1 command behavior separate from the bootstrap exchange
    - Scheduled the enrollment flow after the generic backend adapter and conformance baseline
    - Marked public-image onboarding as incomplete until credential persistence and restart recovery are verified

### Environment
- Design network: CKB testnet
- Current connector contract: Protocol v1
- Planned bootstrap method: short-lived single-use enrollment code
- Runtime authentication: connector ID and bearer token
- Node identity: Fiber node public key
- Credential persistence target: mounted Docker volume
- Backend persistence target: PostgreSQL transaction-backed enrollment records
- Merchant authorization: CKB wallet-authenticated account session
- Connector transport requirement: HTTPS
- Rust study focus: atomic state transitions and memory ordering
- CKB study focus: absolute and relative `since` constraints
- Implementation status: design and acceptance criteria completed; runtime work pending
