# BiasharaOne Security Requirements

## 1. Security objectives

- Strict tenant isolation
- Strict branch and role authorization
- Protected customer and employee data
- Offline operation without embedded secrets
- Safe payment and stock transitions
- Verifiable software updates
- Auditable platform support
- Secure backups and restoration
- Privacy-preserving diagnostics

## 2. Identity

Separate business identities from platform identities. A business owner shall not receive platform administration rights solely by being an account owner.

Passwords use a reviewed password-hash algorithm and appropriate work factors. MFA is recommended or required according to role. Device and session revocation shall apply promptly.

## 3. Authorization

Authorize by tenant membership, branch access, role, permission and explicit feature gate. Server-side authorization is mandatory; hidden UI controls are not security.

Tenant resolution precedes authentication and data access. Arbitrary database or tenant names from clients are prohibited.

## 4. Tenant isolation

Cloud requests use a trusted tenant resolver and separate database context. Direct database helpers and background jobs shall not inherit or choose a previous tenant.

All direct collection access, reports, exports, search, uploads and audit operations shall be tested under concurrent tenants.

## 5. Session and token security

Use secure, HttpOnly, SameSite cookies or reviewed bearer-token handling. Rotate session identifiers after privilege changes. Set short appropriate lifetimes and enforce logout and deactivation.

Issuers, audiences and signing secrets shall be deployment-specific. Shared cookies, secrets or fallback trust shall not cross control/business boundaries.

## 6. Secrets

Store provider credentials, database credentials and signing keys in a secret manager or protected OS store. Never place production secrets in source, environment examples, logs or installers.

Support provider credentials by tenant or business scope. Rotatable credentials shall have documented replacement and rollback procedures.

## 7. Encryption

Use TLS for all network traffic. Encrypt Cloud databases/backups according to reviewed infrastructure controls. Encrypt local backups and retain passphrases outside the package.

Keys shall have explicit ownership, rotation and recovery. Platform keys shall not be accessible to ordinary business API processes unless operationally required.

## 8. Audit and logs

Record actor, action, target, tenant/branch, outcome, time and request trace. Record provisioning, licence, subscription, backup, support and privileged administration actions.

Never log passwords, provider secrets, complete card data or unnecessary identity data. Redact diagnostic exports and apply retention.

## 9. Desktop security

Use Electron context isolation and a narrow capability bridge. Do not expose unrestricted Node, file-system or IPC APIs to a renderer.

Validate sender window/origin and IPC channel schemas. Bind loopback API ports and keep MongoDB access restricted to the authorized local process/network.

## 10. Payment security

Never store unnecessary sensitive card or mobile payment credentials. Store provider references and status only. Verify provider webhooks with signatures and replay protection.

Customer tender payments and platform subscription billing shall use separate ledgers and provider boundaries.

## 11. Software supply chain

Review every dependency for licence, origin and vulnerability. Generate a package/artifact SBOM and provenance if production release requirements call for it.

Sign desktop updates and installers using human-owned certificates/keys. Document key rotation and rollback. Inspect actual package contents before distribution.

No private sync source or private infrastructure component shall be copied into public customer installers without explicit written authorization.

## 12. Vulnerability and incident response

Maintain a private reporting route, response targets and affected-version process. Retain evidence for incidents without publishing customer data. Revoke compromised access and credentials promptly.

## 13. Data protection

Define lawful purpose, consent, collection minimum, access, export, correction and deletion workflows. Apply retention and secure deletion for customer, employee and audit data according to legal review.

## 14. Backup and restore

Backups shall be encrypted, scoped, checksummed and protected by access control. Restore operations require explicit authorization and audit. Perform restore drills against the RPO/RTO goals.

## 15. Kenya and privacy review

No claim of compliance with Kenya Data Protection Act, KRA/eTIMS or payment rules shall be made without official or qualified legal review. Requirements shall include processor roles, retention, official API terms and breach/incident obligations.
