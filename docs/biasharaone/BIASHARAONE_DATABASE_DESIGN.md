# BiasharaOne Database Design

## 1. Design ownership

This is a new logical design. It does not copy existing model definitions or migrate directly to the existing names.

Use:

- A control database for platform records.
- A separate business database per Cloud tenant.
- One business database for a Local installation.

Stable UUID or ULID identifiers are preferred for business events and cross-database references.

## 2. Control database entities

### Business

- id
- registration name
- slug/domain
- status
- owner member
- data region
- provision state
- database reference
- created/updated timestamps

### Owner and membership

- id
- business id
- platform identity id
- role
- status
- invited/accepted timestamps

### Plan and subscription

- id
- business id
- plan id
- status
- period
- grace deadline
- provider customer/subscription references
- audit timestamps

Subscription records are not customer tender or sale payment records.

### Licence entitlement

- id
- business or account owner
- edition
- signed entitlement reference
- devices allowed
- maintenance expiry
- issue/revoke timestamps
- reason

### Device

- id
- business id
- stable device identifier
- public key or attestation reference
- status
- last seen
- branch assignment

### Provisioning job

- id
- business id
- target
- state
- idempotency key
- attempts
- error state
- timestamps

### Platform support grant

- id
- business id
- grantor
- grantee
- scope
- expires at
- reason
- revoked at

### Backup job

- id
- business id
- type
- status
- encrypted storage reference
- checksum
- started/completed timestamps
- restore eligibility

### Platform audit

- id
- actor
- action
- business scope
- target
- outcome
- timestamp
- trace ID

## 3. Business database entities

### Branch

id, tenant scope, name, code, address, timezone, settings, status.

### Employee

id, identity, name, status, role memberships, branch memberships.

### Role and permission

Role and permission IDs, action keys, branch scope rules. No platform role belongs in this table.

### Category

id, branch scope, name, parent, status.

### Item

id, name, SKU, primary barcode, category, unit, tax, cost/price policy, status, media references.

### Variant

id, item id, option values, SKU, barcode, price/cost policy, status.

### Inventory item

id, item/variant id, branch id, on-hand, available/reserved state, as-of checkpoint.

### Stock movement

id, branch, item/variant, quantity delta, type, reason, source document, actor, idempotency key, timestamp.

Stock balances may be projections of movements, never the only payment/stock authority.

### Sale

id, submission key, branch, cashier, customer, status, subtotal, discount, tax, total, timestamps, fiscal reference.

### Sale item

id, sale id, item/variant, quantity, unit price, discount, tax, totals.

### Payment

id, sale or customer-account reference, tender, amount, provider reference, status, reversal relationship.

### Customer

id, identity details, credit policy, consent, status.

### Customer account movement

id, customer, sale/payment/adjustment reference, debit/credit amount, balance impact, timestamp.

### Supplier

id, identity details, terms, status.

### Purchase order

id, supplier, branch, status, totals, version.

### Receiving

id, supplier, purchase order, branch, document number, costs, status, submission key.

### Receiving line

id, receiving id, item/variant, quantity, cost, tax.

### Expense

id, branch, category, amount, tax, supplier, status, approval, attachments.

### Register and shift

Register, opening float, close declaration, cash movements, variance and audit.

### Setting

tenant/branch/module scoped key, typed value, version, updated by, secret reference when applicable.

### Audit event

id, actor, action, entity, entity id, branch, before/after hash or summary, trace ID, timestamp.

### Sync event, checkpoint and tombstone

Only if the deployment enables sync. Include stable ID, event type, tenant/branch, revision, body/hash, status, checkpoint, conflict and acknowledgement.

### Sync outbox

id, event id, priority, state, attempts, next attempt, acknowledgement. Never repeat an already committed payment/stock operation.

## 4. Constraints

- Every business row carries tenant scope enforced by its database connection and application context.
- Branch fields shall reject access outside an actor's allowed branches.
- Unique SKU/barcode rules shall be defined per business and branch policy.
- Stock movement quantity, sale monetary values and credit values use consistent decimal/major-minor units.
- Every external submission uses an idempotency key.
- Audit events are append-only from the application's business perspective.
- Soft deletion shall use status/tombstone fields and not erase referenced history.

## 5. Indexes

At minimum:

- branch + business time for sales
- sale submission key unique
- sale status and branch
- item SKU/barcode
- item/category/variant
- unique inventory item/variant/branch
- stock movement item/variant/branch/time/source
- customer identity and account reference
- supplier identity
- purchase/receiving supplier, branch and document number
- register/shift branch and open/close time
- audit actor/action/entity/time
- sync event ID and checkpoint
- outbox state/next attempt

Final indexes require real query and cardinality analysis.

## 6. Migration and ownership rules

Before migration, inventory every direct collection access and every native model. Preserve old backups and old identifiers during backfills. Provision new databases only through control-plane state machines. Remove or invalidate secrets during deprovisioning and export.
