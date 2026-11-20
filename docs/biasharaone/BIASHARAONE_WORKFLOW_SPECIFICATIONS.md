# BiasharaOne Workflow Specifications

## Checkout

1. Cashier opens an authorized branch register.
2. Items are added by scan, search or manual entry.
3. Price, quantity, discount and tax are validated.
4. Optional customer and tender details are selected.
5. The system validates stock and credit rules.
6. The client sends a unique sale submission key.
7. The server commits one sale and the stock/payment effects using the supported database transaction capability.
8. Receipt and audit data are acknowledged.

**Offline behavior:** the local sale remains valid if the internet is unavailable. Cloud-only services are excluded from the core commit path.

**Failure behavior:** no sale total or stock state shall be applied partially. A retry with the same submission key shall return the original committed result.

## Sale return

1. An authorized role finds the original sale.
2. Eligible lines and quantities are selected.
3. Reason and approval are captured.
4. Refund/reversal is validated against prior returns and payment availability.
5. One reversal document and stock effect are committed.
6. The original sale remains visible as a linked history.

## Stock receiving

1. Inventory officer selects a supplier and branch.
2. They optionally link an approved purchase order.
3. Lines, quantities, costs and documents are entered.
4. Quantity and cost validation complete.
5. The receiving document is committed once.
6. Stock and supplier/purchase balances are updated.

## Stock transfer

1. Sender creates a transfer with stable identifiers.
2. Source stock is reserved or deducted according to policy.
3. Destination accepts or rejects received quantities.
4. Differences are recorded with audit and approval.
5. Transfer history remains reversible only through a documented return workflow.

## Customer credit sale

1. Authorized cashier selects a customer.
2. Credit limit and account status are checked.
3. Sale is committed to customer ledger.
4. Outstanding balance and statement update.
5. Over-limit or overdue policies apply according to settings.

## Register open and close

1. Cashier selects an authorized register.
2. Opening float is recorded.
3. Tender, sale and approved cash movements are aggregated.
4. Cashier declares closing amounts.
5. Variance is identified and approval/reason rules apply.
6. Shift close writes final register state.

## Purchase and supplier return

1. Manager creates or approves purchase order.
2. Goods are received and costed.
3. Supplier return is linked to original receiving.
4. Stock and supplier balance update once.
5. Both documents remain auditable.

## Local backup and restore

1. User initiates a backup or scheduled backup.
2. System writes an integrity manifest and encrypted package.
3. Verification reports readable records and integrity status.
4. Restore requires separate explicit confirmation.
5. Restore preview identifies database version and migration requirements.
6. Restore completes only after integrity and compatibility checks.
7. Users verify key reports and a post-restore sale in a controlled check.

## Cloud tenant provisioning

1. Owner registration is validated.
2. Platform creates tenant control record and provisioning job.
3. A unique business database is provisioned.
4. Owner membership and initial branch/user are created.
5. Entitlements and initial settings are attached.
6. Provisioning emits one success or recoverable failure outcome.
7. Retry uses stable provisioning identifiers and never creates duplicate records/databases.

## Tenant sign-in

1. Resolve tenant through authenticated custom domain/subdomain or device mapping.
2. Load tenant status and database mapping from control plane.
3. Reject unknown, suspended or unprovisioned tenants.
4. Establish tenant context before session/authentication/data access.
5. Authenticate business user.
6. Check membership, branches and permissions.
7. Create a tenant-scoped session and audit event.

## Offline-to-cloud sync

1. Offline sale receives a stable business event ID.
2. Local outbox records the event durably.
3. Sync service authenticates and obtains tenant scope.
4. Events are validated and applied exactly once.
5. Server checkpoint is acknowledged.
6. Reconciliation response reaches the originating till.
7. Retries preserve the original event ID and checkpoint.

Payment commands, print jobs and credential grants are execution records, not generic replayable business state.

## KRA eTIMS submission

1. Tax configuration is reviewed and the fiscal adapter is enabled explicitly.
2. Sale is completed according to ordinary POS rules.
3. A fiscal payload is prepared only from verified official fields and rules.
4. Submission is retried according to official operational guidance.
5. Control number/status is stored against the original sale.
6. Failure stays visible and does not rewrite sale totals.

No eTIMS compliance claim is made until official sandbox and legal verification.

## Universal search

The navigation catalog shall include primary lists, detail/create flows and exact setting sections. It shall use translated labels and operator aliases. Add actions shall open existing forms and shall never submit them.

## Deletion and retention

Business records shall be archived or tombstoned according to policy. Permanent deletion shall require role approval, tenant confirmation and a data-retention decision. Audit events shall retain references without unnecessary personal data.
