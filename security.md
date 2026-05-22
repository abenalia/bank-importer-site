---
layout: default
title: Security Overview · bank_importer
---

# Security Overview

This page summarizes the security posture of the `bank_importer` project for users, partners, and integration reviewers. The detailed operational policy (including internal runbooks and dashboards) is maintained in the project's private repository and is reviewed quarterly.

## Architecture in scope

`bank_importer` is a **single-operator local utility**. It runs on the operator's own workstation, binds only to the loopback interface (`127.0.0.1`), and has no consumer-facing surface. There are no employees or contractors; storage is delegated to Google Sheets and Plaid.

## Key controls

### Identity and access

- Multi-factor authentication is enforced on every system that handles sensitive data: GitHub, the Google account that owns the Cloud project, the Plaid dashboard, the Apple ID, and the macOS workstation.
- SMS-based 2FA is explicitly avoided in favor of TOTP, hardware security keys, or platform authenticators.
- Roles, permissions, and access reviews follow a formal role-based access-control model. Quarterly access reviews are performed across every provider.
- A defined de-provisioning runbook covers device-loss and credential-exposure events.

### Secrets management

- No credentials are ever committed to version control.
- A `gitleaks` pre-commit hook blocks accidental secret commits.
- Credentials live only on the operator's FileVault-encrypted device.
- Documented rotation procedures for every credential type.

### Authentication tokens and TLS

- OAuth 2.0 with least-privilege scopes: only the Google `spreadsheets` scope and the Plaid `transactions` product are requested.
- All outbound API calls use HTTPS with full certificate validation. Validation is never disabled in application code.
- No password-based authentication against any third-party service.

### Vulnerability management

- GitHub Dependabot monitors dependencies continuously for security advisories.
- `pip-audit` runs in CI on every push and pull request.
- A defined patching SLA: critical and high vulnerabilities are patched within **7 days** of disclosure; medium within 30; low at the quarterly review.
- The macOS workstation receives automatic OS-level security updates and runs the latest major release of macOS.

### Privacy and data handling

- Detailed in the [Privacy Policy](privacy.html). Highlights:
  - No advertising identifiers, analytics, telemetry, crash reports, or device fingerprints are collected.
  - Bank-export uploads are deleted from the working directory within 90 days.
  - Plaid API client secrets are rotated annually at minimum.
  - The user (also the operator) can exercise access, export, correction, deletion, and consent-withdrawal rights at any time by direct action.

## Reporting a security issue

If you believe you have found a security issue in `bank_importer`, please report it privately to **adam@benaliaprice.com**. Do not open a public issue.

A response will be provided within 3 business days.

## Compliance references

The internal policy is designed to align with selected controls from NIST SP 800-53 (the AC, SI, RA, and CA families), with Plaid's developer requirements for Production access, and with the privacy obligations of GDPR Article 17 and CCPA §1798.105 (exercisable by the operator as the sole data subject).

---

*Last updated: 2026-05-22.*
