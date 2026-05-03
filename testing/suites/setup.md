<!--
  Suite: Environment & Setup  (testing/suites/setup.md)
  ───────────────────────────────────────────────────────
  PURPOSE : Validates that each environment is correctly configured and that
            all components of the ecosystem can reach their expected Firebase
            project and third-party services.

  GENERATED TEST CASES
  Only test cases whose Component is daen-fb-workers are tagged scope:repo.
  Test cases for daen-scout, fb-admin, and shared libs are tagged scope:system
  and will be authored in their respective repo test passes.

  Scope key:
    repo   — runs against daen-fb-workers in isolation (PR gate, sprint QA).
    system — requires the full ecosystem / multiple repos.
-->

# Suite: Environment & Setup

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
> 
> **Jira plan section:** Environment & setup — prefix `SETUP`
> **TC ID prefix:** `TC-SETUP`
> **Jira task type:** Tâche
> **Reference docs:**
> - [`environments/EnvironmentsManagement.md`](../../environments/EnvironmentsManagement.md)
> - [`ECOSYSTEM_CONTEXT.md` — Shared Development Environments](../../ECOSYSTEM_CONTEXT.md#shared-development-environments)

This suite validates that each environment is correctly configured and that all components of the ecosystem can reach their expected Firebase project and third-party services. These tests are prerequisites for all other suites.

---

## Environments and sub-systems in scope

| Environment | Firebase project | In scope for this suite |
|---|---|---|
| `dev` | `bsc-dev-7a548` | ✅ Primary target for setup validation |
| `sandbox` | `bsc-sandbox-9fbeb` | ✅ Secondary target |
| `staging` | `dsc-staging-eu` | ✅ Pre-release gate |
| `live` | `dsc-live-eu` | ⚠️ Read-only checks only — no write operations |

Third-party sub-systems validated in this suite:
- **Firebase** (Firestore, RTDB, Auth, Cloud Functions, Cloud Storage)
- **Facebook Auth**
- **Google Auth / Google Maps**
- **Sentry** (error tracking)
- **Expo / EAS** (build and OTA delivery)

---

## Test cases

---

### TC-SETUP-001 — Firebase project reachability per environment

**Type:** integration
**Scope:** repo
**Jira type:** Tâche
**Environment:** dev, sandbox, staging, live
**Priority:** P1
**Component:** daen-fb-workers

**Preconditions:**
- Firebase CLI installed and authenticated.
- `.firebaserc` correctly defines aliases `dev`, `sandbox`, `staging`, `live`.

**Steps:**
1. Run `firebase use dev` then `firebase projects:list` and verify `bsc-dev-7a548` is selected.
2. Repeat for `sandbox` (`bsc-sandbox-9fbeb`), `staging` (`dsc-staging-eu`), `live` (`dsc-live-eu`).
3. For each environment, run `firebase firestore:indexes` to confirm Firestore is reachable.

**Expected result:**
- Each `firebase use <env>` selects the correct Firebase project without error.
- Firestore responds to index listing for each project.
- No cross-environment project confusion.

---

### TC-SETUP-002 — daen-scout environment variable selection (`DAEN_TARGET`)

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev, staging, live
**Priority:** P1
**Component:** daen-scout

**Preconditions:**
- `.env_dev`, `.env_staging`, `.env_live` files present (based on `.env.dist`).
- `app.config.js` reads `DAEN_TARGET` to select the target configuration.

**Steps:**
1. Set `DAEN_TARGET=dev` and start the app. Verify it connects to Firebase project `bsc-dev-7a548`.
2. Set `DAEN_TARGET=staging` and start the app. Verify it connects to `dsc-staging-eu`.
3. Set `DAEN_TARGET=live` and start the app. Verify it connects to `dsc-live-eu`.
4. Confirm that `DAEN_TARGET`-prefixed env variables override their non-prefixed counterparts (`process.env.$DAEN_TARGET_VARIABLE ?? process.env.VARIABLE`).

**Expected result:**
- Each `DAEN_TARGET` value loads the correct Firebase config and API credentials.
- No live credentials are loaded when targeting `dev` or `staging`.

---

### TC-SETUP-003 — fb-admin service account key selection

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** fb-admin

**Preconditions:**
- Service account key files present at `keys/{project-id}-admin.json` (gitignored).
- At least one key available for each of `dev`, `sandbox`, `staging`.

**Steps:**
1. Launch fb-admin CLI interactive menu.
2. Select the `dev` service account key. Verify the Admin SDK initialises against `bsc-dev-7a548`.
3. Repeat for `sandbox` and `staging`.
4. Confirm that no `live` service account key is selectable in a non-production context (access control check).

**Expected result:**
- Admin SDK initialises correctly for each selected key.
- Project ID reported by the SDK matches the expected Firebase project for each environment.

---

### TC-SETUP-004 — Firestore collections presence

**Type:** integration
**Scope:** repo
**Jira type:** Tâche
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers

**Preconditions:**
- Firebase project targeted via CLI for the tested environment.
- Seed data present (at minimum one document per collection).

**Steps:**
1. Query Firestore for the presence of collections: `POIs`, `POIs_attic`, `tiles_view`, `users`, `sequences`.
2. Confirm each collection returns at least one document or an empty result (not a permission error).
3. Verify deprecated collections (`poi_tile`, `tile_ids`) are absent or empty.

**Expected result:**
- All expected collections are accessible.
- No permission errors on read for admin-authenticated calls.
- Deprecated collections do not contain active data.

---

### TC-SETUP-005 — Realtime Database paths presence

**Type:** integration
**Scope:** repo
**Jira type:** Tâche
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers

**Preconditions:**
- Firebase project targeted via CLI.
- RTDB initialised with at minimum an empty JSON tree.

**Steps:**
1. Read RTDB root and confirm the following top-level keys exist (may be empty): `buffer`, `tasks`, `logs`, `feedbacks`, `subs`.
2. Confirm no unexpected top-level keys are present that could indicate environment data leakage.
3. Verify read access succeeds with admin credentials and that client SDK read access respects RTDB rules.

**Expected result:**
- Expected RTDB paths are present.
- No cross-environment data leakage at root level.
- RTDB rules deny unauthenticated client access.

---

### TC-SETUP-006 — Facebook Auth configuration per environment

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev, live
**Priority:** P2
**Component:** daen-scout

**Preconditions:**
- Facebook app credentials (`FACEBOOK_APPID`, `FACEBOOK_CLIENT_TOKEN`, `FACEBOOK_DISPLAYNAME`, `FACEBOOK_SCHEME`) set in the target `.env` file.
- Firebase console has Facebook Auth provider configured for the target project.

**Steps:**
1. Build the app for `dev` target. Initiate Facebook login flow.
2. Confirm the Facebook app used is the `DA&N (dev)` app (not the production app).
3. Repeat for `live` target. Confirm the app uses `da&n - observ'acteurs` Facebook app.
4. Verify that no special characters are present in `app.json` Facebook config (known build failure risk).

**Expected result:**
- Each environment uses the correct Facebook app identity.
- Login flow completes without error.
- No cross-environment Facebook app confusion.

---

### TC-SETUP-007 — Google Auth and Maps API credentials

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout

**Preconditions:**
- `buildconfig/<target>/google-services.json` (Android) and `buildconfig/<target>/GoogleService-Info.plist` (iOS) present for the target.
- Google Maps SDK API key configured in the target environment.
- SHA-1 fingerprints of authorized APKs registered in the Google Cloud Console.

**Steps:**
1. Build the app for the target environment.
2. Confirm Google Sign-In completes successfully using the correct OAuth client for the target.
3. Load a map screen and confirm the Google Maps tile loads without API key error.
4. Verify Firebase Authentication accepts the Google credential and creates/matches a user session.

**Expected result:**
- Google Auth flow succeeds end-to-end.
- Map tiles load without API errors.
- Firebase Auth session is created correctly.

---

### TC-SETUP-008 — Sentry error tracking connectivity

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev
**Priority:** P3
**Component:** daen-scout

**Preconditions:**
- `SENTRY_DSN`, `SENTRY_ORG`, `SENTRY_PROJECT`, `SENTRY_AUTH` set in the target `.env`.
- Sentry project `#daen/daen-scout` exists and is accessible.

**Steps:**
1. Build and launch the app in `dev` environment.
2. Trigger a deliberate non-fatal error in the app.
3. Confirm the error appears in the Sentry dashboard under the correct project and environment tag.
4. Verify source maps allow the stack trace to resolve to the correct source file and line.

**Expected result:**
- Error is captured and visible in Sentry within 60 seconds.
- Environment tag matches `dev`.
- Stack trace is human-readable (source maps applied).

---

### TC-SETUP-009 — EAS build profile produces correct artefact per target

**Type:** integration
**Scope:** system
**Jira type:** Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout

**Preconditions:**
- `eas.json` build profiles defined for `dev` (internal) and `staging` (store or internal).
- EAS CLI authenticated.

**Steps:**
1. Run `eas build --profile dev --platform android`. Confirm the resulting APK connects to the `dev` Firebase project.
2. Run `eas build --profile staging --platform android`. Confirm the resulting artefact connects to `dsc-staging-eu`.
3. Verify `DAEN_TARGET` is correctly injected by the EAS profile and not overridden by a local `.env`.
4. Confirm the build does not include any `live` credentials.

**Expected result:**
- Each EAS profile produces an artefact connected to the correct Firebase project.
- No credential leakage between environments in the build output.
