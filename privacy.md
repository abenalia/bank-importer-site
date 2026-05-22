# Privacy Policy

This privacy policy describes how the `bank_importer` project collects, uses, stores, and protects information. It is maintained in the project repository and version-controlled in git; revisions are tracked in the **Revision History** table below.

## Revision History

| Version | Date       | Notes |
| ------- | ---------- | ----- |
| 1.0     | 2026-05-22 | Initial publication. |
| 1.1     | 2026-05-22 | Replaced brief Retention section with a formal *Data Retention and Deletion* policy: per-data-type retention periods, secure deletion procedures, review cadence, and legal-alignment notes (GDPR Art. 17, CCPA §1798.105, US tax record-keeping, Plaid policy). |

## Scope and operator model

`bank_importer` is a **single-operator local utility**. It is not a consumer-facing product, has no user-account model, has no login surface, and is not hosted on the public internet. The "user" whose data is collected by the application is also the **operator** of the application — the same individual.

This policy is written in the conventional privacy-policy structure so that it remains directly portable if the application is ever extended to additional users. The transition triggers and prerequisite controls are listed in the *Deferred Controls — Pre-Deployment Checklist* of `SECURITY.md`. Until those triggers occur, the policy describes the operator's commitments regarding handling of their own financial data.

## Information collected

The application collects:

- **Plaid-derived transaction records.** When a financial institution is linked via Plaid Link, the application requests the read-only `transactions` product. Records retrieved include: date, amount, merchant name (where provided), Plaid-supplied category, account identifier, and Plaid-assigned transaction identifier. The application requests no other Plaid products.
- **Plaid Item access tokens.** Stored in `data/plaid_items.json` to enable subsequent transaction fetches without re-linking the institution.
- **Google OAuth tokens.** A refresh token stored in `data/token.json` that lets the application write to the operator's Google Sheet on the operator's behalf. Scope is restricted to `https://www.googleapis.com/auth/spreadsheets`.
- **Operator-provided file uploads.** When the operator uploads a bank export (CSV / XLS / XLSX / PDF) via the local UI, the file is parsed and the resulting transactions are normalized for inclusion in the Sheet. The original file is retained at `data/uploads/` until the operator removes it.

The application **does not collect**:

- Advertising identifiers.
- Device fingerprints, IP geolocation, or browser fingerprints.
- Analytics or telemetry events.
- Crash or error reports.
- Any data from any user other than the operator.

## How information is used

Collected information is used **only** for the application's stated purpose: importing transactions from bank exports or from Plaid, normalizing the records, and appending them to a Google Sheet owned by the operator.

The application **does not**:

- Use the data for advertising, marketing, profiling, or any form of targeting.
- Train, fine-tune, or evaluate any machine-learning or AI model on the data.
- Sell, rent, or share the data with any third party.
- Aggregate the data with data from other users (no other users exist).

## How information is stored

| Data | Location | Protection |
| ---- | -------- | ---------- |
| Plaid Item access tokens | `data/plaid_items.json` on the operator's local device | Gitignored; FileVault full-disk encryption on the device |
| Google OAuth refresh token | `data/token.json` on the operator's local device | Gitignored; FileVault encryption |
| Google OAuth client secret | `credentials.json` at project root | Gitignored; FileVault encryption |
| Plaid API client secret | `.env` at project root | Gitignored; FileVault encryption |
| Transaction records (working copies, parsed CSVs) | `data/uploads/` and in-memory during processing | Gitignored; FileVault encryption |
| Transaction records (canonical) | Operator-owned Google Sheet | Protected by the operator's Google account and 2-Step Verification; access scope restricted to the operator |
| Source code | `github.com/abenalia/Banking` (private repository) | Private; access restricted to the operator's GitHub account with mandatory MFA |

Plaid retains its own copies of transaction data per its own end-user privacy policy.

## How information is protected

The application's security controls are documented in detail in the [Security Overview](security.html) and include:

- Full-disk encryption on the operator's device (FileVault).
- Multi-factor authentication on every account that has access to the data (GitHub, Google, Plaid, Apple ID, iCloud Keychain). See *MFA Inventory* in `SECURITY.md`.
- Loopback-only Flask binding — the application is not reachable from any remote network.
- Secrets exclusion from version control via `.gitignore`, with continuous enforcement by a `gitleaks` pre-commit hook.
- Dependency vulnerability monitoring via Dependabot and `pip-audit` in CI, with a documented patching SLA. See *Vulnerability Management* in `SECURITY.md`.
- Quarterly access reviews across every provider that touches the data. See *Access Reviews and Audits* in `SECURITY.md`.
- A documented de-provisioning runbook covering device-loss and credential-exposure events.

## User rights

The user — who is also the operator — retains direct control over their data at all times and can exercise the following rights by direct action, without any support request or response SLA:

- **Access.** All collected data is directly viewable in the operator-owned Google Sheet, in `data/`, and in `data/uploads/`.
- **Export.** Transaction records can be exported as CSV from the Google Sheet at any time via Google's export functionality.
- **Correction.** Records in the Sheet can be edited directly. Records in `data/uploads/` are read-only inputs; corrections happen at the Sheet level.
- **Deletion.** The operator may delete any record from the Sheet, delete the entire Sheet, delete `data/`, delete `data/uploads/`, or uninstall the application. None of these actions require contacting anyone.
- **Withdraw consent for Plaid.** Items can be unlinked from the Plaid dashboard at any time, immediately revoking the application's ability to fetch further transactions for that institution. The corresponding access token in `data/plaid_items.json` should be removed during the next access review.
- **Withdraw consent for Google.** Access can be revoked at <https://myaccount.google.com/permissions>, invalidating the existing `token.json`.
- **Discontinue use.** The application can be stopped (close the Flask process) and uninstalled at any time.

## Data Retention and Deletion

This is the formal data retention and deletion policy for the project. It is reviewed quarterly alongside the access review and is enforced by the operator directly — because the operator is also the data subject, every retention/deletion right is exercised by direct action.

### Retention periods

| Data type | Retention period | Trigger for deletion |
| --------- | ---------------- | -------------------- |
| Plaid transaction records (canonical, in the Google Sheet) | Operator-defined; **default 7 years** (consistent with US tax record-keeping guidance) | Operator removes rows or deletes the Sheet. |
| Plaid Item access tokens (`data/plaid_items.json`) | Lifetime of the linked Item | Immediate deletion when the Item is unlinked at Plaid. Verified every quarterly access review. |
| Google OAuth refresh token (`data/token.json`) | Until rotated, revoked, or **90 days idle** | Immediate deletion on revocation or rotation; quarterly cleanup of idle tokens. |
| Google OAuth client secret (`credentials.json`) | Until rotated or replaced | Replaced on suspected exposure or at quarterly review when older than 12 months. |
| Plaid API client secret (`.env`) | **12 months maximum** | Annual rotation; immediate rotation on suspected exposure. |
| Bank export uploads (`data/uploads/`) | **90 days from upload** | Operator review every 90 days; expired files deleted securely. |
| Local source-code working copies | Project lifetime | Deleted with `rm -rf` on the FileVault-encrypted volume when no longer in use. |

### Secure deletion procedures

| Storage location | Procedure |
| ---------------- | --------- |
| Local files (`data/`, `credentials.json`, `.env`) | `rm` on the FileVault-encrypted volume. AES-128-XTS encryption at rest, bound to the operator's account, provides cryptographic erasure on device wipe (see *De-provisioning Runbook* in `SECURITY.md`). |
| Google Sheet rows or the Sheet itself | Deletion via the Google Sheets UI; Google performs a 30-day soft-delete followed by permanent removal per Google's published retention policy. |
| Plaid Items | Detached via the Plaid Dashboard or the `/item/remove` API. Plaid retains its own records per its published retention policy. |
| Local git repository | `rm -rf` on the FileVault-encrypted volume. |
| GitHub-hosted history (when sensitive content is detected) | `git filter-repo` followed by force-push, then a confirmatory `gitleaks detect` pass — the same procedure already applied to the historical secret exposure recorded in `SECURITY.md`. |

### Review cadence

This policy is reviewed quarterly alongside the access review in `SECURITY.md`. The review confirms:

1. Every retention period still reflects current law and Plaid policy.
2. The operator has applied the 90-day upload-deletion rule for `data/uploads/`.
3. No tokens older than the retention period are still present.
4. No acknowledged exceptions exist.

Material changes are recorded in the Revision History at the top of this document.

### Legal alignment

This policy is designed to be consistent with:

- **GDPR Article 17** (right to erasure) — operator-as-data-subject can erase data by direct action at any time.
- **CCPA §1798.105** (right to delete) — same mechanism.
- **Plaid's End User Privacy Policy** retention rules for Plaid-sourced data.
- **US tax record-keeping guidance** (IRS — retain records supporting income/deduction claims for 3 to 7 years). The 7-year default for transaction records is the conservative upper bound.

### Data minimization

Retention is paired with collection minimization: the application requests only the `transactions` Plaid product and the `spreadsheets` Google scope (see *Authentication Tokens and Certificates* in `SECURITY.md`). No advertising identifiers, analytics, telemetry, crash reports, or device fingerprints are collected.

## Third-party processors

The following third parties may receive or hold data on behalf of the operator:

| Third party | Role | Their privacy policy |
| ----------- | ---- | -------------------- |
| **Plaid** | Provides transaction data on behalf of the operator's financial institutions. Holds access tokens and transaction copies per its own retention policy. | <https://plaid.com/legal/> |
| **Google** | Hosts the target Sheet and authenticates the operator. | <https://policies.google.com/privacy> |
| **The operator's financial institutions** | Source of the transaction data prior to Plaid. | Each institution publishes its own policy. |

No other third party receives any data from the application.

## Children

The application is not directed at children under 13 (or 16 in jurisdictions where that threshold applies). It does not knowingly collect personal information from children. As the application is single-operator, this provision is in practice satisfied by the operator's own age.

## Changes to this policy

Material changes to this policy are recorded in the Revision History table at the top of this document and committed to the project repository. The git commit history is the authoritative record of every change.

## Contact

Privacy inquiries: **adambenalia287@gmail.com**

A response will be provided within 3 business days.
