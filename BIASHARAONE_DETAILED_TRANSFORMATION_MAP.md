# BIASHARAONE_DETAILED_TRANSFORMATION_MAP

**Read-only source review.** This report describes the checkout as inspected; it does not change application behavior or establish legal rights. Findings are static unless explicitly marked otherwise. No dependency installation, database access, migration, build, test, or packaging command was run.

## Scope and evidence notes

- The working tree was already dirty before this report: `package.json`, `package-lock.json`, `api/package-lock.json`, and `src/download-mongodb.bat` are modified; `.vscode/` is untracked. These changes were not made or reverted for this audit. Dependency versions below are the versions visible in the current manifests and may not represent the repository's committed baseline.
- The repository has 6,308 tracked files. A case-insensitive tracked-text scan for `posnic`, `PosnicPro`, `Posnic Innovations`, and related forms found 24,344 matches in 1,230 files and 21,252 distinct file:line locations. Matches are heavily repeated in frontend/localization, vendor assets, tests, and docs. The inventory below prioritizes operationally significant occurrences rather than reproducing every repeated translation or vendor-string hit. The result is not a one-row-per-match appendix.
- The sync manifest explicitly describes the Gateway map as authoritative and checks its local mirror during build. Gateway and sync-agent source are not included in this checkout; their behavior and successful end-to-end delivery cannot be independently verified here.
- References in this report use repository-relative paths and source line anchors for navigation. Do not treat the absence of a listed string as proof that no equivalent behavior exists under a different name.

## 1. Executive summary

Posnic is a mature offline-capable Electron desktop POS with a local Express API, MongoDB, broad retail/restaurant workflows, hardware access, local backup, and optional cloud pairing. The UI assets are a separate Gulp-built static frontend rather than a separate SPA application with its own framework runtime. The desktop installer can package MongoDB and an API runtime; a locally hosted API can also serve browser clients.

The checkout is **not ready to be treated as an already operating BiasharaOne SaaS platform**. It contains tenant-database building blocks (`tenant-context`, `tenant-connections`, and a connection-aware model registry), but the normal `api/server.js` startup connects one `MONGODB_URI`, and the reviewed startup path does not enable multi-tenant mode or install a trusted per-request tenant resolver. `api/app.js` attaches a database handle, but that is not itself tenant resolution. The multi-tenant boundary is therefore scaffolding, not proof of tenant-isolated SaaS.

The existing shop-level `license` ObjectId and `branch_access` are legacy business-account/branch scoping concepts, not a platform licence or subscription system. No BiasharaOne control plane, platform owner dashboard, SaaS billing lifecycle, or local perpetual-product activation service was confirmed. Cloud pairing/activation, update checks, branding, sync, support, and other services are coupled to Posnic-owned endpoints or a separately delivered Gateway/agent.

The code is explicitly AGPL-3.0-only according to the root `LICENSE`, package manifests, and governance document. AGPL does not prohibit selling the software, but distribution obligations apply, and AGPL-3.0 section 13 addresses modified versions used over a network. Have qualified counsel determine the exact source-offer and notice obligations for the intended distribution and SaaS model. The governance document also says contributors retain their copyright and the Posnic trademarks/logo are reserved. You may not infer permission to close-source, relicense, or reuse every bundled component merely because the source is available.

**Recommended direction:** preserve the local POS as an independently installable edition while separating (1) a platform control plane, (2) per-business data databases, and (3) tenant-scoped application services. For cloud, prefer a control database plus a separate business database per tenant, consistent with the stated product goal; the existing `useDb` pooling helper can inform implementation but is not yet a complete isolation boundary. Treat sale/payment/stock changes as durable business events with idempotent delivery and reconciliation, not as ordinary last-writer-wins documents.

## 2. Original-project reference inventory

### High-signal references and replacement implications

| File and line | Observed reference / purpose | Classification | BiasharaOne replacement and break risk |
|---|---|---|---|
| [`package.json`](package.json#L2):2-6,70-78,684-720 | npm/package identity, product name, app ID, `posnic://` protocol, publisher metadata, author/license, GitHub repository/issues, homepage | Branding, legal attribution, release identity | New product identity, protocol, publisher/signing identity, repository and support URLs. Coordinate application ID and protocol migration; careless changes can break deep links, Windows app identity, update discovery, and user-data continuity. Do not remove legal notices without counsel. |
| [`api/package.json`](api/package.json#L1):1-5,35-45,113-118 | API name/description, keywords, author, license, repository and issue links | Branding, legal attribution | Replace package/docs/support identity only after legal review; package name changes can affect build scripts and generated runtime archives. |
| [`frontend/package.json`](frontend/package.json#L1):1-5,11-13,25-33 | `posnic-frontend`, Posnic description/author and repository metadata | Branding, attribution | Rebrand descriptions and metadata while preserving the frontend build contract. |
| [`api/server.js`](api/server.js#L29):29-52 and [`api/src/config/config.js`](api/src/config/config.js#L13):13 | Default MongoDB database name `PosnicPro`; single startup connection | Technical identity and data default | New database naming needs an explicit existing-data migration/upgrade plan; changing a string alone can make existing installations appear empty. |
| [`api/src/config/auth.js`](api/src/config/auth.js#L20):20-21,89 | Default JWT issuer/audience and session-cookie namespace use PosnicPro | Runtime identity | New issuer/audience/cookie names need staged token/session invalidation or migration; must not weaken authentication. |
| [`src/main.js`](src/main.js#L107):107-125,1165-1188,1291-1398,1629-1737 | Posnic deep-link scheme, Cloud activation/pairing, device identity, signup/captcha and Gateway default | Branding and operational Cloud dependency | Replace only with an authorized BiasharaOne service contract. Removing it disables pairing/Cloud account flows and may strand existing installations. |
| [`src/main.js`](src/main.js#L1488):1488-1535,2941-3100 | Gateway brand/limits and support upload resolution; URLs may be derived from configured Gateway | Technical dependency and support workflow | New authenticated endpoints and consent/privacy terms required. Deleting the feature breaks branding/limits or support upload. |
| [`src/sync-agent-manager.js`](src/sync-agent-manager.js#L4):4-11,73-118,195-245,330 | Agent is external/private, installed after activation; local config contains Gateway URL/device token; protocol is a contract with Posnic/Gateway | Technical dependency | Must be replaced or separately licensed/authorized with a BiasharaOne sync service and compatible agent. It is not safe to treat as a local POS source folder or include private components in a public/customer package. Removing it disables sync and associated device workflows. |
| [`src/update-service.js`](src/update-service.js#L1):1-5,56-105 and [`dev-app-update.yml`](dev-app-update.yml#L1):1-6 | electron-updater/GitHub release process and Posnic updater cache identity | Technical dependency and branding | New update host, signing keys, rollout channels and cache identity. Changing the feed without a transition plan can stop updates; a signing-key change can prevent update verification. |
| [`api/.env.example`](api/.env.example#L21):21-45,58 | Posnic-managed AI/knowledge/billing variable names, provider credentials placeholders and `PosnicPro` DB default | Configuration and external service coupling | Define BiasharaOne-owned configuration, secrets and service contracts. Example placeholders are not credentials; this report does not disclose secret values. |
| [`api/src/config/config.js`](api/src/config/config.js#L92):92-96,125-126 | Default MSG91 SMS URL/sender and `POSNIC_KEY`/`POSNIC_SECRET` names | External provider configuration and product namespace | Replace namespaces and default sender, preserve user-configured provider behavior, and ensure credentials never migrate across business boundaries. |
| [`src/backup-manager.js`](src/backup-manager.js#L15):15,28-29,164,677,817,973 | `posnic-backup-*` directory naming and `PosnicPro` backup database default | Branding plus data/recovery contract | New backup naming can be added compatibly; restore discovery currently recognizes the Posnic prefix, so removing/renaming it without backward compatibility can hide old backups. |
| [`src/escpos-unicode.js`](src/escpos-unicode.js#L107):107,152-162 | `posnic-receipt-*` temp directory and `#posnic-print-frame` marker | Internal identity, not an external dependency | Rename only with the paired renderer/test contract. A mismatch can break Unicode receipt rendering; this is not a reason to remove the printing implementation. |
| [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md#L1):1-22,50-87,104-139 | Posnic Innovations ownership, AGPL/open-core boundary, trademark and contributor-copyright statements | Legal attribution and rights evidence | Do not rewrite or remove these statements to imply transfer of ownership. A BiasharaOne fork should clearly distinguish its own changes and preserve upstream notices as required. |

### Reference categories, coverage, and limits

The tracked scan found references across API code, desktop code, frontend and translations, tests, build scripts/assets, package metadata, and documentation. High-signal technical occurrences include the paths in the table above plus `src/cloud-activation.js`, `src/asset-updater.js`, `api/src/services/ai.service.js`, `api/src/services/sms-providers.js`, `api/src/services/whatsapp-providers.js`, and `api/src/utils/s3.js`. The 21,252 distinct file:line locations include many low-signal translated labels, old compatibility strings, comments, test fixtures, and minified/vendor copies; those should be triaged by runtime reachability rather than mass-edited.

**Inventory completeness caveat:** the available report does not print all 21,252 hit lines or assign a bespoke replacement/break-risk explanation to every repeated text occurrence. It provides exact citations for the architectural and release-critical references. A release-safe cleanup should generate a reviewed file-by-file inventory from the repository at the implementation stage, distinguish source/runtime assets from vendor/fixtures/docs, and inspect every actual distributed asset before removing or replacing it.

## 3. External service dependency map

No runtime traffic capture was performed. The following are source-confirmed call sites/configuration points; enablement and exact deployment targets can depend on installation settings.

| Service / endpoints seen in source | Caller and configuration | Required? | Replacement work |
|---|---|---|---|
| Posnic Gateway: `/v1/activate`, `/v1/device/identity`, `/v1/pairing/redeem`, `/v1/brand`, `/v1/limits` | `src/main.js:1291-1535,1629-1706`; activation URL is validated by `src/cloud-activation.js:5-27`; device URL/token are persisted in local profile/config | Optional for standalone POS; required for current cloud pairing, sync identity, brand/limit fetch | Define a BiasharaOne-owned activation, device, tenant identity, branding and entitlement API; do not invent compatibility semantics. |
| Posnic website/account/captcha and support | `src/main.js:1689-1737,2941-3100`; account creation and support submissions | Optional, but support/account workflows depend on it | Rebuild account/support services and privacy/consent model, or disable only through a deliberate product decision. |
| GitHub Releases updater | `package.json` publish owner/repo and `src/update-service.js`; `dev-app-update.yml` | Optional to local POS operation; operationally important for updates | New release repository, signed update chain, versioning, rollout, rollback and old-client transition. |
| Sync Gateway and separately delivered agent | `src/sync-agent-manager.js:4-11,73-118,195-245`; `api/src/sync/collections.json` | Optional local-only use; required for current multi-install sync | Gateway and agent sources are absent from this checkout; secure delivery, tenant routing, conflict semantics and end-to-end validation are required. |
| SMTP/Brevo/email | `api/src/utils/email.js`, messaging/config code, `api/package.json` Nodemailer/Brevo packages | Optional (email reminders, messages, verification) | Own sender domain, credentials, bounce/abuse handling, templates, consent and provider configuration. |
| SMS / WhatsApp providers | `api/src/services/sms-providers.js`, `sms.service.js`, `whatsapp-providers.js`; shop settings and provider credentials | Optional and shop-configurable | Maintain provider adapters and per-shop secret custody; confirm provider-specific consent/compliance. Providers include Twilio, Vonage/Nexmo, Plivo, Infobip, Brevo, MessageBird, Sinch, Telnyx, MSG91, Textlocal, Africa's Talking and others visible in the adapters. WhatsApp adapters include Meta Cloud API, Brevo, Twilio, MSG91, 360dialog, Gupshup and Infobip. |
| AI APIs and managed AI services | `api/src/services/ai.service.js:103-486`; `api/src/services/bedrock-provider.js`, `transcribe.service.js`, `ask-posnic-*`; `api/.env.example:21-45` | Optional | Replace Posnic-managed credentials/knowledge/billing endpoints with a BiasharaOne-owned service or explicit customer keys, per-tenant budget/security controls, and opt-in settings. Direct provider use is not itself platform subscription billing. |
| AWS S3, S3 Vectors, Textract, Bedrock | `api/src/utils/s3.js:65`; `api/src/services/paper-order.js`, `ask-posnic-*`; AWS dependencies in `api/package.json` | Optional (uploads, scans, AI/knowledge) | Configure tenant-safe bucket/key separation, retention, access policy, encryption and deletion/export controls. Validate referenced-file sync separately from DB-row sync. |
| Payment providers | API dependency `razorpay`; `api/src/models/sale.model.js` and sales/mobile/captain payment routes; Razorpay webhook collection | Optional shop tender/payment workflows | These are customer sales payments, not platform subscription charges. SaaS billing needs a separate platform ledger and provider/account boundary. |
| Google Analytics / shop analytics | `api/src/services/analytics-config.js` and branch settings | Optional and shop-configured | Confirm the owner and consent of each analytics property. No global shop telemetry is established by this source scan; governance explicitly says no Posnic business telemetry. |
| Local loopback API / sync agent status | `src/main.js`, `src/sync-agent-manager.js:417`; local API endpoint `127.0.0.1` | Required for desktop components that communicate with the bundled API | This is local IPC/network traffic, not an external vendor service; preserve port allocation, origin/CSRF checks and lifecycle supervision. |

The dependency list above separates runtime callers from URLs appearing only in documentation/help text. Search results alone do not prove a service is active. The AI/SMS/WhatsApp/storage providers should be considered user-configurable optional integrations, not unconditional application dependencies.

## 4. Source code reuse and replacement map

“Potentially reusable” means technically useful only; it is **not** a legal clearance.

| Component | Class | Evidence, dependencies and next action |
|---|---|---|
| Electron main process and startup supervision | B — modify | `src/main.js`, `src/server.js`, `src/mongodb-manager.js`, `src/api-runtime.js`; starts MongoDB/API, windows, updates, profiles, activation. Retain the proven local lifecycle after legal review; replace service URLs, identity, per-tenant cloud assumptions and release signing. |
| Preload and IPC | A/B — potentially reusable, requires review | `src/preload.js`, `src/hardware-ipc.js`, `src/ipc-guard.js`, tests such as `tests/preload-bridge.test.js`; typed capability bridge supports printer, hardware, cloud, diagnostics and shop functions. Preserve context isolation and per-handler validation; audit all handlers and renderer origins before reuse. |
| Express API | B — modify | `api/server.js`, `api/app.js`, `api/src/routes/`; extensive API and compatibility surface. One URI/process is the normal start path. Tenant selection, platform-vs-business APIs, sessions, error policy, service configuration and deployment need redesign. |
| Static frontend and build pipeline | A/B — potentially reusable | `frontend/static/`, `frontend/gulpfile.js`, `frontend/package.json`; legacy HTML/CSS/JS and Gulp/Sass/Terser asset generation. It is not a separate modern SPA framework. Verify dependencies, attribution, route/search catalogue and every Posnic asset/string. |
| Authentication/user/branch authorization | B — modify | `api/src/middleware/auth.js`, `api/src/controllers/users.controller.js`, `api/src/models/user.model.js`, branch-context utilities. Useful shop-level auth and branch access; not platform identity, tenant memberships, owner management or control-plane auth. |
| Mongoose models, repositories, services | B — modify | `api/src/models/` has 38 model-directory files including base, plugins and nested plugins (37 model files plus plugin-related files); key files are listed in section 5. Legacy direct native collection access is pervasive; each query must be assigned a tenant DB and authorized scope. |
| Sales, stock, receiving, customers and reports | A/B — potentially reusable after legal and behavior review | API service/repository/controller/model layers plus hundreds of focused tests. Reuse requires tenant scoping, transactional semantics, event/idempotency guarantees, data migration, financial/tax localization and regression parity. |
| Receipt and hardware printing | A/B — potentially reusable | `src/escpos-*`, `src/hardware-ipc.js`, `src/raw-print-service.js`, Windows printer modules; Windows/ESC-POS and Unicode paths exist. `src/escpos-unicode.js` rasterizes non-ASCII text with sandboxed Chromium and bundled fonts. Preserve the printer stack where tests pass; rebrand temporary namespace only with tests. |
| Backup/restore | B — modify | `src/backup-manager.js` writes local EJSON backups, manifests and hashes; `tests/backup-*` covers parts of the flow. It is not evidence of an operational cloud backup service. Add tenant ownership, encryption/key management, off-site retention and restore drills. |
| Sync | C/D — replace or investigate separately | `api/src/sync/collections.json` classifies data, `api/src/sync/outbox.js` records prioritized local sync intents, and `src/sync-agent-manager.js` launches an external agent. The Gateway/agent source is absent and described as separately delivered. Secure rights and service compatibility first; do not embed private source into customer/public builds. |
| Cloud activation, licensing, updates, support | C — replace for independence | `src/main.js`, `src/cloud-activation.js`, `src/update-service.js`, `src/asset-updater.js` directly couple flows to Posnic services/signing identity. Must be BiasharaOne-owned or disabled with a planned UX; simply replacing a hostname is insufficient. |
| Packaging and installation | B/D — modify and investigate rights | `package.json`, `scripts/`, `builds/`, MongoDB bundle and setup scripts. Electron-builder supports Windows NSIS/portable and also macOS/Linux. Inspect actual installer contents and third-party rights; use new publisher/certificate/update keys and public-build boundaries. |
| Tests | A/B — preserve and extend | Root tests and API Jest tests cover sales, stock, branches, printing, backup, offline recovery, sync classification and many route/model behaviors. Preserve them and add tenant-isolation and SaaS lifecycle cases before changing those seams. |
| Posnic marks, author/legal files, separately licensed assets | D — legal investigation | `LICENSE`, `docs/GOVERNANCE.md`, `THIRD-PARTY-NOTICES.md`, package notices and artwork. Do not remove, claim ownership, or distribute without required notices/permissions. |
| Unrelated or advanced first-release workflows (AI, extension marketplace, private registry, restaurant Captain/mobile/kitchen, campaigns) | E — defer from MVP unless required | Broad code exists, but it increases service, security, sync, licensing and support surface. Preserve existing behavior during the first tenant boundary work; choose any product exclusion explicitly. |

## 5. Database transformation map

### Current database and model organization

`api/server.js:29-52` resolves `config.database.uri`, `MONGODB_URI`, then `mongodb://localhost:27017/PosnicPro`, and connects one Mongoose connection. `api/src/config/config.js:13` carries the same default. `api/server.js` applies migrations at startup (including versioned `api/src/db/migrations/` and separate legacy index migrations) against that one database. No replica-set URI or local replica-set provisioning was confirmed in the inspected startup/configuration.

The sync classification manifest accounts for **143 named collections**: 29 synced, 103 local-only and 11 undecided. In addition, `api/src` contains **133 distinct literal** `.collection("name")` access names from the static scan; some are aliases/legacy collections and some accesses are dynamic, so do not equate that count with the entire physical database inventory. Collection names and rows may also be created by legacy/direct code outside Mongoose schemas.

### Existing model catalog

The API model directory contains 38 `*.model.js` files, plus `plugins.js` and three nested plugin files. It includes:

`activity-log.model.js`, `audit.model.js`, `base.model.js`, `branch.model.js`, `campaign-send.model.js`, `campaign.model.js`, `cashback-settings.model.js`, `category.model.js`, `coupon-redemption.model.js`, `coupon.model.js`, `credit-settings.model.js`, `customer-category.model.js`, `customer.model.js`, `dashboard.model.js`, `easy-table.model.js`, `expense.model.js`, `install.model.js`, `inventory.model.js`, `item.model.js`, `loyalty-config.model.js`, `loyalty-ledger.model.js`, `messaging-settings.model.js`, `print-job.model.js`, `print-till.model.js`, `receiving-return.model.js`, `receiving.model.js`, `register.model.js`, `role.model.js`, `sale.model.js`, `setting.model.js`, `shift.model.js`, `stock-log.model.js`, `supplier-legacy.model.js`, `supplier.model.js`, `token.model.js`, `user.model.js`, `variant.model.js`, and `whatsapp-template.model.js`, plus `plugins.js` and nested model plugins.

Not every collection is represented by a Mongoose model. Many services/controllers use `db.collection(...)` directly; a static source scan found 133 distinct literal collection names. A model-only tenant conversion would therefore leave many reads/writes on the process-wide connection.

### Main business fields and relationships

| Entity / file | Important observed shape and relationships |
|---|---|
| `Sale` / `sale.model.js:265-390,625-747` | `sales` records hold branch, user, customer snapshots, dates, status, totals, payment information and broad legacy/business metadata. Sale item rows and payment structures are substantially embedded (e.g. subdocuments near lines 27 and 234), not a normalized separate `sale_items` collection. `branch_id` references `Branch`; creator/user fields reference `User`; item subdocuments may refer to `Item`. `license` is a legacy scope field. |
| `Item` / `item.model.js:6-65,205-230` | `items`: name, barcode/SKU, category/supplier/branch IDs, quantity/available quantity, cost/company/selling prices, unit, tax, inventory flags and image; category/supplier/branch refs. Indexes include license+branch+price/quantity/update/name. |
| `InventoryItem` and `StockMovement` / `inventory.model.js:24-132` | Inventory is branch/item scoped; unique `(item, branch)` index on inventory items. Movement references item, branch and user. `stocklogs` and repository/service paths also maintain legacy stock journals. |
| `Branch` / `branch.model.js` | Branch/business profile and settings, license scope, and defaults for supplier/customer/users. Branch is not a platform tenant abstraction by itself. |
| `User` / `user.model.js:36-120` | Email/username/phone, password (select false), role/usertype, legacy `license`, branch/branch_id, branch_access array, access/role ID and account state. Email is sparse unique; phone, branch, role and status indexes appear at lines 457-460. User can access multiple branches within a shop but the schema does not establish membership in multiple independent businesses. |
| `Category` / `category.model.js:6-85` | Branch and license scoped, created/updated by user; text search and unique `(name, branch_id)` index. |
| `Supplier` / `supplier.model.js:7-80`; `Customer` / `customer.model.js` | Supplier model includes user audit refs and text search/unique key indexes; customer data is represented by a model plus direct collection access and sale customer snapshots. |
| `Receiving` / `receiving.model.js:24-147`; `ReceivingReturn` | Receiving/purchase receipt with embedded item lines, supplier and user references; receiving-number indexes are migrated on startup. Purchase orders are a separate direct collection and currently undecided for sync. |
| `Expense`, `Register`, `Shift`, `Setting`, `Role`, `ActivityLog`, `Audit`, `Token`, `PrintJob`, `PrintTill`, campaign/coupon/loyalty/messaging models | Shop/branch configuration, operational journals, tokens and optional modules. Platform records must not be mixed into these business records merely by naming a role `super_admin`. |

### Relationships, indexes and transaction boundaries

Common references are `Sale -> Branch/User/Item`, `Item -> Category/Supplier/Branch`, `InventoryItem/StockMovement -> Item/Branch/User`, `Receiving -> Supplier/Item/User`, `User -> Branch/Role/License`, and `Branch -> Supplier/Customer/User/License`. These are Mongo ObjectId references, embedded subdocuments, or legacy string identifiers; integrity is enforced in code rather than relational foreign keys.

Indexes are spread across schema declarations and runtime migration scripts. Examples: unique item+branch inventory in `inventory.model.js:132`; branch-scoped category uniqueness and text search in `category.model.js:82-85`; sale report sort indexes and unique partial `billing_transaction_id` per license in `sale.model.js:1125-1137`; user sparse-unique email plus phone/branch/role/status indexes in `user.model.js:457-460`; item compound branch/license indexes in `item.model.js:226-230`. Startup migrations include optional-email and receiving-number index reconciliation and versioned schema migrations. Audit all direct `createIndex` calls and existing customer data before choosing tenant DBs.

Transaction APIs exist, including `withTransaction`/session usage in `api/src/utils/db.js`, `api/src/models/base.model.js`, `api/src/models/inventory.model.js`, and `api/src/services/extension-library-invitations.js`. Atomic sale stock reservation and idempotent sale creation are visible in `api/src/services/sale.service.js:1525-1681` and `api/src/repositories/sale.repository.js:14575+`. MongoDB multi-document transactions require replica-set or sharded deployment; the bundled local MongoDB startup path does not establish a replica set in the reviewed configuration. Determine operation-by-operation whether standalone fallbacks are valid; do not assume all paths use transactions or that all local deployment modes support them.

The local backup manager dumps collections to a user-selected path, writes EJSON manifests/hashes and supports restore (`src/backup-manager.js:150-230` and later restore methods). That is local backup behavior, not proof of encrypted off-site backup, monitoring, scheduled cloud retention, or tested disaster recovery.

### Proposed database organization — recommendation, not implementation

Use a **control database** for platform businesses, business-owner identities/memberships, plans, subscriptions, licence entitlements, device registry, provisioning state, platform staff, support access grants/audit, platform payment ledger, backup jobs and service versions. Keep platform login/session/signing keys separate from business DB secrets.

Use a **separate database per business** for users, branches, items/categories/variants, sales and embedded sale items, customers, suppliers, receivings/purchase orders, expenses, inventory, stock movements/logs, registers, business settings and business audit. This matches the explicit Cloud design and reduces the chance of a forgotten `tenantId` filter. Keep a stable `businessId` in control-plane records and a provisioning state machine; do not infer identity from user-supplied DB names.

This design can use one Mongo cluster with separate database names initially, or separate clusters for stronger operational isolation later. Existing `tenant-connections.js` uses `Connection.useDb(..., {useCache:true})` on one pool and validates names, but it does not establish tenant resolution or authorization by itself. Keep DB selection behind a trusted control-plane mapping and a fail-closed request context. Local edition remains a single-business local DB with its own export/import and optional explicit pairing.

Existing mappings that can be adapted: business entities above. New/reworked control-plane objects have no confirmed equivalent. Existing `license` fields, shop `plan` properties, `install`, local `roles`, `branches`, or account `super_admin` must not be mistaken for SaaS business/subscription/license entities. `payment` is an undecided business collection and sale payment details also exist within `sales`; define their meaning before migration.

## 6. Tenant request-flow analysis

### Observed request flow and SaaS gap

1. `api/server.js:29-52` opens one database connection from the process URI. The reviewed normal startup path does not call `enableMultiTenant()` or resolve a database for a request.
2. Express middleware in `api/app.js` parses requests, mounts session/CSRF middleware around lines 969-1016, then mounts `attachDb` at lines 1099-1102. The database is attached before routers, but no trusted business/tenant resolution is visible before it.
3. `request-db.js:31-54` chooses `currentConnection(...)`, then `currentDb(...)`; it attaches whichever scope already exists. It does not derive or validate a tenant from a request.
4. `tenant-context.js:41-193` provides `AsyncLocalStorage`, `enableMultiTenant`, `runWithTenant`, and fail-closed DB/connection/secret selection. `tenant-connections.js:68` supplies cached `useDb`. `model-registry.js` supplies connection-aware model registration. These are scaffolding until invoked in the live request and background-job paths.
5. Authentication in `api/src/middleware/auth.js` verifies tokens and then `attachTenantContext` sets business license/branch context for the authenticated shop user. That is not equivalent to selecting an isolated business database before auth.
6. Example sale: `api/src/routes/sales.routes.js:476,511-560` applies `protect` to the protected route group, validates the POST body, then calls `SalesController.create`; controller `createOrHoldInternal` delegates to `sale.service`; service checks stock/idempotency and the repository writes the sale. Some routes deliberately appear before `router.use(protect)` for kiosk/device-key access, so tenant scoping must cover those as well.

The critical gap is not absence of all tenant-related code: it is the missing verified composition of tenant resolution, per-request DB context, tenant-specific auth/session keys, and exhaustive data-access binding in the production SaaS path.

### Cross-business access risks to resolve before serving multiple businesses

- Process-wide Mongoose models and `BaseModel.database` can retain a single connection. `tenant-context.js` itself warns that model imports/static handles and direct `mongoose.connection` access bypass per-request selection.
- The source documents 42 direct process-wide Mongoose access locations in `tenant-context.js`; the broader static scan finds 133 literal native collection names. These paths need classification and connection binding; converting only repositories/models is insufficient.
- `attachDb` does not choose a tenant. If a cloud deployment merely enables shared-process serving without opening tenant context before DB-dependent middleware/auth, code can read/write the default database or fail unpredictably.
- Session store, JWT issuer/secret, encryption secrets and API-key verification need a trusted tenant mapping and per-business semantics. A shared secret/incorrect session namespace undermines authentication isolation even if business records are partitioned.
- Legacy `license`, `branch_id`, request-body IDs, route IDs, Kiosk/API keys, mobile/handset grants, and background jobs are all possible scope inputs. Their authorization must be checked against the selected database and current business, not trusted as a tenant selector.
- Direct DB collection accesses, global counters, audit/recycle-bin operations, exports, search, uploads/files and reports can bypass route-level branch checks. Background workers and scheduled jobs need explicit tenant scopes too.
- User branch membership currently represents branch-level access within a shop; it does not establish cross-business membership, account uniqueness rules per tenant, owner invitations, or platform staff separation.

### Recommended middleware order

1. Establish trusted request metadata, proxy/IP policy, request ID, body-size limits and rate limits.
2. Resolve a tenant only from a validated custom domain/subdomain or a server-issued, authenticated device mapping; do not accept arbitrary tenant DB names from headers/body/query.
3. Load tenant status, DB mapping, auth/encryption key references and feature entitlements from the control plane; reject suspended/unprovisioned businesses.
4. Open AsyncLocalStorage tenant scope **before any DB-backed session or authentication middleware**; attach the selected business connection and tenant context.
5. Load tenant-scoped session/cookie and CSRF state, then authenticate a user/device against that selected tenant.
6. Enforce business membership, branch access, role/permission, module feature gates and route-specific validation.
7. Execute controller/service/repository exclusively through tenant-bound model/DB helpers; apply tenant context to files, search, exports and audit.
8. Return response and close request-scoped resources; run scheduled/background work in an explicit tenant scope.

The control plane itself needs its own authentication/authorization boundary and must not be exposed as a business tenant DB.

## 7. Local-cloud synchronization map

### Existing mechanics and limitations

`api/src/sync/collections.json` contains 29 synced branch/global collections, 103 local-only collections and 11 undecided collections. It says the separate Gateway scope map is authoritative and that a build checks the mirror. `api/src/sync/outbox.js` records local `sync_outbox` priority markers for `sales` events and state collections `items`, `variants`, `inventoryitems`; it is an accelerator, not the source of truth, and is disabled in multi-tenant cloud mode. `src/sync-agent-manager.js` launches an external agent and expects the cloud pairing configuration. Source for transport/checkpoint application is not present here, so actual sync delivery, conflicts, tombstones, reconnection behavior, and file transfer are not verified.

### Collection-by-collection decision

The recommendation below is for BiasharaOne's planned architecture, not a claim that the current product already implements it. “Keep local/authority only” means never replicate the executable credential, lock, lease, or command journal through generic collection sync.

**Existing synced (29):**

| Collections | Recommended BiasharaOne behavior |
|---|---|
| `sales`, `stocklogs`, `spoilage_entries`, `cashregister`, `receivings`, `expenses`, `waitercalls`, `tableorder`, `price_history`, `staff_activities`, `activitylogs` | Business DB authority with durable event-oriented replication. Till-originated committed sale/stock/payment effects flow to cloud; other authorized tills receive read projections/updates. Never replay a payment/stock command; retain idempotency keys, stable IDs, event sequence and reversal/return events. |
| `items`, `variants`, `categories`, `customers`, `suppliers`, `inventoryitems`, `easytables`, `branch_channels`, `billing_outlets`, `price_lists`, `modifier_groups`, `recycle_bin` | Tenant/branch-scoped business data. Explicit authority and conflict rules per object; for stock, sync ledger events and reconciled balance, not blind last-writer-wins. |
| `payment_method`, `grouptax`, `unit`, `settings`, `branches`, `users` | Business configuration/identity with cloud/business authority policy. Secrets and session credentials must not be copied as ordinary settings. `branches` and `users` are marked global in the existing manifest, but that scope is not a platform-global control plane. |

**Existing local-only (103):** keep these local to the issuing authority or move them to dedicated authenticated control/service APIs; do not add them to generic sync by default. Some rows below are operator-visible business state, but their delivery/lease/credential semantics still require a purpose-built protocol.

| Local-only family | Collections and recommendation |
|---|---|
| Command execution, device grants, locks, request receipts | `spoilage_commands`, `business_device_grants`, `business_decision_local`, `business_decisions`, `business_authorizations`, `extension_command_receipts`, `extension_stock_commands`, `extension_namespaces`, `extension_hardware_claims`, `inventory_operations`, `mobile_sales`, `mobile_grants`, `mobile_pair_codes`, `mobile_catalogue_pages`, `mobile_cloud_receipts`, `mobile_cloud_capabilities`, `captain_pair_codes`, `captain_sessions`, `handsets`, `kitchen_device_codes`, `kitchen_devices`, `table_seating`, `table_seating_history`, `takeaway_numbers`, `dojo_payment_operations`, `extension_payments`, `captain_payment_plans` — remain at the issuer/executor; use explicit, signed/idempotent APIs for outcomes. Never replay payments, stock actions, device grants or print claims on another installation. |
| Sessions, secrets, identity and registry | `sso`, `login_check`, `shadow_login_tokens`, `user_sessions`, `registry_accounts`, `registry_rate_limits`, `registry_sessions`, `business_sessions`, `business_push_devices`, `business_push_deliveries`, `captain_email_verifications`, `captain_phone_verifications`, `print_tills` — authority-local/control-plane only; no generic till replication. |
| Leases, jobs, counters and execution receipts | `counters`, `device_meta`, `data_change_log`, `price_sheet_jobs`, `price_sheet_rows`, `bulk_price_updates`, `bulk_stock_updates`, `business_reporting_requests`, `business_reporting_candidates`, `business_reporting_audit`, `business_reporting_local`, `business_reporting_publishers`, `business_prepared_summaries`, `business_stock_alert_batches`, `business_stock_alert_local`, `business_stock_snapshot_pages`, `business_stock_recipient_state`, `paper_order_usage`, `printjobs`, `campaign_sends`, `reminder_sends`, `whatsapp_outbox` — local/issuing service only; expose status/projections through authenticated service reads if cross-installation visibility is needed. |
| Private preferences, messages, schedules and account content | `business_notification_preferences`, `business_approval_notification_preferences`, `business_inbox`, `business_stock_notification_preferences`, `ask_posnic_preferences`, `ask_posnic_documents`, `ask_posnic_conversations`, `ask_posnic_action_drafts`, `ask_posnic_audit`, `ask_posnic_feedback`, `ask_posnic_schedules`, `supplier_message_drafts`, `messaging_events`, `messaging_settings`, `campaigns`, `credit_settings`, `cashback_settings`, `cashback_issues`, `roles`, `shifts` — tenant-specific authority or user privacy; executable schedules/provider secrets must not be cloned to independent offline APIs. |
| AI/provider billing and metering | `ai_usage`, `ai_voice_sessions`, `ask_posnic_local_vectors`, `ask_posnic_embedding_budget`, `ask_posnic_metrics`, `managed_ai_credits`, `managed_ai_reservations` — remain with the provider-key/billing authority; aggregate only through an authenticated, privacy-bounded reporting API. |
| Marketplace and installation-scoped assets | `extension_installations`, `extension_lifecycle_events`, `library_download_tickets`, `library_entitlements`, `library_invitations`, `library_memberships`, `library_releases`, `app_brand`, `paper_order_photos`, `captain_voice_archive`, `kitchen_voice_messages` — control/issuer-only, or transfer referenced files through an authorized storage service with per-tenant authorization and deletion rules. `branch` is a legacy alias, not a second business collection. |
| Other operational/config records | `inventory_counts`, `business_stock_alert_batches`, `business_reporting_*`, `business_stock_*`, `credit_settings`, `cashback_settings`, `campaigns`, `campaign_sends`, `spoilage_commands`, `razorpay_webhook`, `branch` — preserve issuer/authority semantics; add only after an explicit consistency, retention and authorization contract exists. |

The local-only family table groups related records; it intentionally retains exact collection names, but does not claim each name has been runtime-exercised. A few names appear in more than one explanatory family where they have multiple roles; the manifest's authoritative list is `api/src/sync/collections.json:44-148`.

**Undecided (11):**

| Collection | Recommendation |
|---|---|
| `quotes` | Sync customer/business quote records with stable IDs and branch scope; define draft conflicts and conversion-to-sale linkage. |
| `coupons` | Tenant-authoritative config/catalogue; offline redemption needs bounded signed eligibility and expiry policy, not blind sync of issuance commands. |
| `coupon_redemptions` | Append-only redemption events to the business authority with idempotency; detect duplicate cross-till redemption. |
| `loyalty_ledger` | Append-only earning/reversal events; preserve event order/unique operation IDs; do not overwrite balances. |
| `loyalty_config` | Business-authoritative versioned configuration; sync only after explicit effective-date and offline policy. |
| `customer_category` | Tenant/branch-scoped business master data with explicit merge/delete behavior. |
| `denomination` | Branch configuration with versioned authority; do not treat as payment records. |
| `branch_preferences` | Business-owned versioned config; secrets/provider values should use a separate credential path. |
| `transaction` | Clarify customer-credit ledger semantics; append-only debit/credit/reversal events, not duplicate-prone current-state sync. |
| `payment` | Clarify if this is a tender record versus payment command journal. Finalized shop-tender facts may sync as read-only/idempotent records; provider operations/webhook journals must remain with the executing authority. |
| `purchase_orders` | Sync orders with stable IDs and revision/conflict rules; receiving already syncs, so preserve order-to-receiving references and avoid replaying a receiving/stock effect. |

For sales and inventory, test till-to-cloud, cloud-to-till, second authorized till, offline/reconnect, duplicate retry, reversal/refund, branch isolation, tombstones, image/document references, and upgrade/backfill. Payment processing must not be re-executed by a receiving peer.

## 8. Licensing, subscription and central administration transformation map

### What exists

- Local install flow: `api/src/routes/install.routes.js`, `api/src/services/install.service.js`, desktop install key bridge in `src/preload.js`, plus per-install `POSNIC_KEY`/`POSNIC_SECRET` configuration. These guard installation/API setup; they are not a customer-facing perpetual product licence lifecycle.
- Shop account fields include `license` ObjectIds, internal plan/expiry properties and `super_admin`/role data (`api/src/models/user.model.js`, `install` code). The install model creates a shop user/branch relationship. These are not an independent platform licence or billing service.
- Cloud device activation/pairing validates and stores a device token and device ID (`src/main.js:1291-1398,1629-1706`, `src/cloud-activation.js`). This is tied to current Posnic Cloud/Gateway, not evidence of a BiasharaOne local perpetual licence.
- Updates are delivered by electron-updater/GitHub and gated by package/update signing identity. No independent BiasharaOne licensing/activation or SaaS subscription service was verified.
- Razorpay and other payment paths apply to shop/customer sales or extensions. They are not platform subscription billing.
- Governance documents say Posnic Cloud, sync, backups, remote dashboards and white-label builds live in separate closed repositories. Those private sources are not present for inspection.

### Capabilities to build separately

1. Control-plane business/owner enrollment and verification; owner membership and role separation from shop cashier/admin users.
2. Perpetual licence record and signed offline entitlement token for Local, with product edition, device limit, expiry/maintenance entitlement, issue/revoke/transfer audit, clock rollback policy, and a documented offline grace/renewal experience.
3. Device enrollment, revocation and transfer workflow that does not make normal local sales depend on internet availability.
4. SaaS plans, subscription schedule/status, invoices, grace/suspension/reactivation and platform payment reconciliation. Store these in control-plane records and keep customer tender payments in business DBs separate.
5. Tenant provisioning/deprovisioning with safe suspend/reactivate behavior, data export/retention, backups and account-owner handoff.
6. Update channels, signed manifests, key custody/rotation and support entitlement checks for both editions; keep Local's core operation usable when the licensing service is unavailable according to the explicit product policy.
7. Platform admin panel, access review/support impersonation controls, audit trails, abuse protection, platform metrics and support queue. No central business-registration/admin dashboard was found in the inspected POS application.

No replacement endpoints or legal rights are asserted here.

## 9. Build/release identity and deployment readiness

| Current setting | Affected files | BiasharaOne change and compatibility concern |
|---|---|---|
| Package/version/product name: `posnic`, `1.9.2`, `Posnic` | `package.json:1-3,70-72`, `api/package.json`, `frontend/package.json` | New package and app names; coordinate auto-updater, install path, existing userData migration, service logs and support scripts. |
| Electron app ID `com.posnic.app`; desktop name `com.posnic.app.desktop` | `package.json:6,70` | New reverse-domain ID; changes Windows app identity and may create a second app installation/data directory. |
| URL protocol `posnic://` | `package.json:72-78`, `src/main.js:107-125` | New URI scheme; publish migration/deep-link behavior, avoid hijacking old links. |
| Executable/installer artifacts from `productName` and electron-builder | `package.json:497-618,656-668` | New branded installer filenames/shortcuts; test upgrades and side-by-side installation. |
| Windows publisher `BillMax`, SHA-256 signing, signature verification | `package.json:497-518` | Obtain and control BiasharaOne signing certificate; do not publish under an unrelated publisher. Existing signed-update trust must be handled deliberately. |
| GitHub owner/repo `Posnic/POS`, release type | `package.json:544-550`, repository metadata near `715-720` | Own release channel and repository, independent artifacts, update keys and rollback process. |
| `posnic-updater` cache and update config under userData | `dev-app-update.yml:6`, `src/update-service.js:23` | New cache namespace; preserve/read old config during migration if old updates/settings matter. |
| Protocol branding, product icon, installer license artwork | `builds/app.ico`, `builds/512-posnic.png`, `builds/installer*`, `builds/installer-license.txt`, `package.json:551-618` | New art and notices; inspect actual packaged resources. Do not remove upstream legal notices or third-party licenses. |
| Linux app metadata, maintainer and description | `package.json:614-670`, `builds/linux/com.posnic.app.metainfo.xml` | New identity/support and appstream metadata; review package signing/distribution terms. |
| Database default `PosnicPro` | `api/server.js:29-30`, `api/src/config/config.js:13`, `.env.example:58`, `src/backup-manager.js:28-29` | Requires an explicit DB migration and import/restore compatibility. |
| Backup prefix `posnic-backup-*` | `src/backup-manager.js:15,164,677,817,973` | Keep legacy restore recognition while introducing a new prefix. |
| API auth issuer/audience/cookie names | `api/src/config/auth.js:20-21,89` | Rotate names in a planned session/token migration; never treat changing names as tenant isolation. |
| Support and account URLs | `src/main.js:1629-1737,2941-3100`, `api/.env.example` | Replace with tested BiasharaOne services or disable with clear UI states; old support/account flows otherwise fail. |
| Sync agent path and local activation config | `src/sync-agent-manager.js:4-11,73-118` | Maintain separately licensed/authorized delivery and installer boundary. Do not point public builds to a private checkout or copy private Gateway/agent files into customer artifacts. |

### Local and browser deployment

**Local:** Confirmed desktop packaging supports Windows NSIS/portable targets and bundles `mongod.exe` plus runtime dependencies when prepared; `src/main.js` supervises local API/DB processes. The product includes barcode/hardware/printer access and can operate locally without continuous internet for local workflows. This does not imply every feature works offline: cloud pairing, update checks, remote sync, configured messaging/providers and remote services need connectivity. The user must verify local backup/restore and printer behavior on representative hardware before sale.

**Browser:** The Express API serves the static frontend and has CORS/session configuration; this is technically a web-accessible application. A browser deployment can host the frontend/API centrally, but the normal startup uses one database and the code has Electron-specific and local-hardware paths. Central hosting, multi-business isolation, web hardware support, tenancy, operational monitoring and data recovery require additional work. Do not call it production-ready SaaS based only on the ability to open an API port.

## 10. Preservation, regression strategy and prioritized sequence

### Existing protection to preserve

Root tests include focused sales, payment, stock, purchases, customers, branch access, offline retry/recovery, backup integrity/config/path, sync classification, receipt/printing and Windows printing coverage. Examples: `tests/desktop-sale-network-retry.test.js`, `tests/external-sale-idempotency.test.js`, `tests/stock-ledger-precision.test.js`, `tests/purchase-history-filters.test.js`, `tests/branch-data-sharing.test.js`, `tests/offline-recovery-setup.test.js`, `tests/backup-integrity.test.js`, `tests/sync-classification.test.js`, `tests/escpos-unicode.test.js`, `tests/windows-print-queue.test.js`, `tests/windows-printer-discovery.test.js`. API tests include service/repository/controller/model/middleware/route suites such as `api/tests/unit/services/sale.service.test.js`, `sale-stock-transitions.test.js`, `purchase-order.repository.test.js`, `user.service.test.js`, `branch-scope.test.js`, `api/tests/unit/models/user.model.test.js`, and sales/users route tests.

The Unicode thermal receipt test suite verifies bundled-font behavior, Arabic/Tamil text, HTML escaping, raster-frame freshness, timeout failures, raw bytes/cut/drawer semantics and packaging inclusion (`tests/escpos-unicode.test.js`). This code is useful to retain subject to legal review; it must not be “rebranded” by deleting safety checks or test contracts.

No test suite was run for this read-only report. File names establish test presence, not pass status or real-device coverage. The checkout already has local tests for sync classification and offline behavior but no evidence here of a full multi-tenant isolation test against concurrent live tenant requests, live Gateway/cloud delivery, multi-tenant DB migration, or a full backup/restore disaster exercise.

### Recommended test additions, ordered before the affected implementation ships

1. **Tenant isolation boundary:** concurrent requests for two tenants across auth, session, models, direct `db.collection`, search, reports, exports, uploads/files, device-key/kiosk routes, and error paths; prove missing context fails closed.
2. **Control-plane lifecycle:** registration/provisioning idempotency, suspended tenant rejection, owner/user membership, branch/device authorization and tenant deletion/export boundaries.
3. **Financial transaction and replay:** duplicate sale requests, payment status/webhooks, refunds/reversals, stock reservation/return/receiving, outbox restart/retry, and tenant/branch isolation.
4. **Sync contract:** the directions and reconnection scenarios described in section 7, with Gateway and the intended installed agent available together; test tombstones, conflict ordering, duplicate IDs and referenced files.
5. **Migration/restore:** legacy single-shop database into a new business DB, repeatable migration, index reconciliation, rollback, restore and backup privacy/encryption.
6. **Licence/update lifecycle:** offline Local activation/grace/transfer, clock manipulation policy, device revocation, SaaS renew/suspend/reactivate, signed update key rotation and rollback.
7. **UI/hardware continuity:** run existing sale, receipt, printer, barcode, backup and browser flows on Windows and representative printer/scanner hardware; test new branding/search navigation after any route or setting change.

### Suggested implementation sequence (recommendation)

1. Resolve legal clearance, source/third-party notices, trademark, contributor-rights and separately licensed Gateway/agent boundaries. Do not start by stripping branding.
2. Document and freeze current data schema, import/export formats, business workflows, POS contracts and regression evidence. Build a safe sample-data migration pipeline.
3. Design control-plane entities and business-database provisioning; define tenant IDs, database ownership, data residency/retention and operator access policy.
4. Complete a source audit for all model/direct native DB paths; convert the request path to trusted tenant resolution before auth/session, bind all queries and workers, and implement fail-closed concurrency tests.
5. Migrate one business database and validate financial/stock invariants and restore before exposing SaaS traffic.
6. Define sync protocol and collection-specific policies with the separately delivered Gateway/agent owners; test both directions and the actual installer/agent delivery boundary.
7. Implement separate platform admin/auth, product licensing and SaaS billing ledgers; keep sales tenders separate.
8. Establish independent app identity, publisher/signing, release/update channels, branded assets, user-data migration and support endpoints.
9. Run regression, tenant isolation, Windows printer/hardware, backup restore, dependency/licence and package-content checks before any pilot distribution.
10. Pilot with isolated tenant data and monitoring; do not claim readiness until end-to-end sync, backups, recovery, account lifecycle and support processes are verified.

## Important file index

| File | Purpose |
|---|---|
| `package.json` | Electron app identity, build/package configuration, root dependencies/scripts and release metadata |
| `api/package.json` | Express API dependencies, test/lint scripts and service metadata |
| `frontend/package.json` | Static frontend build toolchain and frontend metadata |
| `src/main.js`, `src/preload.js`, `src/hardware-ipc.js` | Electron startup, renderer bridge and hardware/printer IPC |
| `src/server.js`, `src/api-runtime.js`, `src/mongodb-manager.js` | Local API/runtime/database lifecycle |
| `api/server.js`, `api/app.js` | API startup, single-URI connection and middleware/router composition |
| `api/src/db/tenant-context.js`, `tenant-connections.js`, `model-registry.js`, `request-db.js` | Tenant-aware database scaffolding; not proof of active SaaS resolution |
| `api/src/middleware/auth.js`, `api/src/utils/tenant-context.js` | Authentication and shop/branch context |
| `api/src/models/`, `api/src/repositories/`, `api/src/services/` | Mongo models, direct data access and business logic |
| `api/src/routes/sales.routes.js`, `api/src/controllers/sales.controller.js`, `api/src/services/sale.service.js`, `api/src/repositories/sale.repository.js` | Representative sale write path and idempotency/stock logic |
| `api/src/sync/collections.json`, `api/src/sync/outbox.js`, `src/sync-agent-manager.js` | Sync classifications, local outbox and external agent launcher |
| `src/backup-manager.js` | Local database backup/restore implementation |
| `src/cloud-activation.js`, `src/update-service.js`, `src/asset-updater.js` | Cloud activation validation, desktop update and asset update identity |
| `src/escpos-unicode.js`, `tests/escpos-unicode.test.js` | Unicode thermal receipt rasterization and focused regression tests |
| `LICENSE`, `docs/GOVERNANCE.md`, `THIRD-PARTY-NOTICES.md` | Project license, governance/trademark/contributor terms and bundled third-party notices |
| `api/.env.example`, `dev-app-update.yml`, `builds/` | Integration configuration template, updater identity and installer resources |

**Verification record:** source and configuration inspection only. No tests, builds, dependency installation, database migrations, live service calls, or package-content inspection were performed.
