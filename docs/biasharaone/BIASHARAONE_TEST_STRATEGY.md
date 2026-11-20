# BiasharaOne Test Strategy

## 1. Test layers

- Static checks and type/lint checks
- Unit tests
- Repository/data integration tests
- API contract tests
- Tenant-isolation and security tests
- Browser/desktop acceptance tests
- Hardware simulation and Windows printer tests
- Migration/restore tests
- Sync/reconciliation tests
- Disaster recovery and backup drills
- Payment/fiscal sandbox tests

## 2. Functional tests

### POS

- fast sale
- holds and recalls
- partial/multiple tenders
- discounts
- returns/refunds
- duplicate submission
- offline sale
- receipt reprint

### Inventory

- sale stock effect
- adjustment
- receiving
- transfer
- low stock
- stock reversal
- duplicate event

### Purchase and accounts

- purchase order
- receiving
- supplier return
- customer credit
- payment allocation
- statement/report totals

### Branch and users

- branch membership
- role permissions
- inactive user
- privilege denial
- cross-branch isolation

## 3. Tenant isolation

Run concurrent requests for at least two tenants. Test:

- auth/session
- sales
- products
- inventory
- reports
- exports
- uploads/files
- search
- device/kiosk routes
- webhooks
- direct database helpers
- background jobs

Missing tenant context shall fail closed.

## 4. Financial invariants

Use property and focused tests for:

- subtotal + tax + discounts = total
- sale return cannot exceed eligible quantity/value
- stock movement equals net source and destination
- duplicate submission is idempotent
- receiving and receiving return net correctly
- customer debit/credit balance reconciliation
- no replay of payment or stock command

## 5. KES and tax

Test major/cents conversion, inclusive/exclusive tax, rounding, zero tax, mixed rates, discounts, cancellation and credit notes. Use fixtures approved by the implementation accountant.

## 6. Sync tests

Test:

- till to cloud
- cloud to authorized till
- second authorized till
- offline/reconnect
- duplicate retry
- conflict
- tombstone
- branch isolation
- referenced files
- migration/backfill
- outbox restart
- payment/stock command non-replay

Live Gateway/agent tests shall use the same separately delivered components and installer manifest as production.

## 7. Licensing and updates

- offline activation/grace
- clock rollback policy
- expiration
- revoke/transfer
- subscription renew/suspend/reactivate
- signed update success
- key rotation
- update rollback
- failed update with local operation retained

## 8. Backup and restore

Create, verify and restore each supported backup format. Test tamper detection, wrong key, partial package, interrupted restore and legacy compatibility. Independently verify RPO/RTO and critical reports.

## 9. Hardware

Windows tests shall cover:

- printer discovery
- queue recovery
- ESC/POS and Unicode receipt rendering
- cash drawer
- scanner
- label/barcode
- device unavailable
- repeat/reprint without sale replay

No conclusion shall claim broader real-hardware success than the tested devices.

## 10. Security tests

- CSRF/session/cookie
- privilege escalation
- SQL/NoSQL injection boundary validation
- upload MIME/size/path
- secret redaction
- rate limiting
- webhook replay
- IPC origin/window
- cross-tenant data and files
- support grant expiry

## 11. Kenya integration tests

Use official sandboxes for eTIMS and M-Pesa. Verify:

- callback signatures
- unknown or delayed status
- replay
- mismatch
- refund rules
- receipt fields
- offline queue

Do not make a live or regulatory claim before formal verification.

## 12. Test evidence

Every release shall record command, environment, commit, test result, unresolved defects and hardware/provider limitations.
