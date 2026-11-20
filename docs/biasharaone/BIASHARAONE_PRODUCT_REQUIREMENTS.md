# BiasharaOne Product Requirements

## 1. Product vision

BiasharaOne is an independent commercial POS and business management platform for Kenyan retailers and service businesses. It must support fast in-store checkout, inventory, purchasing, customer accounts, expenses, reporting and multi-branch operations.

The product has two editions:

- **BiasharaOne Local:** a locally installed desktop application that performs core POS operations without internet access.
- **BiasharaOne Cloud:** a centrally hosted browser service for authorized businesses, with a separate business database per tenant.
- **Platform administration:** an independent control plane for businesses, subscriptions, licences, devices, provisioning, support and operations.

Core sales must remain usable during internet outages. Optional cloud, sync, messaging, payment and tax-service features must fail safely without affecting an already permitted local sale.

## 2. Users

- Business owner
- Branch manager
- Cashier
- Inventory officer
- Accountant or auditor
- Platform administrator
- Support technician
- System integrator

Each role receives only the permissions and branches assigned to it. Platform staff are not automatically business users.

## 3. Functional requirements

### FR-1 Point of sale

The system shall allow scan, search or manual item selection; quantity and price entry; discounts; tax; customer selection; multiple tenders; holds; recalls; returns; refunds; receipts and cash-drawer actions.

### FR-2 Sales management

The system shall record immutable sale identity, branch, cashier, timestamps, line items, totals, taxes, payments, status and audit history. It shall prevent duplicate submission and support approved reversals.

### FR-3 Catalogue

The system shall manage categories, items, variants, units, barcodes, SKUs, suppliers, selling prices, cost prices, images and item settings. Identifiers shall be unique within the applicable business scope.

### FR-4 Inventory

The system shall support stock balances, adjustments, receiving, spoilage, transfers, stock movements and low-stock alerts. A committed sale or receiving document shall produce the appropriate stock effect exactly once.

### FR-5 Purchasing

The system shall support supplier records, purchase orders, receiving, costs, supplier returns and purchase history.

### FR-6 Customers and credit

The system shall support customer profiles, balances, statements, account credit, deposits and payments. Credit limits and due dates shall be configurable by authorization.

### FR-7 Suppliers

The system shall support supplier contact data, purchasing history, balances and returns.

### FR-8 Expenses

The system shall support categorized expenses, branch attribution, approval, payment method and attachments where configured.

### FR-9 Registers and shifts

The system shall support opening floats, sales, tenders, cash movements, shift closing, over/short declarations and blind or reviewed close procedures according to business policy.

### FR-10 Branches

The system shall isolate or share data according to configured branch policies. A user shall not access a branch without membership and permission.

### FR-11 Employees and permissions

The system shall support activation, deactivation, role assignment, branch assignment and fine-grained permissions. Privileged actions shall require appropriate authorization and audit.

### FR-12 Reports

The system shall provide sales, tax, profit, purchase, inventory, expenses, customer, supplier, register, staff and branch reports. Exports shall preserve data scope and use auditable filters.

### FR-13 Printing

The system shall support thermal receipts, invoices, reports and barcode/label printing. Print failures shall be observable and shall not automatically replay completed sales or stock effects.

### FR-14 Import and export

The system shall support reviewed CSV/import mappings and complete, authorized business export. Imports shall be validated, idempotent where practical and reversible only through an approved data operation.

### FR-15 Backup and restore

Local edition shall provide encrypted export or backup, integrity manifests, restore preview and restore confirmation. Restore shall preserve current transaction/sequence integrity and shall require explicit confirmation.

### FR-16 Synchronisation

Where enabled and authorised, it shall synchronise business data and reconciliation events in both directions, preserve stable identifiers and checkpoints, detect duplicates and conflicts, and retry safely.

### FR-17 Licence and subscription

Local shall support a signed optional entitlement whose absence or expiry does not forcibly disable permitted local operations. Cloud shall support plans, grace, suspension and reactivation. Subscription billing shall not share identity or ledgers with customer payments.

## 4. Non-functional requirements

- Local core sale completion shall not require a live internet connection.
- Tenant and branch authorization shall be enforced fail closed.
- Business databases shall remain logically separate in Cloud.
- Password and secret values shall not be logged or exported.
- Browser and desktop sessions shall expire and rotate safely.
- Financial values shall use decimal types or integer minor units.
- The system shall record timezone and UTC timestamps for business events.
- The platform shall support data export, deletion and retention policies.
- Critical writes shall preserve idempotency keys and audit history.
- Public and private APIs shall use bounded requests and rate limiting.
- Destructive restore, export and deletion operations shall require confirmation.

## 5. Security requirements

Authentication, authorization, tenant resolution, audit logging, encrypted secrets, key management and secure release/update signing are specified in the security document.

## 6. Kenya localisation

Initial defaults shall use KES, Africa/Nairobi, Kenyan receipt conventions and KRA-ready tax structures. M-Pesa, bank and cash tenders shall remain configurable. eTIMS obligations and integration contracts require official verification before production claims.

## 7. Acceptance tests

- A sale completes while the internet is unavailable.
- Duplicate sale submission creates one sale and one stock effect.
- A refund reverses stock and financial totals once.
- One tenant cannot access another tenant's records, files or reports.
- A restricted user cannot access a disallowed branch or action.
- Restore produces the same business data and passes integrity verification.
- A low-level sync failure retries without replaying a completed payment or stock command.
- KES totals round consistently at the configured decimal/rounding policy.
- Universal search shows authorized navigable destinations and excludes restricted ones.

## 8. Verification gaps

The existing map does not establish:

- A complete live feature regression matrix.
- Verified KRA eTIMS endpoints or Kenya-specific legal rules.
- Verified M-Pesa merchant integration.
- Verified local or cloud disaster-recovery operations.
- Verified private sync Gateway or agent delivery.
- Legal clearance for source reuse or third-party components.
