# BiasharaOne Component Provenance Register

| Classification | Meaning |
|---|---|
| 1. Functionality documented | Independent behavior requirement with no reuse right claimed |
| 2. Potential reuse after clearance | Technically useful component but legal and ownership review required |
| 3. Restricted or unsuitable | Rebrand/product/dependency boundary or AGPL/contributor/trademark concern |
| 4. Third-party dependency | Separate licence and notice review required |
| 5. Unknown/verify | Ownership, source or exact licence not verified |

## 1. Project and identity

| Entry | Path | Copyright holder if known | Licence/status | Classification | Reason | Action |
|---|---|---|---|---|---|---|
| Electron application shell | `src/main.js`, `src/server.js`, `src/mongodb-manager.js`, `src/api-runtime.js` | Existing Posnic project evidence | Repository AGPL-3.0-only | 2 | Lifecycle patterns may inform independent design | Legal clearance before reuse; rebuild new product identity |
| Preload/IPC | `src/preload.js`, `src/hardware-ipc.js`, `src/ipc-guard.js` | Existing Posnic project evidence | Repository AGPL-3.0-only | 2 | Useful security model, but implementation remains source | Review/adapt only after licence decision; add independent tests |
| Express API | `api/server.js`, `api/app.js`, `api/src/routes/` | Existing Posnic project evidence | Repository AGPL-3.0-only | 2 | Large implementation with tenant gap | Independent implementation or recorded clearance |
| Static frontend | `frontend/static/`, `frontend/gulpfile.js` | Existing Posnic project evidence plus bundled assets | AGPL plus third-party assets | 2/4 | Useful workflow evidence; vendor assets may differ | Clean-room UI or clearance-specific reuse; preserve notices |
| Mongo models/repositories/services | `api/src/models/`, `api/src/repositories/`, `api/src/services/` | Existing Posnic project evidence and contributors | AGPL-3.0-only | 2 | Functional and implementation evidence; direct ownership not transferable | Independent design/implementation or clearance |
| Printing and hardware | `src/escpos-*`, `src/hardware-ipc.js`, `src/raw-print-service.js` | Existing Posnic project evidence plus third-party stack | AGPL plus third-party | 2/4 | Functional device behavior is documented; implementation licence matters | Legal review, dependency audit and independent tests |
| Backup manager | `src/backup-manager.js` | Existing Posnic project evidence | AGPL-3.0-only | 2 | Useful integrity/recovery concepts | Independent implementation or clearance |
| Sync manifest/outbox | `api/src/sync/` | Existing Posnic project evidence | AGPL-3.0-only | 2 | Sync policy evidence but transport incomplete | Independent protocol; verify private Gateway/agent separately |
| Product names/logos/icons/release identity | `builds/`, `package.json`, `src/main.js` | Posnic Innovations/trademark owners and other asset owners | Trademarks and reserved names | 3 | Explicitly not to be reused | Replace with independent brand/assets/signing identity |
| Governance and DCO | `docs/GOVERNANCE.md` | Posnic Innovations and contributors | Project policy | 3 | Defines ownership boundaries | Preserve in source review; do not transplant as BiasharaOne policy |

## 2. External integrations

| Entry | Path/evidence | Classification | Action |
|---|---|---|---|
| Posnic Gateway | `src/main.js`, `src/cloud-activation.js`, `src/sync-agent-manager.js` | 3 | Replace with independently operated/authorized service contracts |
| GitHub updater identity | `src/update-service.js`, `package.json`, `dev-app-update.yml` | 3/4 | Replace repository, signing keys and updater identity |
| SMS/WhatsApp providers | `api/src/services/` | 4 | Choose providers under separate contracts and licences |
| Email/Brevo/Nodemailer | `api/src/utils/email.js`, package manifests | 4 | Independent provider and template review |
| AWS S3/Textract/Bedrock | `api/src/utils/s3.js`, provider services | 4 | Separate cloud terms, tenancy and notice review |
| Razorpay and payment adapters | Sale/payment routes, package dependencies | 4 | Separate customer tender integration; not platform billing |
| AI APIs | `api/src/services/ai.service.js` | 4/5 | Review provider terms, opt-in, billing and data handling |
| Analytics | Analytics configuration | 5 | No assumptions; verify property ownership and consent |

## 3. Third-party notices and dependencies

Every npm dependency, bundled MongoDB binary, font, icon, vendor JavaScript, installer tool and platform framework requires package-specific legal review. The repository's `THIRD-PARTY-NOTICES.md` and component licences are evidence only; they do not clear new proprietary or independent implementation.

Required entries shall include path, package/version, copyright, licence, modifications, network-distribution impact and replacement or clearance action.

## 4. Unknown items

Items without a clear copyright statement or exact licence are classification 5 and shall not be reused until the legal owner and rights are documented.

## 5. Recommended implementation decision

- Functionality may inform independent requirements.
- Existing source may not be copied or close-rebranded until counsel and licensing clearance.
- Third-party dependencies require separate licence/notice review.
- Posnic names, marks, logos, protocols and release identity are excluded.
- Cloud, sync and private components must be designed or separately licensed.
- Unknown assets and dependencies remain blocked.
