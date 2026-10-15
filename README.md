# Core Banking Platform — Frontend Mobile

Flutter applications for customers and banking agents using the Core Banking Platform.

> **Project status:** Active development by a volunteer team. This software is **not approved for production banking operations**, real customer data, or real funds.

## Repository scope

This repository owns **mobile applications only**, built with Flutter and Dart:

| Application | Audience | Primary capabilities |
| --- | --- | --- |
| Customer Banking App | Individual banking customers | Onboarding, account access, transfers, payments, cards, loans, savings, notifications |
| Agent Banking App | Banking agents, field officers, microfinance officers, agency supervisors | Customer onboarding/verification, cash-in/out, transfers, bill payments, float, commissions, receipts, controlled offline workflows |

Related but independently maintained repositories:

- `frontend-user` — Individual and corporate internet banking web applications.
- `frontend-admin` — Staff operations, system administration, risk/compliance, and finance/reporting web applications.

The backend owns authentication decisions, balances, transaction processing, approval rules, and authoritative financial calculations. Mobile clients interact **only through approved APIs**, never directly with databases.

## Project objectives

- Provide clear, accessible, reliable banking journeys on Android and iOS.
- Support multi-institution branding and configurable feature availability.
- Deliver predictable behavior on poor networks and low-memory devices.
- Reuse design tokens, API contracts, validation rules, and patterns where appropriate.
- Protect customer identity, device credentials, financial information, and transaction integrity.
- Keep the customer and agent applications separately buildable, testable, and releasable.

## Application features

### Customer Banking App

**Identity and security**
- Secure onboarding, registration, and sign-in
- Multi-factor authentication and step-up verification
- Biometric unlock where supported and approved
- Device binding, device management, and session timeout
- Profile, security settings, and notification preferences

**Accounts and transactions**
- Account overview, balances, and transaction history
- Internal and interbank transfers
- Beneficiary management
- Bill payments, airtime, and mobile data
- QR payments where supported
- Review/confirm/submit flows and transaction receipts

**Other services**
- Card controls
- Loan applications
- Savings goals
- Statements and receipt access
- Push notifications and secure customer support

### Agent Banking App

**Access and customers**
- Agent sign-in, device binding, and supervised access
- Customer registration and identity verification
- Device/terminal status and agent support

**Agent operations**
- Cash deposits and withdrawals
- Transfers, bill payments, airtime, and data purchases
- Loan collections and disbursement support
- Agent float and commissions
- Transaction receipts and daily activity summary
- Customer transaction lookup according to permissions

**Connectivity and sync**
- Connectivity-aware UI and clear pending/error states
- Controlled offline capture only for explicitly approved, low-risk operations
- Background synchronization with conflict and replay safeguards

> **Critical rule:** Offline records are **not completed financial transactions**. Never display a cash movement or account balance as final until the backend confirms it. Cash-in/out and other value-moving operations must not be executed offline unless the backend explicitly implements an approved, audited offline authorization protocol.

## Recommended technology stack

| Area | Recommendation |
| --- | --- |
| Mobile framework | Flutter and Dart |
| State management | Riverpod or Bloc; select one and document the decision |
| HTTP networking | Dio with centralized interceptors and timeout handling |
| API contracts | OpenAPI-generated clients where feasible |
| Secure storage | Platform-backed secure storage for approved credentials only |
| Local persistence | Approved local database, with encryption and data minimization |
| Routing | Approved Flutter routing package |
| Push messaging | Platform-approved push notification integration |
| Tests | Flutter unit, widget, and integration tests |
| Quality | `dart format`, `flutter analyze`, dependency and secret scans |

Versions and libraries must match the committed project configuration; do not infer them from this README.

## Suggested repository structure

```text
frontend-mobile/
├── apps/
│   ├── customer_banking/
│   │   ├── lib/
│   │   ├── test/
│   │   ├── integration_test/
│   │   ├── android/
│   │   └── ios/
│   └── agent_banking/
│       ├── lib/
│       ├── test/
│       ├── integration_test/
│       ├── android/
│       └── ios/
├── packages/
│   ├── design_system/
│   ├── api_client/
│   ├── authentication/
│   ├── secure_storage/
│   ├── validation/
│   └── shared_utils/
├── docs/
│   ├── architecture/
│   ├── security/
│   ├── offline-sync/
│   ├── accessibility/
│   └── release/
├── .github/workflows/
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── LICENSE
└── README.md
```

This is a **target structure**, not a claim that the directories already exist. Shared packages must be genuinely reusable; avoid putting app-specific logic into them. Keep native platform configuration inside each app.

## Local development

### Prerequisites

- Flutter SDK, with the project-approved version
- Dart SDK (bundled with Flutter)
- Android Studio and Android SDK, or another configured Android development environment
- Xcode on macOS for iOS builds
- A simulator/emulator or physical test device
- Git and a supported IDE (e.g. VS Code)
- Access to approved development APIs and test credentials

Check your setup:

```bash
flutter doctor
flutter devices
```

From the intended app directory, install packages and run:

```bash
cd apps/customer_banking
flutter pub get
flutter run --dart-define=APP_ENVIRONMENT=development
```

For agents:

```bash
cd apps/agent_banking
flutter pub get
flutter run --dart-define=APP_ENVIRONMENT=development
```

Commands are illustrative until the new repository is scaffolded. If using a multi-package Flutter workspace, document its actual bootstrap command and toolchain.

## Environment configuration

Use separate, approved configurations for development, test, staging, and production. Examples of **non-secret** build-time settings:

```bash
flutter run \
  --dart-define=APP_ENVIRONMENT=development \
  --dart-define=API_BASE_URL=https://api-dev.example.invalid
```

- Do not commit API keys, signing material, credentials, or real customer information.
- `--dart-define` values can be extracted from a distributed app: **never treat them as secrets**.
- Keep signing keys and CI secrets in approved secret managers.
- Maintain separate app IDs/bundle identifiers and signing profiles for each app/environment.
- Use fake/sandbox identities and synthetic financial data during development.

## Authentication and authorization

- Use approved identity APIs for sign-in, MFA, session renewal, logout, and trusted-device flows.
- Request biometric authentication only through supported platform mechanisms; biometrics do not replace backend authorization.
- Keep the amount and lifetime of stored credentials minimal; use platform-backed storage when credentials must persist.
- Avoid credentials in logs, crash reports, analytics, screenshots, clipboard, and deep links.
- Enforce short-lived sessions and appropriate reauthentication for sensitive actions.
- Agent roles and permissions must be checked by the backend on **every protected action**.
- A hidden button or restricted screen is not a security boundary.

## Transaction UX and integrity

Every value-moving journey should:

1. Gather and validate inputs.
2. Display the payee, account, amount, currency, and applicable fees.
3. Show a final confirmation screen.
4. Apply step-up authentication or maker-checker approval where required.
5. Submit with backend-supported idempotency/replay protection.
6. Show a **processing** state while the result is unknown.
7. Confirm the authoritative status through the backend before declaring success or failure.
8. Show a reference and receipt only when the backend provides or authorizes them.

Prevent double-taps and uncontrolled retries. A timeout is **not** proof that a transaction failed. Never invent balances, success receipts, exchange rates, or fees on the device.

## Connectivity and offline policy

- Distinguish **draft**, **queued**, **submitted**, **processing**, **confirmed**, and **failed** states.
- Synchronize only permitted records and only through authenticated, approved APIs.
- Use bounded retries with exponential backoff for safe operations.
- Use idempotency keys where required by backend contract.
- Prevent duplicate replay, stale data presentation, and inconsistent client/server state.
- Encrypt sensitive cached data where storage is unavoidable, and apply retention/expiry rules.
- Clearly label cached balances and histories as potentially outdated.
- On shared/compromised devices, minimize exposure and follow the project's device-security policy.

## Accessibility and visual design

- Follow the approved cross-platform design system: color, typography, spacing, forms, transaction summaries, and feedback states.
- Aim for WCAG 2.1 AA-equivalent accessibility in applicable mobile journeys.
- Support screen readers, dynamic text scaling, contrast, touch targets, and logical focus order.
- Provide loading, empty, validation, offline, pending, error, success, permission-denied, and session-expired states.
- Test small displays, reduced-motion preferences, rotation where supported, and slow network scenarios.
- Source institution branding from approved configuration where appropriate; avoid hard-coded names/currencies.

## Security and privacy baseline

Never log or send to analytics:

- Passwords, OTPs, access/refresh tokens, PINs, private keys
- Card CVVs or full payment-card numbers
- Full BVNs, NINs, or customer identity documents
- Sensitive banking details or raw API responses containing protected data

Also:

- Use TLS and approved API hosts; reject insecure transport in release builds.
- Minimize app permissions and request them contextually.
- Follow mobile threat modeling and OWASP MASVS guidance.
- Prevent exposure through notifications, screenshots, and background app previews where risk requires it.
- Do not hard-code encryption keys or implement custom cryptography.
- Route vulnerability reports privately through `SECURITY.md`.

## Testing strategy

| Level | Examples |
| --- | --- |
| Unit | Currency/date formatting, validators, state transitions, error mapping |
| Widget | Login, account cards, transaction review, empty/error/pending states |
| Integration | Authentication, API client, biometric fallback, device registration, offline synchronization |
| End-to-end | Customer onboarding, transfer, bill payment, statement access; agent cash workflow and receipt |
| Security | Token protection, unauthorized access, tampering, duplicate submissions, logout/session expiration |
| Accessibility | Screen readers, text scaling, contrast, keyboard/focus where applicable |
| Device/network | Android/iOS, low memory, loss of connectivity, interrupted responses, background/foreground |

Typical checks in each app:

```bash
flutter format --set-exit-if-changed .
flutter analyze
flutter test
```

Use `dart format --set-exit-if-changed .` if `flutter format` is not supported by the pinned SDK/toolchain. Run `flutter test integration_test` or the configured device-test command for integration suites.

For financial requests, explicitly test insufficient funds, account restrictions, exceeded limits, timeouts, replay attempts, unknown status, reversals, and provider failures.

## CI/CD and releases

Each app should have independent pipelines for:

1. Formatting, static analysis, dependency and secret scanning.
2. Unit/widget/integration testing, including security-critical journeys.
3. Android/iOS build validation.
4. Signing through secured CI infrastructure.
5. Deployment to an approved test track/TestFlight or internal channel.
6. Manual review and authorization before production promotion.

Maintain separate package names, signing identities, version codes, store listings, privacy disclosures, and release notes for the customer and agent apps. Never put signing credentials into the repository.

## Collaboration and contributions

This is currently an **unpaid volunteer project**. Participation does not create employment or guarantee compensation.

- Read `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `SECURITY.md` before contributing.
- Do not commit directly to `main`.
- Use focused branches such as `feature/TASK-0410-customer-transfer-review` or `fix/TASK-0411-agent-sync-retry`.
- Use conventional commits (for example, `feat: add agent float summary`).
- Pull requests should include task/issue, affected app, UI evidence, API dependencies, test results, device coverage, accessibility notes, and security implications.
- Do not include real customer data in issues, logs, or screenshots.

## Definition of done

A feature is complete when its approved design, API contract, authorization checks, all relevant UX states, accessibility review, automated tests, security checks, and documentation are complete and reviewed. Transaction features additionally require confirmed backend behavior for idempotency, status reconciliation, and failure recovery.

## License and disclaimer

Refer to the repository's `LICENSE` file once included. Do not contribute third-party code or assets without the necessary rights.

This project is for development, education, and engineering collaboration. **Do not deploy it for handling real customer funds or regulated banking workloads** without formal security, regulatory, legal, privacy, performance, penetration-testing, disaster-recovery, and production-readiness approvals.
