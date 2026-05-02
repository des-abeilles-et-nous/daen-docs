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

### 5.2 Test case classification

Every test case within a suite is classified as one of two types. This classification is independent of the suite — a predominantly IT-focused suite can contain Functional test cases and vice versa.

| Type | Label in suite file | Jira task type | Description |
|---|---|---|---|
| **Functional** | `**Type:** functional` | `Tâche` / `Story` | Validates observable user-facing or domain behaviour: submission flows, lifecycle transitions, notification delivery, authentication. Black-box perspective. |
| **Integration Test (IT)** | `**Type:** integration` | `Tâche` / `IT` | Validates technical interactions between components: Firebase rules, Cloud Function triggers, queue mechanics, SDK boundaries. White-box / infrastructure perspective. |

> A suite file must clearly label each test case type. Suite-level orientation (Functional or IT) is indicated in the suite header and in the master table, but does not restrict the types of individual test cases within it.

### 5.3 Suite list

The eleven suites below are the authoritative specification source for all test cases.
They maintain a **1:1 correspondence with the Jira test plan Epics** — see the mapping table in [Section 10](#10-daen-docs--jira-correspondence).

| # | Suite | File | TC prefix | Jira Epic prefix | Orientation | Audience | Default scope |
|---|---|---|---|---|---|---|---|
| 1 | Environment & Setup | [`suites/setup.md`](suites/setup.md) | `TC-SETUP` | `SETUP` | IT | Developers | `system` |
| 2 | Build, Deployment & Configuration | [`suites/build-deployment.md`](suites/build-deployment.md) | `TC-BUILD` | `IT` | IT | Developers, release manager | `repo` |
| 3 | POI Reporting & Feedback | [`suites/poi-reporting.md`](suites/poi-reporting.md) | `TC-REPORT` | `FUNC` | Functional | QA, beta users | `repo` |
| 4 | POI Lifecycle & Backend | [`suites/poi-lifecycle.md`](suites/poi-lifecycle.md) | `TC-POI` | `FUNC` | Functional | QA, beta users | `repo` |
| 5 | Notifications & Follow-up | [`suites/notifications.md`](suites/notifications.md) | `TC-NOTIF` | `FUNC` | Functional | QA, beta users | `repo` / `system` |
| 6 | Authentication & User Access | [`suites/user-auth.md`](suites/user-auth.md) | `TC-AUTH` | `FUNC` | Functional | QA, developers | `repo` |
| 7 | Cloud Functions & Task Orchestration | [`suites/task-orchestration.md`](suites/task-orchestration.md) | `TC-WORKER` | `IT` | IT | Developers, backend QA | `repo` |
| 8 | Firebase Security & Data Access | [`suites/firebase-security.md`](suites/firebase-security.md) | `TC-FB` | `IT` | IT | Developers | `repo` |
| 9 | Data Integrity & Schema Validation | [`suites/data-integrity.md`](suites/data-integrity.md) | `TC-DATA` | `IT` | IT | Developers, backend QA | `repo` |
| 10 | Realtime Database | [`suites/rtdb.md`](suites/rtdb.md) | `TC-RTDB` | `IT` | IT | Developers | `repo` |
| 11 | Shared Utilities & Cross-repo Helpers | [`suites/shared-utils.md`](suites/shared-utils.md) | `TC-SHARED` | `IT` | IT | Developers | `dev` |

> **Note on tile rendering:** tile refresh logic (`tiles_view`, `_clotho` flag, POI-to-tile trigger chain) is covered as a subsection of suite 7 (Cloud Functions & Task Orchestration), consistent with its treatment as a backend technical concern in the Jira plan.

---

## 6. Test Case Format

Each test case in a suite file follows this structure:

```
### TC-<PREFIX>-<NNN> — <Short title>

**Type:** functional | integration
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

**Type values:**
- `functional` — validates observable user-facing or domain behaviour. Black-box perspective.
- `integration` — validates technical interactions between components. White-box / infrastructure perspective.

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

> 🔶 **Columns marked To Be Refined** (Jira components, env scope, default scope) are inferred from architecture docs and require validation against actual code and Jira configuration.

| daen-docs suite file | Jira Epic title | TC prefix | Jira prefix | Orientation | Jira components 🔶 | Env scope 🔶 |
|---|---|---|---|---|---|---|
| `suites/setup.md` | Environment & Setup | `TC-SETUP` | `SETUP` | IT | `firebase` `gcp` `expo-build` | dev / sandbox / staging |
| `suites/build-deployment.md` | Build, Deployment & Configuration | `TC-BUILD` | `IT` | IT | `fb-workers` `gcp` `expo-build` | dev / staging |
| `suites/poi-reporting.md` | POI Reporting & Feedback | `TC-REPORT` | `FUNC` | Functional | `beefree` `fb-workers` `firebase` | dev / staging |
| `suites/poi-lifecycle.md` | POI Lifecycle & Backend | `TC-POI` | `FUNC` | Functional | `fb-workers` `firebase` | dev / sandbox |
| `suites/notifications.md` | Notifications & Follow-up | `TC-NOTIF` | `FUNC` | Functional | `beefree` `fb-workers` `expo-build` | dev / staging |
| `suites/user-auth.md` | Authentication & User Access | `TC-AUTH` | `FUNC` | Functional | `beefree` `firebase` | dev / staging |
| `suites/task-orchestration.md` | Cloud Functions & Task Orchestration | `TC-WORKER` | `IT` | IT | `fb-workers` `gcp` | dev / sandbox |
| `suites/firebase-security.md` | Firebase Security & Data Access | `TC-FB` | `IT` | IT | `firebase` `fb-workers` | dev / sandbox |
| `suites/data-integrity.md` | Data Integrity & Schema Validation | `TC-DATA` | `IT` | IT | `firebase` `fb-workers` | dev / staging |
| `suites/rtdb.md` | Realtime Database | `TC-RTDB` | `IT` | IT | `firebase` `fb-workers` | dev / sandbox |
| `suites/shared-utils.md` | Shared Utilities & Cross-repo Helpers | `TC-SHARED` | `IT` | IT | `fb-workers` `beefree` | dev |

**Rules:**
- Each suite has a corresponding **Jira Epic** in `DATEST`. The Epic key must be referenced in the suite's `.md` file header.
- A test case defined in daen-docs (`TC-<PREFIX>-NNN`) **must** have a corresponding Jira Task/Story under that Epic.
- Execution data (run date, result, assignee, defect links) lives **only** in Jira.
- Specification data (preconditions, steps, expected result) lives **only** in suite `.md` files in daen-docs.
- If a Jira ticket has no matching `TC-*` ID in daen-docs, it must be flagged for backfill in the next documentation sprint.
- Each Jira task must carry one of the labels `scope:repo` or `scope:system` to enable filtered test plan runs.
- Each Jira task must carry one of the labels `type:functional` or `type:it` to enable type-filtered runs.
