<!--
  DAEN-SCOUT — Test Strategy  (testing/README.md)
  ─────────────────────────────────────────────────
  PURPOSE : Master reference for what to test, how to approach it, and where
            each testing activity belongs across the four-repo DAEN-SCOUT
            ecosystem.

  SCOPE MODEL
  Each test case carries a Scope field:
    • repo   — targets a single repository in isolation; used for feature
               development and per-repo non-regression (PR gates, sprint QA).
    • system — requires the full ecosystem running together; reserved for
               large environment changes, cross-repo integration passes, and
               release gates.
  The default execution mode is repo-scoped. System-scoped runs are opt-in.

  GENERATED TEST CASES
  Suite files (.md) only contain test cases whose Component is daen-fb-workers
  (scope:repo) or cross-repo cases explicitly tagged scope:system.
  Test cases for daen-scout, fb-admin, and shared libs are authored separately.

  EXECUTION TRACKING
  Specification lives here. Execution (run date, result, assignee, defects)
  lives exclusively in Jira — see Section 10 for the correspondence table.
-->

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

### 5.1 Suite model

A **suite** is a scoped collection of test cases targeting a specific domain, designed to be executed by a **targeted group of users** (developers, QA, beta users, release managers, etc.).

Key principles:

- **Suites may overlap.** A test case can appear in more than one suite if it is relevant to different audiences or assessment contexts. Overlap is intentional and acceptable.
- **Coverage is defined by union.** What matters is that the union of all suites covers the entire test universe — every meaningful behaviour of the ecosystem must be reachable through at least one suite.
- **Suites are not mutually exclusive.** They are audience-oriented views over the test space, not partitions of it.
- **The current suite list is fixed** — do not add, rename, or split suites without updating Section 10 and the corresponding Jira Epics.
- **Scope is a run-time selector, not a suite property.** Every suite contains a mix of `repo` and `system` test cases. A `repo` run filters by `Component: daen-fb-workers` + `Scope: repo`. A `system` run takes all test cases tagged `Scope: system`.

### 5.2 Suite list

The eight suites below are the authoritative specification source for all test cases.
They maintain a **1:1 correspondence with the Jira test plan Epics** — see the mapping table in [Section 10](#10-daen-docs--jira-correspondence).

| # | Suite | File | Jira Epic prefix | Audience | Domain | Default scope | daen-fb-workers coverage |
|---|---|---|---|---|---|---|---|
| 1 | Environment & setup | [`suites/setup.md`](suites/setup.md) | `SETUP` | Developers | Environment config, credentials, Firebase project access | `system` | Partial — TC-SETUP-001, 004, 005 are `repo` |
| 2 | Report submission | [`suites/feedback-pipeline.md`](suites/feedback-pipeline.md) | `FUNC` | QA, beta users | Feedback ingestion, counter updates, double-feedback prevention | `repo` | Primary — backend pipeline logic |
| 3 | Report review and lifecycle | [`suites/poi-lifecycle.md`](suites/poi-lifecycle.md) | `FUNC` | QA, beta users | POI creation, status transitions, archival | `repo` | Primary — POI write and trigger chain |
| 4 | Notifications and follow-up | [`suites/notifications.md`](suites/notifications.md) | `FUNC` | QA, beta users | Alert subscriptions, push notifications, news roll | `repo` / `system` | Partial — dispatch logic is `repo`; FCM delivery requires `system` |
| 5 | Authentication and user access | [`suites/user-auth.md`](suites/user-auth.md) | `FUNC` | QA, developers | Firebase Auth, user profile, roles (`isBeekeeper`, `isHunter`) | `repo` | Partial — Firestore user record and role flags |
| 6 | Cloud Functions behavior | [`suites/task-orchestration.md`](suites/task-orchestration.md) | `IT` | Developers, backend QA | Worker dispatch, `buffer`/`tasks` queue, tile refresh, triggers | `repo` | Full — all test cases target `daen-fb-workers` |
| 7 | Firebase security and data access | [`suites/firebase-security.md`](suites/firebase-security.md) | `IT` | Developers | Firestore rules, RTDB rules, client vs admin SDK boundaries | `repo` | Full — rules files live in `daen-fb-workers` |
| 8 | Build, deployment and configuration | [`suites/build-deployment.md`](suites/build-deployment.md) | `IT` | Developers, release manager | `DAEN_TARGET`, `.firebaserc`, Bit components, env config | `repo` | Partial — `.firebaserc`, `firebase.json`, `functions/` build |

> **Note on tile rendering:** tile refresh logic (`tiles_view`, `_clotho` flag, POI-to-tile trigger chain) is covered as a subsection of suite 6 (Cloud Functions behavior), consistent with its treatment as a backend technical concern in the Jira plan.

---

## 6. Test Case Format

Each test case in a suite file follows this structure:

```
### TC-<SUITE>-<NNN> — <Short title>

**Type:** unit | integration | e2e | regression | smoke
**Scope:** repo | system
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

**Scope values:**
- `repo` — the test case can be executed against a single repository in isolation. Use for feature development, per-repo non-regression, and PR gates.
- `system` — the test case requires the full ecosystem (multiple repos, live Firebase services, or a deployed environment). Use for large environment changes, cross-repo integration passes, and release gates.

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

| daen-docs suite file | Jira Epic | Jira prefix | Jira task types | TC ID prefix |
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
- Each suite has a corresponding **Jira Epic** in `DATEST`. The Epic key must be referenced in the suite's `.md` file header.
- A test case defined in daen-docs (`TC-<PREFIX>-NNN`) **must** have a corresponding Jira Task/Story under that Epic.
- Execution data (run date, result, assignee, defect links) lives **only** in Jira.
- Specification data (preconditions, steps, expected result) lives **only** in Jira tickets — suite `.md` files in daen-docs contain scope, audience, and Epic link only.
- If a Jira ticket has no matching `TC-*` ID in daen-docs, it must be flagged for backfill in the next documentation sprint.
- Each Jira task must carry one of the labels `scope:repo` or `scope:system` to enable filtered test plan runs. All `daen-fb-workers`-only test cases default to `scope:repo`.
