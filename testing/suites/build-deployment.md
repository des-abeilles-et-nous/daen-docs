<!--
  Suite: Build, Deployment & Configuration  (testing/suites/build-deployment.md)
  ─────────────────────────────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests deployment mechanics, not user features
  
  These tests validate that build artifacts are correctly configured for each
  environment and that deployment pipelines (Expo EAS, Firebase, GCP) are properly
  connected without user-facing feature validation.
-->

# Suite: Build, Deployment & Configuration

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **deployment mechanics and build configuration**, not user-facing features.
> Focus: build chains, environment configuration, artifact generation, deployment pipelines.
>
> **Jira plan section:** Build, Deployment & Configuration — prefix `BUILD`
> **TC ID prefix:** `TC-BUILD`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`environments/EnvironmentsManagement.md`](../../environments/EnvironmentsManagement.md)
> - [`ECOSYSTEM_CONTEXT.md` — Deployment Architecture](../../ECOSYSTEM_CONTEXT.md)

This suite validates that the build and deployment infrastructure is correctly configured for each target environment. These are prerequisites for shipping releases.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Build & Deployment Infrastructure)
- **Expo EAS builds** — profile configuration, artefact generation, profile-specific variable injection
- **Local build chains** — React Native Metro bundler, native Android/iOS build tools
- **Environment-specific configuration** — `.env` file selection, Firebase project aliasing
- **Firebase deployment** — Firestore rules, RTDB rules, security rules deployment
- **Cloud Functions deployment** — Code bundling, dependency resolution, function registration
- **GCP configuration** — Cloud Scheduler jobs, Pub/Sub topics, IAM roles

### ❌ Out of Scope (User-Facing Functionality)
- **Feature functionality** — tested in `[AUTH]`, `[POI]`, `[NOTIF]`, `[REPORT]` suites
- **User workflows** — tested in functional suites
- **API behavior** — covered by other functional suites (e.g., POI lifecycle in `[POI]`)

---

## Test Cases

---

### TC-BUILD-001 — EAS build profile produces correct artefact per target

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, expo-build

**Preconditions:**
- `eas.json` build profiles defined for `dev` (internal) and `staging` (store or internal).
- EAS CLI authenticated and configured.
- `DAEN_TARGET` correctly injected by EAS profile system.

**Steps:**
1. Run `eas build --profile dev --platform android`. Confirm the resulting APK connects to the `dev` Firebase project.
2. Run `eas build --profile staging --platform android`. Confirm the resulting artefact connects to `dsc-staging-eu`.
3. Verify `DAEN_TARGET` is correctly injected by the EAS profile and not overridden by a local `.env`.
4. Confirm the build does not include any `live` credentials.

**Expected result:**
- Each EAS profile produces an artefact connected to the correct Firebase project.
- No credential leakage between environments in the build output.
- Build logs show correct profile and variable injection.

---

### TC-BUILD-002 — Local build produces APK with correct Firebase config

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, expo-build

**Preconditions:**
- React Native / Expo CLI installed.
- Local build tools configured (Android SDK, Xcode for iOS).
- `app.config.js` correctly reads `DAEN_TARGET` and selects target configuration.

**Steps:**
1. Set `DAEN_TARGET=dev` and run `eas build --profile dev --local`.
2. Extract the resulting APK and verify `google-services.json` contains `bsc-dev-7a548` as the Firebase project ID.
3. Repeat for `staging` with `dsc-staging-eu`.
4. Confirm no `live` credentials are embedded in either build output.

**Expected result:**
- Local builds produce APKs with correct Firebase project configuration.
- Firebase credentials match the target environment.

---

### TC-BUILD-003 — OTA update configuration per environment

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, expo-build

**Preconditions:**
- Expo Updates configured in `app.json`.
- Update channels defined for `dev` and `staging`.
- EAS credentials configured for OTA publish.

**Steps:**
1. Build and run the app for `dev`. Trigger `expo-updates` to check for updates.
2. Verify it checks against the `dev` update channel (not `staging` or `live`).
3. Publish an OTA update to the `dev` channel via `eas update --branch dev`.
4. Verify the app detects and applies the update correctly.

**Expected result:**
- OTA update channel configuration is environment-specific.
- Updates are published to the correct channel.
- Apps pull from the correct channel without cross-environment confusion.

---

### TC-BUILD-004 — Firebase rules deployment per environment

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Firebase CLI authenticated and configured with `.firebaserc` aliases.
- Firestore and RTDB rule files present (`firestore.rules`, `database.rules.json`).

**Steps:**
1. Run `firebase use dev` then `firebase deploy --only firestore:rules`.
2. Verify rules are deployed to `bsc-dev-7a548` (not another environment).
3. Repeat for `sandbox` and `staging`.
4. Query Firestore and RTDB to confirm rules are enforced (e.g., unauthenticated reads should fail).

**Expected result:**
- Each environment has its own rule deployment.
- No cross-environment rule application.
- Security rules are enforced post-deployment.

---

### TC-BUILD-005 — Cloud Functions deployment and environment variable injection

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Cloud Functions source code present in `daen-fb-workers`.
- Environment-specific `.env.<target>` files with function configuration.
- Firebase Cloud Functions runtime configured.

**Steps:**
1. Deploy Cloud Functions to `dev` environment: `firebase deploy --only functions`.
2. Invoke a test Cloud Function and verify environment variables are correctly injected (log output shows correct values).
3. Repeat for `staging`.
4. Confirm no cross-environment variable leakage.

**Expected result:**
- Cloud Functions deploy without errors.
- Environment-specific variables are correctly injected.
- Function invocations succeed with correct configuration.

---

### TC-BUILD-006 — Cloud Scheduler job configuration per environment

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Cloud Scheduler jobs defined in Terraform or manually in GCP Console.
- Job configurations point to the correct Cloud Functions per environment.
- Service accounts and IAM roles configured.

**Steps:**
1. List Cloud Scheduler jobs in `dev` GCP project: `gcloud scheduler jobs list --project=<dev-project>`.
2. Verify each scheduled job points to a Cloud Function in the `dev` project (not `staging` or `live`).
3. Manually trigger a job and verify it invokes the correct function.
4. Repeat for `staging`.

**Expected result:**
- Cloud Scheduler jobs are environment-specific.
- Jobs invoke functions in the same environment.
- Manual invocation succeeds and function executes.

---

### TC-BUILD-007 — Pub/Sub topic and subscription configuration

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Pub/Sub topics and subscriptions defined in GCP.
- Cloud Functions subscribed to topics.
- Topic names include environment identifier (e.g., `dev-poi-feedback`, `staging-poi-feedback`).

**Steps:**
1. List Pub/Sub topics in `dev` GCP project.
2. Verify topic names indicate they belong to `dev` (not `staging` or `live`).
3. Publish a test message to a `dev` topic.
4. Verify a subscribed Cloud Function receives and processes it.
5. Repeat for `staging`.

**Expected result:**
- Pub/Sub topics are environment-specific.
- Subscriptions route messages to the correct Cloud Functions.
- Functions receive and process messages correctly.

---

### TC-BUILD-008 — Dependency version consistency across repos

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P3
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- `package.json` (or equivalent) present in each repo.
- Shared dependencies documented in `ECOSYSTEM_CONTEXT.md`.
- Bit component dependencies configured.

**Steps:**
1. Extract Firebase SDK versions from each repo's `package.json` (or lock file).
2. Verify all repos use the same major Firebase SDK version (e.g., all v9.x, not mixed 8.x/9.x).
3. Check Bit scope dependencies: verify shared utility versions are consistent.
4. Attempt a build in each repo; confirm no version conflicts.

**Expected result:**
- Firebase SDK versions are consistent across repos.
- Shared dependency versions are pinned and aligned.
- Builds succeed without version conflict errors.

---

### TC-BUILD-009 — GitHub Actions CI pipeline execution

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, fb-admin, gcp, expo-build

**Preconditions:**
- GitHub Actions workflows defined in `.github/workflows/`.
- CI pipeline triggers on PR creation or commit to `develop`/`staging` branches.
- GCP credentials and Expo tokens configured in GitHub Secrets.

**Steps:**
1. Create a test PR against the repository.
2. Wait for GitHub Actions CI pipeline to execute.
3. Verify all workflow steps complete successfully (lint, test, build, deploy-to-staging).
4. Confirm deployment stage deploys to the correct environment (should be `staging` for develop, `dev` for feature branches).

**Expected result:**
- CI pipeline executes without errors.
- Deployments go to the correct environment per branch.
- Build and test artifacts are generated and archived.

---

### TC-BUILD-010 — Rollback procedure: revert a Cloud Function deployment

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** staging
**Priority:** P2
**Component:** daen-fb-workers, gcp

**Preconditions:**
- A previously working Cloud Function version exists.
- Deployment history is available in Firebase Console.
- Rollback command documented in CLAUDE.md.

**Steps:**
1. Deploy a modified version of a Cloud Function to `staging`.
2. Verify the new version is active.
3. Identify the previous version in Firebase Console.
4. Execute rollback command: `firebase deploy --only functions --force` targeting the previous version (or manual rollback in Console).
5. Verify the previous version is now active.
6. Invoke the function and confirm it behaves as before.

**Expected result:**
- Rollback procedure works without manual intervention.
- Previous version is restored and functional.
- No data loss or corruption from the rollback.
