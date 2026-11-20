# BIASHARAONE_PROJECT_FILE_BLUEPRINT

## What this document is

This is a proposed, followable file map for a new BiasharaOne codebase. It explains the purpose and expected contents of the important authored files for:

1. **BiasharaOne Local** — Electron desktop POS, local API and local database; core checkout must work without the internet.
2. **BiasharaOne Cloud** — browser POS/business management with a separate business database for each business.
3. **BiasharaOne Admin** — a separate platform administration UI/API for businesses, licences, subscriptions, devices, provisioning, backups and support.
4. **Sync services** — a separately operated/delivered sync Gateway and till agent, not bundled as private source in a public/customer POS package.

The map is intentionally broad, but it does not list generated files, every translation, vendor file, or every one-off UI component. Those are created as features require them. It is a design document—not a claim that the files already exist, an implementation plan with approved scope, or legal clearance to copy Posnic code. The source-grounded current-state evidence is in [BIASHARAONE_DETAILED_TRANSFORMATION_MAP.md](./BIASHARAONE_DETAILED_TRANSFORMATION_MAP.md).

## Proposed baseline and decisions to confirm

### Baseline used by this map

- JavaScript / Node.js for the first implementation, matching the existing project's ecosystem; Electron for Local, Express for APIs, MongoDB/Mongoose for persistence, and a web frontend shared where the runtime allows it.
- One npm-workspace monorepo for public business applications, shared business code, database migrations, and tests. Deployable processes remain separate.
- One **control database** for platform records. One **business database per Cloud business**. Local uses its own single-business database.
- The same business API/core runs locally and in Cloud. Local startup supplies one local DB; Cloud startup must resolve a trusted tenant before tenant DB-backed session/auth/data access.
- The Admin UI/API is a separate application and security boundary from a business user's POS.
- A feature is not complete until its permissions, settings page, universal-search entry (if navigable), tests, data ownership, and sync decision are handled.

### Must be decided before implementation

Record each decision in `docs/decisions/` before it changes the corresponding design:

- Legal right to reuse each Posnic component, contribution, trademark, and separately licensed dependency; AGPL compliance and any commercial licence.
- Whether a bundled local MongoDB is a standalone server or a single-node replica set. Multi-document transactions require replica-set/sharded support; test every transaction path on the supported Local mode.
- Whether Local Cloud sync is enabled by explicit customer opt-in, what is authoritative, what data is copied, and how offline changes reconcile.
- Supported countries, tax/fiscal requirements, currency/rounding rules, privacy/data-retention rules, data residency and customer-support access.
- Local perpetual licence policy (device count, offline validation and transfer) and Cloud subscription/grace/suspension policy.
- Local API/browser access model, deployment regions, backups, RPO/RTO, key custody and operations ownership.
- How private sync components are licensed, hosted, delivered, updated and verified without embedding private source in a public/customer installer.

Until these are decided, keep the related service/configuration optional and disabled by default; do not make a local sale depend on licensing or sync availability.

## System at a glance

```text
BiasharaOne Local
  Electron main/preload <-> local renderer
       |                       |
       +---- local business API +---- local MongoDB
                    |
                    +---- optional authenticated sync agent
                               |
                               v
                         Sync Gateway

BiasharaOne Cloud
  Browser UI -> Business API -> trusted tenant resolver -> business DB for that tenant
                                  |
                                  +-> Control-plane API (limited provisioning/entitlement calls)

BiasharaOne Admin
  Admin UI -> Control-plane API -> Control DB
                    |
                    +-> audited provisioning / backup / support operations
```

The browser must never choose a MongoDB database name. The API resolves a verified business identity from a trusted domain/device mapping and loads the database mapping from the control plane. Business API code only accesses the currently selected business DB. Control-plane records and credentials never use the business DB as their authority.

## Proposed repository tree

`[file]` means an authored file; `[dir]` means a folder whose files are described below. Keep generated output, runtime data, secrets, and private sync implementation out of source-controlled public build inputs.

```text
biasharaone/
├── README.md
├── LICENSE                         # only after the actual licence is selected
├── NOTICE                          # upstream/third-party notices, if required
├── CONTRIBUTING.md
├── SECURITY.md
├── .gitignore
├── .npmrc
├── package.json                    # workspaces and local developer commands
├── package-lock.json
├── eslint.config.js
├── .prettierrc.json
├── .prettierignore
├── .env.example                    # names only; never credentials
├── apps/
│   ├── local-desktop/
│   ├── business-api/
│   ├── cloud-web/
│   ├── control-plane-api/
│   └── admin-web/
├── packages/
│   ├── business-contracts/
│   ├── business-domain/
│   ├── business-data/
│   ├── business-auth/
│   ├── tenant-context/
│   ├── control-plane-contracts/
│   ├── sync-contracts/
│   ├── shared-ui/
│   ├── configuration/
│   └── observability/
├── database/
│   ├── control/
│   └── business/
├── tests/
│   ├── fixtures/
│   ├── integration/
│   ├── security/
│   ├── migration/
│   └── acceptance/
├── scripts/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   │   └── 000-template.md
│   ├── operations/
│   ├── security/
│   └── product/
└── private-delivery-boundary.md
```

## Root files

| File | What it should contain |
|---|---|
| `README.md` | Product editions, supported platforms, prerequisites, local development commands, local-only quick start, links to architecture, security and operations docs. Never claim SaaS/sync readiness before the relevant evidence exists. |
| `LICENSE` | The chosen licence's complete text only after legal approval. Do not copy/remove Posnic notices as a shortcut to changing ownership or licence. |
| `NOTICE` | Attribution and third-party notices that the chosen licence/dependencies require. Keep notices for shipped binaries, fonts, icons and assets. |
| `CONTRIBUTING.md` | Human contributor rules, review process, coding/test conventions and legal contribution terms approved by the project owner. No AI/tool authorship credits. |
| `SECURITY.md` | Supported versions, private vulnerability reporting route, response expectations and safe disclosure policy. Do not publish secrets or exploitable customer data. |
| `.gitignore` | Dependencies, build output, test databases, logs, local `.env`, signing material, certificates, customer data and generated installers. Do not rely on ignore rules as the only release-content check. |
| `.npmrc` | Only intentional npm workspace/install policy; no credentials or private registry tokens. |
| `package.json` | Workspaces, pinned/runtime engines, scripts for test/lint/type-independent static checks/build/package/attribution. Avoid adding CI workflows or jobs without explicit approval. |
| `package-lock.json` | Reproducible dependency graph generated by the selected package manager. Review licence/advisory changes on dependency updates. |
| `eslint.config.js`, `.prettierrc.json`, `.prettierignore` | Consistent lint/format rules and generated/vendor exclusions. |
| `.env.example` | Document variable names and safe local defaults; values must be blank or unmistakably non-secret examples. Keep production credentials in a secret manager, not source or installer files. |
| `private-delivery-boundary.md` | Public/private repository boundary, authorized sync-agent delivery method, package exclusions, release inspection steps and ownership contacts. This is a policy document, not a way to include private source in the POS package. |

## `apps/local-desktop/` — BiasharaOne Local

### Desktop shell and local runtime

| File | What it should contain |
|---|---|
| `package.json` | Electron entry, builder identity, allowed packaged files, supported installer targets, signing/update configuration references and scripts. Product name, app ID, protocol, publisher, icons and update host must all be BiasharaOne-owned/authorized. |
| `src/main.js` | Electron lifecycle, window creation, app lock, startup/shutdown orchestration and service composition. It should delegate business rules and avoid direct sale/database logic. |
| `src/preload.js` | Small `contextBridge` API with explicit capabilities. Never expose unrestricted `ipcRenderer`, Node `require`, filesystem or arbitrary channel invocation to the renderer. |
| `src/ipc/channels.js` | Central list of allowed IPC channels, argument schemas and return contracts. |
| `src/ipc/register-handlers.js` | Register validated, permission-checked handlers; handlers call narrow services and surface failures explicitly. |
| `src/ipc/window-guard.js` | Validate sender frame/window, origin and lifecycle for privileged IPC. |
| `src/services/local-api.js` | Start, monitor, health-check and stop the local business API; choose a protected loopback port and report startup errors. |
| `src/services/mongodb.js` | Start/stop the supported local database binary, select its data directory, enforce loopback binding/auth policy, and handle recovery/shutdown. No user-data directory or credentials should be hard-coded. |
| `src/services/user-data.js` | Stable app-data path, permissions, migration between app identity changes, local config and log locations. Preserve users' existing data during upgrades. |
| `src/services/backup.js` | User-triggered/scheduled local backup, manifest, integrity checks, restore preview, explicit restore confirmation and failure reporting. A successful export is not a verified restore. |
| `src/services/updates.js` | Signed update discovery/download/install orchestration, user settings and rollback/backup hooks. Local checkout/trading remains available offline. |
| `src/services/printer.js` | Enumerate printers and route receipt/report/PDF jobs to OS-specific adapters; record job lifecycle and recovery without replaying a completed print automatically. |
| `src/services/scanner.js` | Keyboard-wedge/barcode input support and any approved device integration. Barcode scanning should not require an internet provider. |
| `src/services/diagnostics.js` | Collect minimum necessary diagnostics only after explicit user action/consent; redact secrets, sale data and customer data before export/upload. |
| `src/windows/main-window.js`, `src/windows/splash-window.js` | Window options, navigation allowlist, secure web preferences and splash/startup UX. |
| `src/assets/app.ico`, `src/assets/app.png`, `src/assets/installer-*` | BiasharaOne assets for each platform; verify licenses and inspect actual installer resources. |
| `build/electron-builder.yml` | Builder config: app ID, product name, protocol, artifacts, platform targets and explicit include/exclude rules. Never auto-discover a private sibling checkout. |
| `build/sign-windows.js` (or approved platform signer) | Invoke the human-owned release signing process without storing a certificate/private key in the repository. |

### Renderer

| File | What it should contain |
|---|---|
| `renderer/index.html` | Local renderer entry, CSP, accessible app shell and static assets. No remote script execution. |
| `renderer/src/main.js` | Renderer bootstrap, dependency wiring and global error boundary. |
| `renderer/src/router.js` | Allowed local destinations, route guards and deep-link mapping. |
| `renderer/src/api/client.js` | Business API transport, timeout/error handling, CSRF/session/device auth and offline/network-state reporting. Never accept a database name or arbitrary API host from a UI field. |
| `renderer/src/auth/` | Sign-in, lock/unlock, local session state and permission-aware navigation. Do not store plaintext passwords or reusable secrets in browser storage. |
| `renderer/src/features/` | POS screens grouped by business modules (sales, products, inventory, customers, purchases, expenses, reports, branches, users, settings). Each screen calls the business API and does not implement a second persistence path. |
| `renderer/src/search/catalog.js` | Register every navigable page, list/detail/create action and setting section for universal search. Use translated labels, operator aliases, module/feature gates, permissions and stable record resolvers. Add actions open forms; they never submit them. |
| `renderer/src/search/action-phrases.js` | Shared operator-action phrase index for search and command palette. |
| `renderer/src/lists/toolbar.js` | Shared search/filter/sort/page controls. Filters must reach the authorized local query before count, totals, export and pagination. Show Branch only if the user can access multiple branches. |
| `renderer/src/settings/` | Module-owned settings pages/tabs. A Features screen contains switches only; actual controls belong on their module page. |
| `renderer/src/styles/`, `renderer/src/i18n/` | Shared themes, responsive/narrow layouts, accessibility and translations. |
| `renderer/src/hardware/` | UI integration with the narrow preload printer/scanner/scale APIs; graceful device-unavailable state. |

## `apps/business-api/` — shared business API runtime

Run this API locally for Local and centrally for Cloud. Runtime mode changes the trusted DB resolver, not the business authorization rules.

| File or directory | What it should contain |
|---|---|
| `package.json` | API runtime dependencies and local test/lint/start scripts. Keep business tender gateways distinct from platform subscription billing. |
| `src/server.js` | Process entry: load validated config, connect to the appropriate database service, apply approved migrations, start server and close resources gracefully. |
| `src/app.js` | Compose HTTP middleware and route modules in one reviewed order. Health endpoints should not reveal tenant/customer information. |
| `src/config/index.js` | Validate required environment settings and deployment mode at startup; fail clearly instead of using unsafe defaults. |
| `src/middleware/request-id.js` | Create/propagate a safe request ID for logs and support. |
| `src/middleware/tenant-resolution.js` | Cloud-only resolver using trusted host/domain or authenticated device mapping plus control-plane lookup. Reject unknown/suspended tenants. Never trust a client-supplied database name. |
| `src/middleware/tenant-scope.js` | Open AsyncLocalStorage with selected tenant DB, tenant-specific secret references and verified context before DB-backed sessions/auth. Missing scope fails closed in Cloud. |
| `src/middleware/session.js` | Load sessions against the correct business/control authority; secure cookie flags, rotation and expiry. |
| `src/middleware/csrf.js` | CSRF protection for cookie-authenticated mutations. |
| `src/middleware/authenticate.js` | Verify user/device credentials using the selected tenant context; no cross-tenant shared fallback secret. |
| `src/middleware/authorize.js` | Enforce business membership, branch access, role permissions, feature gates and action-level rights. |
| `src/middleware/validate.js` | Run request schemas and reject unknown/invalid fields at the boundary. |
| `src/middleware/rate-limit.js` | Rate limits for login, recovery, device pairing and expensive endpoints, backed by the right deployment-wide store in Cloud. |
| `src/middleware/error-handler.js` | Stable error contract, safe client messages, structured server diagnostics; no swallowed database/security errors. |
| `src/routes/index.js` | Mount route modules after common context/auth middleware according to route access requirements. |
| `src/routes/auth.routes.js` | Business login/logout/session/password workflows. |
| `src/routes/sales.routes.js` | Sale creation, list/detail, payment-recording and return/cancel routes. Validate idempotency and authorization. |
| `src/routes/items.routes.js`, `categories.routes.js`, `inventory.routes.js` | Product/catalogue and stock operations. |
| `src/routes/customers.routes.js`, `suppliers.routes.js` | Party records and privacy/branch scoping. |
| `src/routes/purchases.routes.js`, `expenses.routes.js`, `reports.routes.js` | Purchasing/receiving, expenses and authorized reporting/export. |
| `src/routes/branches.routes.js`, `users.routes.js`, `settings.routes.js` | Business administration; no platform-admin powers. |
| `src/controllers/` | HTTP parsing/response mapping only. Controllers call domain use cases; they do not choose a tenant DB from request data. |
| `src/jobs/` | Scheduled work with explicit tenant iteration/context, durable leases and idempotent handlers. A job must never inherit an arbitrary previous request's tenant. |
| `src/realtime/` | Tenant/branch-scoped invalidation or event delivery. Events carry no data a client is not authorized to fetch. |
| `src/files/` | Tenant-scoped upload/download authorization, MIME/size limits, storage adapter and deletion. Keep file permissions aligned with the DB record. |

### Recommended middleware order

1. Request ID, trusted proxy/IP settings, body-size limit and basic rate limit.
2. Parse/validate host and resolve tenant in control plane (Cloud only).
3. Open tenant context and bind DB/secret references; local mode uses its one configured business DB.
4. Load session, CSRF and authentication within the selected authority.
5. Check business membership, branch, roles, permissions and optional feature enablement.
6. Validate route input, execute domain operation, audit, enqueue sync if explicitly configured, and return a bounded response.

Health/readiness, device-key, kiosk and webhook routes need explicit access policies; “registered before the normal auth middleware” must not mean “outside tenant isolation.”

## `packages/` — shared contracts and business behavior

| Package | Expected files and contents |
|---|---|
| `business-contracts/` | `index.js`, `sales.js`, `inventory.js`, `parties.js`, `branches.js`, `users.js`, `settings.js`, `errors.js`; request/response/event schemas shared by browser, desktop and API. Avoid leaking database documents as public API contracts. |
| `business-domain/` | `sales/create-sale.js`, `sales/refund.js`, `inventory/adjust-stock.js`, `purchases/receive-goods.js`, plus customer/supplier/expense/report use cases. Keep money math, idempotency, state transitions and domain validation independent of HTTP/UI. |
| `business-data/` | `connection.js`, `model-registry.js`, models (`sale.js`, `item.js`, `stock-movement.js`, `user.js`, etc.), repositories, indexes and transaction helpers. Every repository obtains its handle from explicit tenant context; no global mutable DB singleton. |
| `business-auth/` | Password hashing, token/session handling, branch membership, permission evaluation and secret/key interfaces for business users. Platform identities must use a separate package/namespace. |
| `tenant-context/` | Validated tenant context, AsyncLocalStorage wrapper, connection selection, fail-closed accessors and tests for missing/concurrent contexts. |
| `control-plane-contracts/` | Schemas for business, plan, entitlement, subscription, device, provisioning, support grant and backup status exchanges. Contains no provider secrets or database connection strings. |
| `sync-contracts/` | Versioned event envelope, stable IDs, ordering/checkpoint/tombstone/idempotency and schema-version policy. It defines contracts only; it is not sync transport or authority. |
| `shared-ui/` | Accessible shared components, table toolbar, form controls, permission-aware route helpers, translations and theme tokens. Do not put module settings controls on the Features switch grid. |
| `configuration/` | Shared environment parsing and safe defaults. Separate schemas for Local, business Cloud and control-plane deployments. |
| `observability/` | Structured redacting logger, metrics/event names and trace context. Explicitly exclude passwords, tokens, prompt content, payment secrets and unnecessary customer data. |

Each package has `package.json`, an entry module, and tests next to or under `test/`. Add package-specific README files only where the package's public contract is not self-explanatory.

## `database/` — schemas and migrations

| File/folder | What it should contain |
|---|---|
| `database/control/README.md` | Control DB ownership, backup/restore, migration and access boundary. |
| `database/control/migrations/0001-platform-core.js` | First approved control schema: businesses, owners/memberships, plans, licences/entitlements, subscriptions, devices and provisioning. No sample customer data or secrets. |
| `database/control/migrations/0002-support-and-audit.js` | Audited support grants, admin actions and control-plane audit records with retention policy. |
| `database/control/indexes.js` | Unique/business lookup indexes and migration checks for control entities. |
| `database/business/README.md` | Business DB ownership, tenant data inventory, migration and restore rules. |
| `database/business/migrations/0001-business-core.js` | Business identity/settings, users, branches, roles and core catalogue; business records remain inside that business DB. |
| `database/business/migrations/0002-sales-inventory.js` | Sales, embedded sale lines or selected sale-line model, payment records, stock movements and inventory invariants. |
| `database/business/migrations/0003-purchasing-parties.js` | Customers, suppliers, purchases/receivings and expenses. |
| `database/business/indexes.js` | Query indexes with tenant DB context, unique constraints and safe index rollout. |
| `database/migration-runner.js` | Ordered, repeatable, ledgered migrations; dry-run, backup gate, resume and explicit rollback limitations. |
| `database/migration-data/` | Versioned transformations for importing existing shop data only after schema and licence decisions. Never silently change production data. |

Control-plane and business migrations are separate processes. The control-plane API must not be a backdoor into arbitrary business databases; provisioning operations are narrowly scoped and audited.

## `apps/control-plane-api/` and `apps/admin-web/`

### Control-plane API

| File | What it should contain |
|---|---|
| `apps/control-plane-api/src/server.js`, `app.js`, `config.js` | Independent process/bootstrap, strict configuration and control DB only. |
| `apps/control-plane-api/src/middleware/admin-auth.js` | Platform staff authentication (MFA-capable), session lifecycle and privileged action checks. Never reuse shop cashier tokens as platform-admin tokens. |
| `apps/control-plane-api/src/middleware/admin-authorize.js` | Fine-grained platform roles, least privilege and support-access checks. |
| `apps/control-plane-api/src/routes/businesses.routes.js` | Register/inspect/suspend/reactivate businesses and owners with audit. |
| `apps/control-plane-api/src/routes/licences.routes.js` | Issue/revoke/transfer perpetual entitlements and device rules. |
| `apps/control-plane-api/src/routes/subscriptions.routes.js` | SaaS plan/status, renewal/grace/suspension transitions and manual-payment records. |
| `apps/control-plane-api/src/routes/devices.routes.js` | Device enrollment, revocation and transfer workflow. |
| `apps/control-plane-api/src/routes/provisioning.routes.js` | Tenant DB provision/migrate/deprovision orchestration with durable state and retries. |
| `apps/control-plane-api/src/routes/backups.routes.js` | Backup status and restore requests; restore is privileged, confirmed and audited. |
| `apps/control-plane-api/src/routes/support.routes.js` | Support queue and time-bounded, customer-approved support access; record every access. |
| `apps/control-plane-api/src/routes/webhooks.routes.js` | Provider signature validation, idempotent billing webhook ingestion and replay protection. |
| `apps/control-plane-api/src/services/` | Business lifecycle, licensing, subscriptions, device registry, provisioning, backup orchestration and audit use cases. |
| `apps/control-plane-api/src/repositories/` | Control DB access only; never query a business collection without a separately authorized provisioning/support operation. |

### Admin web UI

`apps/admin-web/src/` should contain a separate bootstrap/router/API client, sign-in/MFA flow, dashboards and pages for businesses, owners, licences, plans/subscriptions, devices, branches metadata, provisioning, backups, software versions, support requests and platform statistics. Every write view needs explicit confirmation and a permission gate. Cross-tenant search and data display must be minimized and audited; the Admin UI should not silently impersonate a business user.

## `apps/cloud-web/`

| File or directory | What it should contain |
|---|---|
| `src/main.js` | Cloud business UI bootstrap and shared business routes/components. |
| `src/router.js` | Business navigation, guards, branch-aware route resolution and not-found behavior. |
| `src/api/client.js` | Same business contracts as Local with secure cookie/token policy and network errors. |
| `src/features/` | Business POS, catalogue, customers, purchases, expenses, reports, branches, users and module settings. |
| `src/search/catalog.js`, `src/search/action-phrases.js` | Register every eligible list/detail/create destination and setting section in All; respect permissions, feature gates and aliases. |
| `src/lists/` | Shared list toolbar and query adapter; apply changed filters before count/totals/export/pagination and test narrow layouts. |
| `src/settings/` | Each feature owns its settings page/tab; Feature cards show enabled/disabled switches only. |
| `src/realtime/` | Tenant-authorized live invalidation/subscription client; reconnect without leaking cross-tenant data. |
| `src/styles/`, `src/i18n/` | Responsive, accessible design system and translated labels/aliases. |

The Cloud UI uses the business API, never a direct database connection. Local and Cloud screens may share components, but capabilities such as direct printer access must be feature-detected and have a browser-safe alternative or a clear unavailable state.

## Sync Gateway and Local-cloud agent — separate delivery boundary

Create separate repositories/deployment artifacts for the Gateway and till agent if they are private or separately licensed. `private-delivery-boundary.md` describes ownership and packaging policy; the POS builder must not copy source from a sibling checkout.

**Gateway repository** should include, at minimum: service entry/config, device authentication, tenant/business resolver, collection allowlist/scope registry, pull/push protocol handlers, checkpoints, idempotency, tombstones, conflict policy, rate limiting, audit, health/metrics, migrations, tests and deployment/backup runbooks.

**Agent repository** should include: signed installation/update bootstrap, local DB connector, explicit config schema, outbound-only authenticated transport, per-collection pull/push workers, durable outbox/checkpoint handling, bounded retries/backoff, duplicate detection, conflict/deletion handling, file-reference transfer, health status, tests and install/upgrade/uninstall procedures.

The POS monorepo may contain only public **contract** and adapter files such as `packages/sync-contracts/` and `apps/local-desktop/src/services/sync-status.js`. It must not include private Gateway/agent source in Local source, `resources`, archives or customer release files. Sync is opt-in in the owning module's settings, default off, and cannot replay payments, stock movements, messages or print jobs.

## `tests/` — required quality gates

| File/folder | What it should prove |
|---|---|
| `tests/fixtures/` | Synthetic business/branch/users/items/sales; no customer production data or secrets. |
| `tests/integration/tenant-isolation.test.js` | Two concurrent tenants cannot read/write one another's models, direct collections, sessions, exports, search, files or errors; missing tenant context fails closed. |
| `tests/integration/sales.test.js` | Sale, tender record, duplicate retries, returns/refunds, totals and authorization preserve invariants. |
| `tests/integration/stock.test.js` | Concurrent reservations, receiving, adjustment, reversal and branch isolation maintain ledger/balance consistency. |
| `tests/integration/auth-branches.test.js` | Login, session expiry/revocation, owner/staff roles and branch access. |
| `tests/integration/backup-restore.test.js` | Create backup, verify manifest/hash, restore to isolated DB and compare required business state. |
| `tests/integration/migrations.test.js` | Repeatable migration, preflight backup, index creation, existing-data mapping and failure/resume. |
| `tests/security/` | Tenant-context bypass, IDOR, upload abuse, CSRF, rate limits, secret redaction, support access and admin/business boundary. |
| `tests/migration/` | Compatibility fixtures from legally cleared source versions; source lineage and expected transformation are documented. |
| `tests/acceptance/local-offline.test.js` | Local checkout and local printing/data persistence with network unavailable; reconnection must not duplicate operations. |
| `tests/acceptance/sync-directions.test.js` | If sync is approved: cloud-to-till, till-to-cloud and second authorized till; create/update/delete, retry, conflict, branch isolation and referenced files. Run with the actual Gateway and delivered agent. |
| `apps/local-desktop/test/` | IPC sender checks, startup/shutdown, DB recovery, update signing, backup-before-update and platform printer behavior. Include Windows printer tests on Windows. |
| `apps/business-api/test/` | Middleware order, route permissions, API contracts, invalid input and error behavior. |
| `apps/control-plane-api/test/` | Provisioning idempotency, suspension/grace/reactivation, licence transfer, subscription webhook replay and audited support access. |
| `apps/*/test/` | UI navigation/search, restricted-user search results, View/Add destination behavior, table filters beyond page one, clear filters and responsive layouts. |

Passing a manifest comparison or unit test is not evidence that a live device receives the intended sync or that a backup can be restored.

## `scripts/` — local tooling

| File | Purpose |
|---|---|
| `scripts/check-config.js` | Validate documented example config and ensure no production secret is present. |
| `scripts/check-attribution.js` | Detect prohibited software-tool credits and assistant footers in files/commit messages. |
| `scripts/check-sync-coverage.js` | Compare business collections and sync contracts to Gateway/agent manifests when authorized sources are available; fail on undecided collections. |
| `scripts/check-package-contents.js` | Inspect unpacked installers and nested archives for secrets, customer data, logs, stale private files and unapproved components. |
| `scripts/migrate-shop.js` | Explicitly invoked, dry-run-capable import tool; requires backup, target tenant confirmation, mapping report and operator confirmation. |
| `scripts/` test runners | Run existing local tests/lint/format/package checks; do not add GitHub Actions/workflows unless explicitly requested. |

## `docs/` — required operator and developer material

| File | What it should explain |
|---|---|
| `docs/architecture/overview.md` | Process boundaries, Local/Cloud/Admin responsibilities, data flows and trust boundaries. |
| `docs/architecture/data-model.md` | Control vs business entities, relations, branch scope, indexes and ownership. |
| `docs/architecture/tenant-request-flow.md` | Trusted tenant lookup, middleware order, secret/session binding and background-job scope. |
| `docs/architecture/sync-contract.md` | Per-collection authority, directions, conflicts, idempotency, tombstones, file handling and opt-in behavior. |
| `docs/architecture/decision-log.md` | Index of accepted/rejected architecture decisions linking each record in `docs/decisions/`. |
| `docs/decisions/000-template.md` | Required decision-record format: context, options, choice, tradeoffs, date, approver and follow-up. |
| `docs/decisions/` | One short record per consequential decision using the template above. |
| `docs/security/threat-model.md` | Assets, actors, boundaries, abuse cases, mitigations and review evidence. |
| `docs/security/secrets.md` | Secret issuance, storage, rotation, revocation and redaction. Never put actual secret values here. |
| `docs/operations/backup-restore.md` | Backup schedule, encryption, retention, restore drills, RPO/RTO and customer export. |
| `docs/operations/release.md` | Human release approval, signing keys, artifact inspection, update channels and rollback. |
| `docs/operations/incident-response.md` | Incident roles, containment, evidence, customer notification and recovery steps. |
| `docs/product/local-licensing.md` | Approved Local licence, offline validation, grace, transfer and support policy. |
| `docs/product/cloud-subscriptions.md` | Plans, billing, renewal, grace, suspension/reactivation and platform-vs-shop-payment distinction. |
| `docs/product/privacy-and-retention.md` | Data collected, consent, deletion/export and retention by feature. |
| `docs/third-party-notices.md` | Current dependencies, versions, licences, required notices and shipped-vs-build-only status. |
| `docs/test-strategy.md` | Local commands, coverage layers, hardware requirements and what remains unverified. |

## File responsibilities and boundaries

- **Controllers translate HTTP.** Domain rules live in use cases; repositories own persistence.
- **A business DB connection is request-scoped.** No global “current tenant” mutable variable and no process-default database fallback in Cloud.
- **Platform and shop identity are distinct.** A shop owner/admin is not automatically a platform administrator.
- **Customer sales tenders and BiasharaOne subscriptions are separate ledgers.**
- **Optional workflows default off.** Explicit shop opt-in is stored in the owning module's settings; preserve saved settings on upgrade.
- **Every new persisted field/write/delete has a sync decision.** If it is intentionally local-only, state why and define how users see it elsewhere.
- **Settings live with their module.** Features shows switches only. Search settings open the exact owning section/control.
- **Navigation is permission-aware.** Every list/detail/create and report destination is discoverable in All; Add opens the existing form and never submits it.
- **Do not claim release readiness** until the correct app, API, database migrations, Gateway/agent delivery (if used), backups and regression tests are present together and verified.

## Suggested build order

1. Resolve legal and third-party rights; approve this architecture and record open decisions.
2. Create root workspaces, configuration/secret policy, business/control contracts and basic local test commands.
3. Build the local business API + local DB boot path, then a small vertical slice: login, one item, one checkout, local receipt and backup/restore.
4. Add inventory, customers, suppliers, purchasing, expenses, reports and business settings, preserving the sales/stock financial invariants.
5. Add tenant context and control-plane provisioning; prove tenant isolation before adding production tenants.
6. Build Admin identity and audited business/licence/subscription/device workflows.
7. Build Cloud web against the same business contracts and tenant-aware API.
8. Approve sync semantics and separately deliver/test the Gateway and agent; keep sync opt-in.
9. Add signed updates, Local licences, SaaS billing, support and operational monitoring after legal, threat-model and recovery reviews.
10. Run local tests, Windows printer checks, attribution checks, license/package-content inspections and restore drills before any customer pilot.

This blueprint is a starting specification. It intentionally does not generate these files or claim that all implementation choices are settled.
