## Builder Track Weekly Report - Week 21

**Name:** Divine Oshione Anesi

**Week Ending:** 08-10-2026


### Milestones Completed

- Audited privileged backend boundaries in a deployed payment application:
  * Traced internal payout routes from controller guards through the public reverse proxy
  * Compared application-level authentication with edge-level route denial as independent controls
  * Applied fail-closed secret handling so missing production configuration disables privileged operations
- Studied CKB header dependencies and consensus-data access:
  * Distinguished `header_deps` from cell dependencies and normal transaction inputs
  * Reviewed how scripts can load explicitly referenced block headers without reading arbitrary chain state
  * Considered when historical block metadata belongs in deterministic transaction validation
- Continued advanced Rust self-study with a focus on `unsafe` boundaries:
  * Reviewed raw pointers, `NonNull`, `MaybeUninit`, and calls whose correctness depends on caller-maintained invariants
  * Studied why an `unsafe` block permits specific operations but does not suspend Rust's ownership model everywhere around it
  * Compared exposing a small safe abstraction with spreading unchecked assumptions across application code
- Completed a responsive product-design pass across the main Loavix routes:
  * Reworked information priority for desktop, tablet, narrow mobile, and 320-pixel screens
  * Used rendered screenshots to find defects that were not visible through source review alone
  * Added motion-reduction, focus, mobile viewport, form-control, and text-overflow safeguards
- Published the standalone Fiber Node Connector source repository:
  * Performed repository-boundary, dependency, secret, and configuration scans before the first push
  * Split the initial history into seven incremental commits covering scaffold, runtime, gateway, example, tests, documentation, and CI
  * Confirmed the public repository and its CI without claiming an unreleased container tag

### Key Learnings
- A route prefix such as `internal` is a naming convention, not an authorization boundary:
    - a catch-all reverse-proxy rule can make every backend controller reachable from the public domain
    - controller-level authentication remains necessary even when infrastructure is expected to hide a route
    - the reverse proxy should still deny operator routes to reduce exposure before requests reach the application
    - privileged route discovery should also be limited by disabling production API documentation

- Security-sensitive configuration should fail closed:
    - treating an unset internal token as unrestricted access repeats the original trust failure
    - refusing the entire internal API makes missing deployment configuration visible immediately
    - secret comparison should check byte length before using a timing-safe equality operation
    - smoke and maintenance tools must authenticate through the same path as other internal callers

- Payment administration endpoints require stronger protection than normal read APIs:
    - changing payout state can alter whether a merchant is considered paid or still owed funds
    - an incorrect terminal transition may prevent an automatic worker from revisiting the payout
    - operator routes need authentication, auditability, idempotency, and restricted network reachability
    - production rollout must include the new secret before the protected workflow can operate

- CKB header dependencies make historical consensus data explicit:
    - a transaction commits to the exact headers its scripts are allowed to inspect
    - this keeps script execution deterministic instead of allowing open-ended blockchain queries
    - header data and cell data solve different validation problems and should not be treated as interchangeable dependencies
    - transaction builders must include every required header or the script cannot load it during verification

- Rust `unsafe` shifts proof obligations rather than removing safety requirements:
    - the compiler cannot verify the invariant behind a raw-pointer dereference or uninitialized memory operation
    - the developer must document validity, alignment, initialization, aliasing, and lifetime assumptions
    - a narrow safe wrapper lets most callers continue relying on normal Rust guarantees
    - tests can exercise behavior but cannot prove that every unsafe memory invariant is valid

### Practical Progress
- Redesigned the Loavix account page as a four-stage money pipeline:
    - Presented Fiber node, hosted balance, automatic payouts, and merchant wallet on one continuous flow rail
    - Derived ready, warning, idle, and locked stage states from live connector and payout data
    - Animated each connector only when value can move through that part of the pipeline
    - Reconciled hosted net balance into available, queued, and paid-out segments using integer shannon values
    - Integrated Fiber connector onboarding into the first stage and added one-click Docker command copying

- Added a shared responsive design-token layer:
    - Introduced reusable surface, ink, accent, radius, spacing, measure, and typography variables
    - Added fluid gutters and figures while preserving fixed readable content limits
    - Added visible focus treatment, reduced-motion behavior, mobile text-size protection, and form-control sizing
    - Added a mobile viewport theme color that matches the application surface

- Reworked the shared header for continuous width reduction:
    - Kept the header on one row instead of stacking navigation and wallet controls
    - Collapsed Fiber status to an accessible dot at tablet width
    - Progressively removed back-label, brand wordmark, network badge, and balance text only when space required it
    - Kept wallet identity as the flexible, truncating element
    - Fixed a selector collision that caused the Fiber status label to collapse into the status dot

- Improved dashboard and payer-page responsiveness:
    - Stacked dashboard panes before the invoice table became narrower than its row constraints
    - Added mobile invoice field labels and kept amount values grouped with the CKB unit
    - Allowed long invoice outpoints to wrap cleanly at 320 pixels
    - Removed 208 lines of CSS left behind by the earlier shared-header migration
    - Converted payer details to an available-width grid and added compact full-width mobile actions

- Secured operator-only backend routes locally:
    - Added a NestJS guard requiring `x-loavix-internal-token` for the internal invoice controller
    - Used length-checked `timingSafeEqual` token comparison
    - Returned service unavailable when `INTERNAL_API_TOKEN` is not configured
    - Disabled Swagger route publication in production
    - Denied internal and documentation paths at the HTTPS Nginx edge
    - Updated payout and settlement smoke scripts to send the internal token
    - Added the new secret to local and testnet environment templates

- Added focused internal-authentication tests:
    - Covered missing server configuration, missing request header, wrong token, prefix-only token, valid token, and surrounding whitespace
    - Verified unauthenticated and incorrect-token requests are rejected
    - Verified correct credentials retain the maintenance workflow
    - Confirmed the normal health endpoint remains available
    - Marked production deployment and `LOAVIX_PROD_ENV` secret configuration as required follow-up

- Published the Fiber Node Connector repository safely:
    - Confirmed the connector has its own Git repository and no tracked relationship to Loavix
    - Confirmed there are no Loavix names, workspace dependencies, runtime packages, real environment files, or committed secrets
    - Published seven incremental commits to `github.com/leothatguy/fiber-node-connector`
    - Confirmed all 12 connector tests passed before publication
    - Confirmed branch CI is green and dependency-update pull requests also pass
    - Left version tagging and the first public container release as a separate release action

- Verification completed across code and rendered behavior:
    - Passed 58 backend tests across seven suites
    - Completed clean backend and scripts builds
    - Completed frontend lint and production build with all six static pages generated
    - Checked rendered layouts at 1280, 1000, 834, 700, 390, and 320 pixels
    - Exercised populated balance, payout, and invoice states with temporary local seed data and removed the scaffolding afterward
    - Verified guarded internal routes with correct, incorrect, missing, and unconfigured credentials against a running backend

- Identified the next invoice-capital recovery milestone:
    - Add a merchant recovery action for expired on-chain invoice cells
    - Reuse the contract's existing cancellation path without turning recovery into a server-controlled spend
    - Verify the outpoint remains live and prevent duplicate recovery attachment
    - Preserve the invoice's expired business status while recording that its cell capacity was recovered

### Environment
- Report period ending: 08-10-2026
- CKB application: Loavix testnet invoice and Fiber payment system
- Backend security: NestJS guard plus Nginx route denial
- Internal authentication: `INTERNAL_API_TOKEN`
- Secret comparison: Node.js `timingSafeEqual`
- Frontend: Next.js responsive application routes
- Visual verification widths: 1280, 1000, 834, 700, 390, and 320 pixels
- Backend verification: 58 tests across seven suites
- Connector repository: `github.com/leothatguy/fiber-node-connector`
- Connector publication status: source pushed; versioned container release still pending
- Rust study focus: unsafe boundaries and safe abstractions
- CKB study focus: explicit block-header dependencies
- Loavix change status: implemented and verified locally; production deployment pending
