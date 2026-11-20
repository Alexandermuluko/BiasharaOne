# BiasharaOne Independent Implementation Roadmap

## 1. Gate 0: Legal and ownership

Do not copy source or distribute existing components. Produce:

- Legal source reuse decision
- AGPL/third-party notice plan
- Contributor rights register
- Trademark/name clearance
- Third-party asset licence register
- Private sync/Gateway delivery boundary
- Billing, payment and fiscal review plan

**Exit:** counsel has approved a source/use strategy and the team has an independent repository and identity.

## 2. Gate 1: Independent requirements baseline

- Review and approve product requirements
- Freeze first-release scope
- Define Local and Cloud feature parity
- Approve acceptance tests
- Define KES/tax/receipt policy boundaries
- Define offline behaviour
- Define data export, retention and deletion

**Exit:** the product requirements and Kenya assumptions have named owners.

## 3. Gate 2: Architecture and data design

- Choose independent desktop/web/API/database technologies
- Design control and business database boundaries
- Define tenant resolution and authorization
- Define stable IDs and money/stock types
- Define sync and conflict policy
- Define key, backup and secret management

**Exit:** architecture and data review passes threat-model and recovery review.

## 4. Gate 3: Platform foundation

Build:

- Independent identity and session boundaries
- Control-plane business registration
- Tenant provisioning state machine
- Separate business database resolver
- Audit and observability
- Desktop shell and protected local API
- Independent app/update identity

**Exit:** no cross-tenant request can reach another business database.

## 5. Gate 4: Business core

Implement or independently rebuild:

- Catalogue, stock and variants
- Sale/return/payment
- Purchasing/receiving
- Customers/suppliers
- Expenses/register
- Branch/user/role
- Reports/export
- Local backup/restore
- Universal search and settings

**Exit:** financial, stock and authorization acceptance tests pass.

## 6. Gate 5: Kenya integrations

Implement behind feature gates:

- KES/timezone/date/currency formatting
- Configurable tax model
- M-Pesa adapter
- Bank/cash/terminal providers
- Kenya receipt template
- eTIMS adapter after official verification

**Exit:** sandboxes, legal review and reconciliation tests pass.

## 7. Gate 6: Cloud and subscriptions

Implement:

- Cloud tenant access
- Plans and subscription ledger
- Licence/device entitlement
- Platform admin
- Backup jobs
- Support grants
- Deployment/release management
- Aggregated authorized analytics

**Exit:** platform and business boundaries pass security review.

## 8. Gate 7: Synchronisation

Define and implement only after rights, architecture and data authority decisions:

- Sync Gateway contract
- Local agent delivery
- Event/checkpoint/tombstone model
- Branch conflict policy
- File transfer
- Offline queue/retry
- Reconciliation

**Exit:** live two-way tests pass across connected tills and reconnect conditions. No private source is embedded without authorization.

## 9. Gate 8: Release readiness

- Complete hardware matrix
- Complete restore drills
- Complete Windows signed packaging
- Inspect installer content
- Generate notices/SBOM if required
- Run security review
- Run attribution checks before PR/publish
- Document support and incident processes

**Exit:** pilot can be rolled back and support has evidence and scope.

## 10. Suggested delivery order

1. Legal clearance and clean independent repository
2. Tenant/control architecture and database foundation
3. Core POS stock/sale invariant
4. Local desktop package
5. Auth, branch, employees and reports
6. Kenya and optional payment/fiscal adapters
7. Cloud business database deployment
8. Licensing/subscriptions/control plane
9. Authorized sync
10. Production release and support

## 11. Explicit exclusions from the first independent build

The following remain optional until separately planned:

- AI assistant
- Restaurant features unless required
- Extension marketplace
- Advanced marketing
- Messaging campaigns
- Private sync/agent
- Any third-party closed service

Exclusion must be recorded in product scope; it must not become an undocumented runtime dependency.
