# BiasharaOne System Architecture

## 1. Architectural principles

- Independent source and product identity
- Local-first core operations
- Separate control and business data
- Separate database per Cloud business
- Fail-closed tenant resolution
- Idempotent financial and stock events
- Explicit synchronisation authority
- Secure, signed updates
- Least-privilege administration
- Verifiable migration and recovery

## 2. Deployment models

### 2.1 BiasharaOne Local

The desktop shell starts a protected local business API and the selected local database runtime. A browser renderer communicates only through an explicit capability/HTTP boundary.

Core features operate from the local data store. Internet dependency is limited to optional activation, sync, updates, external payments and external fiscal/messaging services.

### 2.2 BiasharaOne Cloud

A load-balanced edge routes browser traffic to the business API. A trusted tenant resolver loads the business database mapping from the control plane. The API establishes tenant context before session, authentication or database access.

The Cloud deployment uses one database per independent business. Separate databases may share a cluster initially and move to stronger isolation later.

### 2.3 Control plane

A separate control-plane API and admin application manage:

- Business registration
- Tenant provisioning
- Licence entitlements
- Subscriptions and billing
- Device activation
- Deployment/version records
- Backup jobs and restore approval
- Support access grants
- Platform audit
- Aggregated authorised analytics

The control plane shall not become an implicit business database or expose customer records.

## 3. Components

```text
Local/Desktop:
Electron shell -> controlled renderer -> local business API -> local business database
                                      -> optional sync agent

Cloud:
Browser -> business API -> tenant resolver -> tenant business database
                         -> optional control-plane entitlement reads

Admin:
Admin browser -> control-plane API -> control database
                                  -> provisioning/backup services

Sync:
Authorized till -> sync agent -> sync Gateway -> business database
                                -> other authorized till
```

## 4. Tenant resolution

Cloud tenant resolution shall use only a validated custom domain/subdomain or server-issued authenticated device identity. A browser or API client shall not choose a database name.

The middleware order is:

1. Request identity, proxy policy and limits
2. Tenant resolution
3. Tenant status and database mapping
4. Tenant AsyncLocalStorage scope
5. Tenant session and authentication
6. Membership, branch and permission checks
7. Domain operation
8. Audit and sync decision

Missing or conflicting tenant context shall fail closed.

## 5. Data architecture

Control database:

- Businesses
- Owner identities and memberships
- Plans and subscriptions
- Entitlements/licences
- Devices
- Provisioning state
- Support grants
- Platform audit
- Backup jobs
- Release/deployment metadata

Business database:

- Branches and users
- Catalogue and variants
- Inventory and stock movements
- Sales and sale items
- Payments and returns
- Customers and supplier accounts
- Purchasing and receiving
- Expenses
- Registers and shifts
- Business settings
- Business audit

## 6. Financial and stock operations

Sales, receiving, adjustment, transfer and refund operations shall use stable business IDs and idempotency keys. Where the database supports transactions, use them. If standalone Local MongoDB cannot provide the required transaction semantics, implement a reviewed durable journal and reconciliation state, and do not claim atomicity.

Reversal and return events shall be separate from the original committed event. Generic sync shall not replay a payment or stock command.

## 7. Synchronisation

Synchronisation shall use a versioned event contract with stable IDs, sequence/checkpoint, tenant/branch scope, acknowledgements and conflict policy. The authority of every collection and file shall be documented.

Credentials, sessions, payment operation journals, print queues, hardware claims and local execution leases shall not be copied through ordinary business-state synchronisation.

## 8. Authentication

BiasharaOne should use independent business identity providers and separate platform identities. A chosen implementation shall support secure password storage, MFA, session/device revocation, recovery and tenant-scoped cookies/tokens.

## 9. Technology recommendation

No implementation choice is final until technical evaluation. A practical baseline is:

- Electron for Local desktop
- Node.js/TypeScript for the shared business API
- React or another independently selected web framework for browser UI
- MongoDB only if tested for transaction and operational needs
- PostgreSQL for control-plane relational billing/entitlements if that better fits operations
- npm workspaces or a reviewed monorepo system
- Signed releases and software-bill-of-materials generation
- Automated unit, integration, security and acceptance testing

The existing stack may inform maintainability analysis but is not automatically the new architecture. The team must own and select every runtime dependency.

## 10. Deployment and operations

- Separate staging and production credentials
- Secret manager for provider and database secrets
- Encrypted backups
- Health/readiness metrics
- Per-tenant audit and support boundaries
- RPO/RTO targets
- Update channels and rollback
- Incident and vulnerability response
- Release-package content inspection

## 11. Open verification items

- KRA/eTIMS integration contract
- M-Pesa payment contract
- Local MongoDB transaction mode
- Cloud database isolation tier
- Sync transport ownership
- Third-party licences
- Platform billing provider
- Regulatory advice and data residency
