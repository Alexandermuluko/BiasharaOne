# BiasharaOne API Specification

## 1. Scope

This document defines independent API requirements, not a list of existing routes or implementation signatures.

The API uses versioned JSON, stable resource IDs, UTC timestamps, typed decimal/monetary values or integer minor units, structured errors and request IDs.

## 2. API boundaries

### Business API

Handles authentication and operations for one selected business:

- Auth/session
- POS/sales
- Products/catalogue
- Inventory
- Purchasing/receiving
- Customers/suppliers
- Expenses/registers
- Branches/users/roles
- Reports/exports
- Business settings
- Sync event acknowledgements

### Control-plane API

Handles platform records and is separate from business user authentication:

- Business registration
- Tenant provisioning
- Plan/subscription
- Licence entitlement
- Device activation
- Deployment/release records
- Backup and support operations
- Platform audit

### Integration API

Handles explicitly authorized providers and webhooks:

- M-Pesa
- Banks
- Fiscal/KRA adapters
- Messaging
- Sync Gateway

Provider secrets and customer-business mappings shall never be accepted from public client request fields.

## 3. Required cross-cutting behavior

All mutations accept a request ID and state-changing financial operations accept an idempotency key.

Errors use:

```json
{
  "code": "RESOURCE_CONFLICT",
  "message": "Safe user-facing explanation",
  "request_id": "server-generated-id",
  "field_errors": []
}
```

Sensitive values and stack traces shall not appear in client responses.

## 4. Authentication

- Cloud tenant resolution occurs before authentication.
- Business authentication uses tenant-scoped session or token state.
- Platform authentication is isolated from business credentials.
- Device-key routes receive explicit scopes and are not public fallbacks.
- Session rotation, logout, deactivation and device revocation shall invalidate access.
- Password recovery and privileged actions require authorization and audit.

## 5. Endpoint groups

| Group | Representative operations | Mandatory checks |
|---|---|---|
| Auth | login, logout, refresh, recovery, MFA | tenant scope, rate limit, secure session |
| Sales | create, hold, recall, list, detail, return, refund | branch, role, idempotency, stock/credit |
| Catalogue | items, variants, categories, units, barcodes | tenant/branch scope, uniqueness |
| Inventory | list, adjust, transfer, receiving, movement | branch, approval, movement invariant |
| Purchasing | supplier, PO, receiving, return | tenant/branch, approval, exactly-once effect |
| Parties | customer/supplier create/list/detail/payment | privacy, branch/role |
| Registers | open, cash movement, close | register/shift authorization |
| Expenses | create, approve, list, export | branch/role/document policy |
| Reports | sales, tax, inventory, purchase, expense, audit | authorized scope, bounded export |
| Settings | branch, tax, tender, receipt, feature settings | module permission, explicit opt-in |
| Users | invite, update, deactivate, branch/role | privilege escalation protection |
| Tenants | register, provision, suspend, reactivate | platform auth |
| Licences | issue, renew, revoke, transfer | platform auth/audit |
| Subscriptions | create, change, grace, suspend | billing provider reconciliation |
| Devices | pair, inspect, revoke | signed/verified device mapping |
| Backups | create, monitor, restore approval | encryption, tenant scope |
| Sync | acknowledgement, checkpoint, event status | tenant/device scope, idempotency |
| Webhooks | payment/fiscal/provider events | signature, replay protection, stable event ID |

## 6. Sale create contract

Required fields:

- stable sale/submission ID
- branch
- register/shift when applicable
- lines
- customer when applicable
- tender/payment details
- idempotency key
- optional fiscal request data

The API returns:

- committed sale ID
- status
- totals
- stock state
- payment status
- receipt/fiscal status
- server timestamp

The same idempotency key shall not apply a second stock or payment effect.

## 7. Pagination and filtering

List endpoints support:

- cursor or stable page token
- bounded size
- explicit sort
- authorized date/branch/entity filters
- match counts only with scoped aggregate queries

## 8. Files and exports

Uploads use MIME, size and malware-safety validation where applicable. File access is authorized from the business record and tenant context. Exports use a bounded asynchronous job if full export may exceed request limits.

## 9. Webhooks

Webhook verification shall use provider signatures and replay protection. A provider event ID shall map to one business outcome. A payment command shall not be replayed merely because sync retries a business record.

## 10. Offline contract

The business API in Local must serve loopback-only or explicitly authorized network clients according to configuration. Core operations shall not call external licence or sync services before committing a permitted sale.

## 11. OpenAPI and contract tests

The implementation shall publish a reviewed OpenAPI contract, generated contracts for clients and integration tests for:

- tenant isolation
- branch and role denial
- idempotency
- money rounding
- stock invariants
- refund/return
- sync acknowledgements
- webhook replay
- provider timeout
