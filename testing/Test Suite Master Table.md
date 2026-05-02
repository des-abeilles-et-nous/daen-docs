# Test Suite Master Table

> **Status: 🔶 To Be Refined**
> Columns marked 🔶 (Jira components, env scope, default scope) are inferred from architecture docs.
> Requires code inspection and Jira configuration validation before being considered stable.
> `README.md` is the source of truth — this table is a quick-reference view.

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

| # | Suite Label | TC Prefix | Jira prefix | Orientation | Suite file | Jira Epic title | Audience | Default scope 🔶 | Jira components 🔶 | Env scope 🔶 |
|---|-------------|-----------|-------------|-------------|------------|-----------------|----------|-------------------|--------------------|-------------|
| 1 | [SETUP] | `TC-SETUP` | `SETUP` | IT | `suites/setup.md` | Environment & Setup | Developers | system | `firebase` `gcp` `expo-build` | dev / sandbox / staging |
| 2 | [BUILD] | `TC-BUILD` | `IT` | IT | `suites/build-deployment.md` | Build, Deployment & Configuration | Developers, release manager | repo | `fb-workers` `gcp` `expo-build` | dev / staging |
| 3 | [REPORT] | `TC-REPORT` | `FUNC` | Functional | `suites/poi-reporting.md` | POI Reporting & Feedback | QA, beta users | repo | `beefree` `fb-workers` `firebase` | dev / staging |
| 4 | [POI] | `TC-POI` | `FUNC` | Functional | `suites/poi-lifecycle.md` | POI Lifecycle & Backend | QA, beta users | repo | `fb-workers` `firebase` | dev / sandbox |
| 5 | [NOTIF] | `TC-NOTIF` | `FUNC` | Functional | `suites/notifications.md` | Notifications & Follow-up | QA, beta users | repo / system | `beefree` `fb-workers` `expo-build` | dev / staging |
| 6 | [AUTH] | `TC-AUTH` | `FUNC` | Functional | `suites/user-auth.md` | Authentication & User Access | QA, developers | repo | `beefree` `firebase` | dev / staging |
| 7 | [WORKER] | `TC-WORKER` | `IT` | IT | `suites/task-orchestration.md` | Cloud Functions & Task Orchestration | Developers, backend QA | repo | `fb-workers` `gcp` | dev / sandbox |
| 8 | [FB] | `TC-FB` | `IT` | IT | `suites/firebase-security.md` | Firebase Security & Data Access | Developers | repo | `firebase` `fb-workers` | dev / sandbox |
| 9 | [DATA] | `TC-DATA` | `IT` | IT | `suites/data-integrity.md` | Data Integrity & Schema Validation | Developers, backend QA | repo | `firebase` `fb-workers` | dev / staging |
| 10 | [RTDB] | `TC-RTDB` | `IT` | IT | `suites/rtdb.md` | Realtime Database | Developers | repo | `firebase` `fb-workers` | dev / sandbox |
| 11 | [SHARED] | `TC-SHARED` | `IT` | IT | `suites/shared-utils.md` | Shared Utilities & Cross-repo Helpers | Developers | dev | `fb-workers` `beefree` | dev |

---

## Repo Reference

| Short name | Full repo |
|------------|-----------|
| `daen-scout` | des-abeilles-et-nous/daen-scout |
| `daen-fb-workers` | des-abeilles-et-nous/daen-fb-workers |
| `fb-admin` | des-abeilles-et-nous/fb-admin |
| `daen-docs` | des-abeilles-et-nous/daen-docs |

---

## Jira Components Reference

> Source of truth for components is Jira project `DATEST`. Do not modify this table here — update in Jira first.

| Component | Description |
|-----------|-------------|
| `beefree` | Mobile frontend (daen-scout) |
| `expo-build` | Expo EAS and local build chains |
| `fb-workers` | Backend Cloud Functions (daen-fb-workers) |
| `firebase` | Firebase tenant (Firestore, RTDB, Auth, Storage) |
| `gcp` | GCP tenant (Cloud Scheduler, Pub/Sub, logging) |

---

_Last updated: 2026-05-02 — reconciled from README.md and prior Master Table draft_
