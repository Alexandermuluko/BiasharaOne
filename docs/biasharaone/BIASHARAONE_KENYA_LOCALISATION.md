# BiasharaOne Kenya Localisation

## 1. Currency and timezone

- Currency: Kenyan shilling (KES)
- Currency minor unit: cents
- Timezone: Africa/Nairobi
- Store UTC timestamps and display local time explicitly
- Document tax and rounding behavior per official Kenya rules

## 2. Tax configuration

The system shall support:

- Configurable tax IDs and tax rates
- Inclusive and exclusive tax calculations
- Tax categories and exemptions
- Branch/document tax settings
- Tax reports suitable for qualified accountant review

Vat/eTIMS amounts and tax codes shall not be assumed. The implementation shall use official KRA documentation and sandbox contracts.

## 3. eTIMS status

eTIMS integration is an optional fiscal adapter. Before release:

- Verify registration and taxpayer scope
- Verify official API/authentication contract
- Verify receipt fields and control-number behavior
- Verify reversal/credit-note behavior
- Complete sandbox testing
- Obtain qualified legal/accounting sign-off
- Document downtime and reconciliation behavior

No eTIMS compliance claim is made in this documentation.

## 4. M-Pesa

M-Pesa shall be an optional payment provider. It shall support:

- Merchant configuration
- Explicit initiation
- Provider callback verification
- Stable provider reference
- Timeout and unknown-status handling
- Reconciliation
- One confirmed tender per sale
- Refund/reversal only through an authorized provider flow

Customer tender payment remains separate from platform subscription billing.

## 5. Other payments

The system shall support cash, bank transfer, card or terminal tenders only through reviewed integrations. Provider operations shall remain tied to business and branch scope.

## 6. Receipts

Receipt templates shall support:

- Business name and registration details configured by the business
- Branch/address/contact
- Item, quantity, price, discount and tax
- Payment tender
- Customer reference when explicitly allowed
- Receipt number
- KRA/eTIMS fields when officially verified
- Optional QR/barcode

Receipt requirements shall be configurable and validated; the system shall not print unverified fiscal claims.

## 7. Language and formatting

English and Kiswahili support shall be considered in UI, receipts and exports. Initial implementation shall define supported locales and allow fallback to English.

Date, currency, phone and address formats shall follow locale configuration while preserving stable internal values.

## 8. Offline

Kenyan retail networks and power may be unreliable. Local checkout, stock and customer-credit operations shall work without internet when permitted. External fiscal or payment submissions shall queue with visible status and reconciliation.

## 9. Data protection

Map tenant, customer and employee data responsibilities. Define:

- Lawful purpose
- Consent where required
- Retention
- Export/deletion
- Processor relationships
- Support access
- Cross-border hosting
- Incident reporting

Qualified counsel should verify applicable obligations.

## 10. Acceptance criteria

- KES calculation and display pass reviewed cases.
- Africa/Nairobi is used consistently for local-day reports.
- Fiscal fields remain optional until verified and explicitly enabled.
- M-Pesa callback replay cannot duplicate a sale or tender.
- Offline sale completes and fiscal/payment submission remains pending visibly.
- Receipt customization cannot print unsupported tax/fiscal claims.
