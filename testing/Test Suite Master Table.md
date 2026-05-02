# Test Suite Master Table

> **Status: 🔶 To Be Refined**
> Component and repo assignments are inferred from architecture docs (ECOSYSTEM_CONTEXT.md, CLAUDE.md).
> Requires code inspection to validate — see functional debt issue.

---

## Legend

| Symbol | Meaning |
|--------|---------|
| 🔶 | To Be Refined |
| ✅ | Validated |
| ❌ | Not applicable |

---

## Suites

| Suite Label | Suite ID | Description | Primary Repo | Components | Env Scope | Notes |
|-------------|----------|-------------|--------------|------------|-----------|-------|
| [SETUP] | TC-SETUP | Environment bootstrap & config validation | `daen-fb-workers` | `firebase` `gcp` `expo-build` | dev / sandbox | 🔶 components to verify |
| [AUTH] | TC-AUTH | Authentication flows (Firebase Auth) | `daen-scout` | `firebase` `beefree` | dev / staging | 🔶 components to verify |
| [FB] | TC-FB | Firestore CRUD & rules | `daen-fb-workers` | `firebase` `gcp` `fb-workers` | dev / sandbox | 🔶 components to verify |
| [DATA] | TC-DATA | Data integrity & schema validation | `daen-fb-workers` | `firebase` `fb-workers` | dev / staging | 🔶 components to verify |
| [RTDB] | TC-RTDB | Realtime Database read/write/listeners | `daen-fb-workers` | `firebase` `fb-workers` | dev / sandbox | 🔶 components to verify |
| [WORKER] | TC-WORKER | Cloud Function / task worker execution | `daen-fb-workers` | `fb-workers` `gcp` | dev / sandbox | 🔶 components to verify |
| [SHARED] | TC-SHARED | Shared utilities & cross-repo helpers | `daen-fb-workers` | `fb-workers` `beefree` | dev | 🔶 components to verify |
| [POI] | TC-POI | Points of Interest domain logic | `daen-scout` / `daen-fb-workers` | `beefree` `fb-workers` `firebase` | dev / staging | 🔶 components to verify |
| [USER] | TC-USER | User profile & preferences | `daen-scout` | `beefree` `firebase` | dev / staging | 🔶 components to verify |
| [NOTIF] | TC-NOTIF | Push notification delivery (Expo) | `daen-fb-workers` | `beefree` `fb-workers` `expo-build` | dev / staging | 🔶 components to verify |
| [FEED] | TC-FEED | Feed generation & ranking | `daen-fb-workers` | `beefree` `fb-workers` | dev / staging | 🔶 components to verify |

---

## Repo Reference

| Short name | Full repo |
|------------|-----------|
| `daen-scout` | des-abeilles-et-nous/daen-scout |
| `daen-fb-workers` | des-abeilles-et-nous/daen-fb-workers |
| `fb-admin` | des-abeilles-et-nous/fb-admin |
| `daen-docs` | des-abeilles-et-nous/daen-docs |

---

_Last updated: 2026-05-02 — inferred from ECOSYSTEM_CONTEXT.md & CLAUDE.md_
