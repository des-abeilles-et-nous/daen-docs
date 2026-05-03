# Test Suite Master Table

> **Status: ⚠️ WORK IN PROGRESS — All 11 Suite Files Authored (under review) + Jira Epics Complete**
> - **TC-SETUP:** ✅ Validated against source — components: firebase, gcp, expo-build, sentry
> - **TC-BUILD through TC-SHARED:** ✅ All 10 suite files authored with clear **IT vs Functional** distinction
>   - **IT Suites (🔧):** TC-BUILD, TC-WORKER, TC-FB, TC-DATA, TC-RTDB, TC-SHARED — infrastructure & technical focus
>   - **Functional Suites (👤):** TC-AUTH, TC-REPORT, TC-POI, TC-NOTIF — user-facing workflows & domain behavior
> - **Jira alignment:** All 11 suites have corresponding Epics (DATEST-1 through DATEST-11)
> - **Next work:** Implement test cases in Jira under each Epic; verify component/repo columns against source code once test case implementation reveals actual dependencies
> - Execution model: manual test case runs + manual campaign tracking in Jira (no automated E2E tooling)
> - See README.md Section 10 for Jira correspondence rules and implementation checklist
> 
> `README.md` is the source of truth — this table is a quick-reference view. Jira Epic list: https://desabeillesetnous.atlassian.net/jira/software/c/projects/DATEST/issues

---

## Legend

| Symbol | Meaning |
|--------|---------|
| 🔶 | To Be Refined |
| ✅ | Validated |
| ❌ | Not applicable |
| `FUNC` | Functional orientation — user-facing / domain behaviour |
| `IT` | Integration Test orientation — technical / infrastructure |

---

## Suites

| # | Suite Label | TC Prefix | Jira prefix | Orientation | Suite file | Jira Epic title | Audience | Default scope 🔶 | Repos 🔶 | Jira components | Env scope 🔶 |
|---|-------------|-----------|-------------|-------------|------------|-----------------|----------|-------------------|----------|-------------------|-------------|
| 1 | [SETUP] | `TC-SETUP` | `SETUP` | IT | `suites/setup.md` | Environment & Setup | Developers | system | `daen-scout` `daen-fb-workers` `fb-admin` | ✅ `firebase` `gcp` `expo-build` `sentry` | dev / sandbox / staging |
| 2 | [BUILD] | `TC-BUILD` | `IT` | IT | `suites/build-deployment.md` | Build, Deployment & Configuration | Developers, release manager | repo | `daen-scout` `daen-fb-workers` | `expo-build` `fb-workers` `gcp` | dev / staging |
| 3 | [REPORT] | `TC-REPORT` | `FUNC` | Functional | `suites/poi-reporting.md` | POI Reporting & Feedback | QA, beta users | repo | `daen-scout` `daen-fb-workers` | `beefree` `fb-workers` `firebase` | dev / sandbox / staging |
| 4 | [POI] | `TC-POI` | `FUNC` | Functional | `suites/poi-lifecycle.md` | POI Lifecycle & Backend | QA, beta users | repo | `daen-fb-workers` `fb-admin` | `fb-workers` `firebase` | dev / sandbox |
| 5 | [NOTIF] | `TC-NOTIF` | `FUNC` | Functional | `suites/notifications.md` | Notifications & Follow-up | QA, beta users | repo / system | `daen-scout` `daen-fb-workers` | `beefree` `fb-workers` `expo-build` | dev / staging |
| 6 | [AUTH] | `TC-AUTH` | `FUNC` | Functional | `suites/user-auth.md` | Authentication & User Sessions | QA, developers | repo | `daen-scout` `daen-fb-workers` | `beefree` `firebase` `fb-workers` | dev / staging |
| 7 | [WORKER] | `TC-WORKER` | `IT` | IT | `suites/task-orchestration.md` | Cloud Functions & Task Orchestration | Developers, backend QA | repo | `daen-fb-workers` | `fb-workers` `gcp` | dev / sandbox |
| 8 | [FB] | `TC-FB` | `IT` | IT | `suites/firebase-security.md` | Firebase Security & Data Access | Developers | repo | `daen-scout` `daen-fb-workers` | `firebase` `fb-workers` | dev / sandbox |
| 9 | [DATA] | `TC-DATA` | `IT` | IT | `suites/data-integrity.md` | Data Integrity & Schema Validation | Developers, backend QA | repo | `daen-fb-workers` `fb-admin` `shared` | `firebase` `fb-workers` | dev / staging |
| 10 | [RTDB] | `TC-RTDB` | `IT` | IT | `suites/rtdb.md` | Realtime Database | Developers | repo | `daen-scout` `daen-fb-workers` | `firebase` `fb-workers` `beefree` | dev / sandbox |
| 11 | [SHARED] | `TC-SHARED` | `IT` | IT | `suites/shared-utils.md` | Shared Utilities & Cross-repo Helpers | Developers | repo | `daen-scout` `daen-fb-workers` `fb-admin` | `beefree` `fb-workers` | dev |

---

## Repo Reference

| Short name | Full repo | Jira component |
|------------|-----------|----------------|
| `daen-scout` | des-abeilles-et-nous/daen-scout | `beefree` |
| `daen-fb-workers` | des-abeilles-et-nous/daen-fb-workers | `fb-workers` |
| `fb-admin` | des-abeilles-et-nous/fb-admin | _(no Jira component — covered under `firebase` or `fb-workers` depending on context)_ |
| `shared` | Bit scope `desabeillesetnous.daen-js-shared` | _(no dedicated Jira component)_ |
| `daen-docs` | des-abeilles-et-nous/daen-docs | _(documentation only, not a test target)_ |

---

## Jira Components Reference

> Source of truth for components is Jira project `DATEST`. Do not modify this table here — update in Jira first.

| Component | Description | Repo |
|-----------|-------------|------|
| `beefree` | Mobile frontend (daen-scout) | `daen-scout` |
| `expo-build` | Expo EAS and local build chains | `daen-scout` |
| `fb-workers` | Backend Cloud Functions (daen-fb-workers) | `daen-fb-workers` |
| `firebase` | Firebase tenant (Firestore, RTDB, Auth, Storage) | cross-repo |
| `gcp` | GCP tenant (Cloud Scheduler, Pub/Sub, logging) | `daen-fb-workers` |

> **Note:** `fb-admin` and `shared` (Bit components) have no dedicated Jira component.
> `fb-admin` test cases should be filed under `firebase` (for Firestore/RTDB operations) or `fb-workers` (for data consistency with backend).
> `shared` component test cases should be filed under the repo that authors the component (`beefree` for `daen-utils`/`looped-carousel`/`snackbar`; `fb-workers` for `daen-firebase`/`daen-objects`).

---

_Last updated: 2026-05-03 — All 11 suite files authored (TC-SETUP through TC-SHARED) with clear IT vs Functional distinction. All 11 Jira Epics created (DATEST-1 through DATEST-11). TC-SETUP components validated from source (firebase, gcp, expo-build, sentry). Remaining suite components are inferred from architecture docs and will be validated once test cases are implemented in Jira and their actual dependencies are revealed. Next: implement test cases in each Epic (1-3 test cases per suite), then create Jira campaigns and validate component/repo mappings against actual source code coverage._
