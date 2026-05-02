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

The eight suites below are the authoritative specification source for all test cases.
They maintain a **1:1 correspondence with the Jira test plan sections** — see the mapping table in [Section 10](#10-daen-docs--jira-correspondence).

| # | Suite | File | Jira plan prefix | Domain |
|---|---|---|---|---|
| 1 | Environment & setup | [`suites/setup.md`](suites/setup.md) | `SETUP` | Environment config, credentials, Firebase project access |
| 2 | Report submission | [`suites/feedback-pipeline.md`](suites/feedback-pipeline.md) | `FUNC` | Feedback ingestion, counter updates, double-feedback prevention |
| 3 | Report review and lifecycle | [`suites/poi-lifecycle.md`](suites/poi-lifecycle.md) | `FUNC` | POI creation, status transitions, archival |
| 4 | Notifications and follow-up | [`suites/notifications.md`](suites/notifications.md) | `FUNC` | Alert subscriptions, push notifications, news roll |
| 5 | Authentication and user access | [`suites/user-auth.md`](suites/user-auth.md) | `FUNC` | Firebase Auth, user profile, roles (`isBeekeeper`, `isHunter`) |
| 6 | Cloud Functions behavior | [`suites/task-orchestration.md`](suites/task-orchestration.md) | `IT` | Worker dispatch, `buffer`/`tasks` queue, tile refresh, triggers |
| 7 | Firebase security and data access | [`suites/firebase-security.md`](suites/firebase-security.md) | `IT` | Firestore rules, RTDB rules, client vs admin SDK boundaries |
| 8 | Build, deployment and configuration | [`suites/build-deployment.md`](suites/build-deployment.md) | `IT` | `DAEN_TARGET`, `.firebaserc`, Bit components, env config |

> **Note on tile rendering:** tile refresh logic (`tiles_view`, `_clotho` flag, POI-to-tile trigger chain) is covered as a subsection of suite 6 (Cloud Functions behavior), consistent with its treatment as a backend technical concern in the Jira plan.

---

## 6. Test Case Format

Each test case in a suite file follows this structure:

```
### TC-<SUITE>-<NNN> — <Short title>

**Type:** unit | integration | e2e | regression | smoke
**Jira type:** Tâche | Story | IT
**Environment:** dev | sandbox | staging | live
**Priority:** P1 (critical) | P2 (high) | P3 (medium) | P4 (low)
**Component:** daen-scout | daen-fb-workers | fb-admin | shared
**Jira:** <ticket or test plan reference — managed in Jira>

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

## 10. daen-docs ↔ Jira Correspondence

This table is the authoritative mapping between the specification space (daen-docs) and the execution space (Jira). It must be kept in sync whenever a suite is added, renamed, or split.

| daen-docs suite file | Jira plan section | Jira prefix | Jira task types | TC ID prefix |
|---|---|---|---|---|
| `suites/setup.md` | Environment & setup | `SETUP` | Tâche | `TC-SETUP` |
| `suites/feedback-pipeline.md` | Report submission | `FUNC` | Tâche, Story | `TC-FEED` |
| `suites/poi-lifecycle.md` | Report review and lifecycle | `FUNC` | Tâche, Story | `TC-POI` |
| `suites/notifications.md` | Notifications and follow-up | `FUNC` | Tâche, Story | `TC-NOTIF` |
| `suites/user-auth.md` | Authentication and user access | `FUNC` | Tâche, Story | `TC-AUTH` |
| `suites/task-orchestration.md` | Cloud Functions behavior | `IT` | Tâche, IT | `TC-FUNC` |
| `suites/firebase-security.md` | Firebase security and data access | `IT` | Tâche, IT | `TC-SEC` |
| `suites/build-deployment.md` | Build, deployment and configuration | `IT` | Tâche, IT | `TC-BUILD` |

**Rules:**
- A test case defined in daen-docs (`TC-<PREFIX>-NNN`) **must** have a corresponding entry in the matching Jira plan section.
- Execution data (run date, result, assignee, defect links) lives **only** in Jira.
- Specification data (preconditions, steps, expected result) lives **only** in daen-docs.
- If a Jira ticket has no matching `TC-*` ID in daen-docs, it must be flagged for backfill in the next documentation sprint.
