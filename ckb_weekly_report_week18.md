## Builder Track Weekly Report - Week 18

**Name:** Divine Oshione Anesi

**Week Ending:** 07-21-2026


### Courses Completed

- Advanced the Loavix Merchant Fiber Connector toward a reusable standalone Docker agent:
  * Completed the reusable runtime model where one container build accepts merchant-specific configuration at startup
  * Kept Fiber node access private while the connector communicates with a remote backend over outbound HTTPS
  * Brought the connector close to project-independent extraction without requiring merchant-specific code changes
- Implemented the outbound connector architecture for privately hosted payment nodes:
  * Kept every backend connection connector-initiated so the backend never needs to enter the merchant network
  * Compared same-host, shared Docker network, private LAN, and VPN deployment arrangements
  * Clarified the separate networking requirements for private Fiber RPC access and public Fiber peer-to-peer routing
- Continued Rust self-study with a focus on async internals:
  * Reviewed the `Future::poll` model and how `Context` and `Waker` allow an executor to resume pending work
  * Studied why self-referential async state can require `Pin` and how `Unpin` changes movement guarantees
  * Connected async state-machine behavior to long-running connector polling, timeouts, cancellation, and retry safety
- Finalized the core connector protocol and capability design:
  * Completed heartbeat, command polling, command result, authentication, timeout, and node identity responsibilities
  * Considered versioned endpoint contracts and backward-compatible environment variable aliases
  * Kept Fiber RPC capability allowlisting as a connector-owned security boundary
- Identified the final production hardening needed before standalone release:
  * Identified durable command storage as necessary for backend restarts and multiple replicas
  * Considered reusable backend adapters for NestJS, Express, and other application frameworks
  * Reviewed token scope, idempotency, command ownership, replay protection, and observable connector status

### Key Learnings
- A generic Docker image is not automatically a generic integration protocol:
    - the connector implementation already separates runtime configuration from application code
    - the connector still expects Loavix endpoint paths, request headers, and payload shapes
    - after packaging, another application can use the same container build only if it implements that contract
    - a standalone connector product needs a documented and versioned protocol rather than only renamed environment variables

- The outbound polling direction is the main operational advantage:
    - the connector can run behind NAT or a residential router
    - the merchant does not need to expose a connector port
    - the backend does not need direct access to the merchant's Fiber RPC
    - only outbound HTTPS access to the backend is required for connector coordination

- Fiber RPC privacy and Fiber network reachability are different concerns:
    - the RPC endpoint should stay on loopback, a Docker network, LAN, or VPN
    - the connector needs private access to that RPC endpoint
    - the Fiber peer-to-peer port may still need public reachability for public routing
    - opening the RPC publicly would create unnecessary node and fund-management risk

- A connector should be an allowlisted agent, not an unrestricted RPC proxy:
    - Loavix currently allows only `new_invoice` and `get_invoice`
    - the backend should not be able to request arbitrary Fiber methods through configuration
    - new capabilities should be explicitly added, reviewed, versioned, and tested
    - the merchant should be able to understand what authority the connector grants to a backend

- Durable command state is required for horizontal backend deployment:
    - in-memory command queues can disappear during a backend restart
    - multiple replicas can route a polling connector away from the process holding its command
    - Redis or PostgreSQL should own pending command, lease, timeout, and result state
    - idempotent command execution is necessary when delivery or result submission is retried

- Rust async internals make connector lifecycle behavior easier to reason about:
    - a pending future represents suspended state rather than a blocked operating-system thread
    - a waker tells the executor when polling can make progress again
    - pinning prevents movement when an async state machine relies on stable internal references
    - cancellation and timeout paths still need explicit persistent-state rules outside the future itself

- Merchant-owned Fiber and hosted Fiber remain separate Loavix settlement models:
    - the merchant connector creates invoices through the merchant's own Fiber node
    - those funds remain in the merchant's channels and do not enter Loavix hosted Fiber accounting
    - the hosted Fiber path receives through the Loavix-operated node and can queue an automatic on-chain payout
    - making the connector reusable should preserve this custody distinction for every integrating application

### Practical Progress
- Completed most of the reusable connector runtime foundation:
    - Added Docker packaging for the connector runtime
    - Made the container configurable through backend URL, connector token, merchant address, and Fiber RPC URL
    - Added optional Fiber RPC bearer-token support
    - Kept every merchant-specific value outside the container build

- Implemented the connector backend contract:
    - `POST /fiber-connector/heartbeat` reports node identity and ready-channel state
    - `POST /fiber-connector/poll` retrieves the next merchant command
    - `POST /fiber-connector/commands/:id/result` returns a command result or error
    - Connector requests authenticate through the `x-loavix-connector-token` header
    - Wallet authentication remains required when creating or rotating a connector registration

- Implemented connector identity and token security:
    - Connector tokens use 32 random bytes with the `lfc_` prefix
    - Loavix stores a SHA-256 token hash instead of the raw token
    - Regenerating a token invalidates the old connector immediately
    - The first successful heartbeat binds the registration to the merchant Fiber node public key
    - A later node-key mismatch makes the connector unavailable

- Implemented connector health and command behavior:
    - The connector reads `node_info` and `list_channels` from the merchant Fiber node
    - Heartbeats run every 10 seconds by default
    - Loavix considers the connector inactive after a 30-second heartbeat timeout
    - Command polling runs every 1.5 seconds by default
    - Connector commands time out after 30 seconds by default
    - The UI distinguishes unregistered, offline, connected, ready, and wallet-locked states

- Validated separated deployment topologies:
    - Confirmed that the backend can run on a different public server from the merchant node
    - Confirmed that the Fiber node and connector can run together on the merchant machine
    - Confirmed that Linux host networking allows a connector container to reach a host Fiber RPC on `127.0.0.1:8227`
    - Confirmed that two containers can instead communicate through a private Docker network and service name
    - Confirmed that a remote private Fiber RPC can be reached through LAN or VPN addressing without exposing it publicly

- Brought the standalone connector extraction close to completion:
    - Completed the container runtime, configuration, outbound polling, authentication, node binding, health reporting, and command execution foundation
    - Kept the RPC method allowlist inside the connector binary
    - Reduced the remaining extraction work to generic environment aliases, a versioned contract, and reusable backend adapters
    - Identified Redis or PostgreSQL command storage as the main remaining production implementation task
    - Preserved compatibility with existing Loavix connector registrations during the planned migration

- Clarified the relationship to Fiber Offers and future projects:
    - The same outbound connector model can support another application if that backend implements the connector contract
    - The connector does not remove the need for application-specific invoice, offer, settlement, or accounting logic
    - Fiber Offers could use the agent for merchant node access while retaining its own reusable-offer resolver model
    - Loavix can retain its invoice-specific accounting while sharing the lower-level node connector

- Verification completed:
    - Validated the connector runner, backend controller, backend service, database entity, Dockerfile, account setup flow, and merchant documentation together
    - Confirmed the exact supported Fiber RPC command allowlist
    - Confirmed the connector uses outbound polling rather than inbound backend connections
    - Confirmed the same container build can connect a remote backend to a merchant Fiber node running on another machine
    - Separated the completed connector runtime from the remaining durability and generic adapter work

### Environment
- CKB network: testnet
- Hosted Loavix API: `https://loavix.duckdns.org`
- Merchant connector packaging: Docker image implementation in progress
- Default merchant Fiber RPC: `http://127.0.0.1:8227`
- Connector transport: outbound HTTPS polling
- Default heartbeat interval: 10 seconds
- Default heartbeat timeout: 30 seconds
- Default command polling interval: 1.5 seconds
- Default command timeout: 30 seconds
- Allowed Fiber RPC commands: `new_invoice` and `get_invoice`
- Backend framework: NestJS
- Connector runtime: Node.js in Docker
- Current connector command storage: backend process memory
- Proposed durable command storage: Redis or PostgreSQL
