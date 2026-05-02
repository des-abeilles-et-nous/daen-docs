# DAEN-SCOUT — Test Strategy

> **Scope of this document:** defines *what* to test, *how* to approach it, and *where* each testing activity belongs.
> Test plans, test runs, and execution results are managed in **Jira**.

---

## 1. Objectives

The DAEN-SCOUT test strategy aims to:

- Validate the correctness and stability of each ecosystem component in isolation and in integration.
- Protect production data and users from regressions introduced by new features or backend changes.
- Provide a shared reference for developers, QA, and AI coding agents when designing or reviewing changes.

---

## 2. Ecosystem Scope

The strategy covers all four repositories of the DAEN-SCOUT ecosystem and the Firebase services they share.

| Component | Repository | Primary test concerns |
|---|---|---|
| Mobile client | `daen-scout` | UI behaviour, Redux state, Firebase client SDK integration |
| Backend workers | `daen-fb-workers` | Cloud Functions logic, task queue, Firestore/RTDB triggers |
| Admin CLI | `fb-admin` | Bulk operations, service account access, data consistency |
| Shared libraries | Bit components (`daen-objects`, `daen-firebase`, `daen-utils`) | Data model integrity, helper correctness |

Firebase services that cut across all components:

- **Firestore** — business objects (`POIs`, `users`, `tiles_view`, `sequences`)
- **Realtime Database** — transitive data (`buffer`, `tasks`, `logs`, `feedbacks`, `subs`)
- **Authentication** — user identity and session
- **Cloud Storage** — media assets
- **Pub/Sub / Cloud Scheduler** — scheduled function triggers

---

## 3. Test Environments

All test activities use the four-tier environment system shared by all repositories.

| Environment | Firebase project | Purpose | Who uses it |
|---|---|---|---|
| **dev** | `bsc-dev-7a548` | Integration testing during development | Developers |
| **sandbox** | `bsc-sandbox-9fbeb` | QA, exploratory testing, feature validation | Internal testers |
| **staging** | `dsc-staging-eu` | Pre-production regression and acceptance | Beta users, pilots |
| **live** | `dsc-live-eu` | Production — smoke tests only, no destructive data | On-call / release manager |

> Destructive test data (POI creation, feedback injection, user creation) must **never** run against `live`.
> Each environment is fully isolated with its own database, credentials, and user base.

---

## 4. Test Types and Responsibilities

### 4.1 Unit tests
- **Scope:** pure functions, data transformations, business rule helpers in `daen-utils`, `daen-objects`.
- **Owner:** developer writing the code.
- **Environment:** local only, no Firebase dependency.
- **Tooling:** Jest (existing in `daen-fb-workers`).

### 4.2 Integration tests
- **Scope:** Cloud Functions interacting with Firestore/RTDB, task queue lifecycle, trigger chains.
- **Owner:** backend developer + QA.
- **Environment:** `dev` or `sandbox` (Firebase Emulator Suite preferred for `dev`).
- **Tooling:** Firebase Emulator Suite + Jest.
- **Principle:** prefer real Firebase interactions over mocks — see ECOSYSTEM_CONTEXT.md development standards.

### 4.3 End-to-end (E2E) tests
- **Scope:** full user journeys through the mobile app (POI reporting, feedback, alert subscription).
- **Owner:** QA.
- **Environment:** `sandbox`.
- **Tooling:** to be decided (Detox or manual scripted runs).

### 4.4 Regression tests
- **Scope:** defined set of critical paths re-run before each release.
- **Owner:** QA, executed as a Jira test plan.
- **Environment:** `staging`.
- **Reference:** release checklist in [`testing/checklists/release-qa.md`](checklists/release-qa.md) *(to be created)*.

### 4.5 Smoke tests
- **Scope:** minimal set of health checks run after deployment to any environment.
- **Owner:** deploying developer / release manager.
- **Environment:** any, including `live` (read-only checks only on live).

---

## 5. Test Suites

The following functional suites group test cases by domain. Each suite has a dedicated file in `testing/suites/`.

| Suite | File | Domain |
|---|---|---|
| POI lifecycle | [`suites/poi-lifecycle.md`](suites/poi-lifecycle.md) | POI creation, status transitions, archival |
| Task orchestration | [`suites/task-orchestration.md`](suites/task-orchestration.md) | `buffer`/`tasks` queue, worker dispatch, error handling |
| User & auth | [`suites/user-auth.md`](suites/user-auth.md) | Profile creation, roles, alert subscriptions, news roll |
| Tile rendering | [`suites/tile-rendering.md`](suites/tile-rendering.md) | Tile refresh, `_clotho` flag, POI visibility in tiles |
| Feedback pipeline | [`suites/feedback-pipeline.md`](suites/feedback-pipeline.md) | Feedback ingestion, counter updates, double-feedback prevention |

---

## 6. Test Case Format

Each test case in a suite file follows this structure:

```
### TC-<SUITE>-<NNN> — <Short title>

**Type:** unit | integration | e2e | regression | smoke
**Environment:** dev | sandbox | staging | live
**Priority:** P1 (critical) | P2 (high) | P3 (medium) | P4 (low)
**Component:** daen-scout | daen-fb-workers | fb-admin | shared
**Jira:** link or ticket reference (managed in Jira)

**Preconditions:**
- ...

**Steps:**
1. ...
2. ...

**Expected result:**
- ...
```

> **Execution tracking** (assignee, run date, pass/fail, linked bugs) lives exclusively in Jira.
> This file defines the test case specification only.

---

## 7. Priorities

Test priority follows a four-level scale aligned with Jira priority values:

| Priority | Meaning | Examples |
|---|---|---|
| **P1 — Critical** | System unusable if broken; blocks release | POI write to Firestore, Firebase Auth, task queue processing |
| **P2 — High** | Core feature degraded; must fix before release | Tile refresh after POI event, feedback counter update, push notification dispatch |
| **P3 — Medium** | Feature partially affected; fix in next sprint | `notseen` counter edge cases, alert filter options, news roll ordering |
| **P4 — Low** | Cosmetic or minor; tracked but not blocking | Deprecated field compatibility, sequence counter gap handling |

---

## 8. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Write contention on Firestore (5 writes/s limit on a document) | Test tile and counter updates with concurrent load; use `sandbox` for stress scenarios |
| RTDB task queue race conditions | Integration tests with Firebase Emulator to replay concurrent worker dispatch |
| Cross-environment data leak | Strict environment isolation enforced by `.firebaserc` project aliasing and service account scoping |
| Shared Bit component regression | Unit test `daen-objects` and `daen-firebase` before exporting a new version |
| Mobile client OTA update breaking live users | E2E regression on `staging` mandatory before any OTA push |

---

## 9. Out of Scope

The following are explicitly out of scope for this test strategy:

- Performance / load testing (not yet planned).
- Security penetration testing (covered by Firebase security rules review, separate process).
- Accessibility testing for the mobile UI (future initiative).

---

## 10. Relationship to Jira

```
daen-docs / testing/          →  WHAT to test and HOW (specifications, strategy, fixtures)
Jira                          →  WHO, WHEN, and RESULT (plans, runs, defects, coverage)
```

When a test case is created or updated in this repository, a corresponding test case should exist or be created in Jira referencing the same ID (e.g. `TC-POI-001`).
