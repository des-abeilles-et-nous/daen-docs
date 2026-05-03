<!--
  Suite: Firebase Security & Data Access  (testing/suites/firebase-security.md)
  ─────────────────────────────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests security rules, not user features
  
  These tests validate that Firestore CRUD operations and security rules
  are correctly configured and enforced, without testing user-facing workflows.
-->

# Suite: Firebase Security & Data Access

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **Firestore CRUD operations and security rules enforcement**, not user workflows.
> Focus: read/write permissions, field validation rules, collection access control, rule evaluation.
>
> **Jira plan section:** Firebase Security & Data Access — prefix `FB`
> **TC ID prefix:** `TC-FB`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`environments/EnvironmentsManagement.md`](../../environments/EnvironmentsManagement.md)
> - [`data model/`](../../data%20model/) — all collections documented

This suite validates that Firestore security rules are correctly configured and enforced, preventing unauthorized access while allowing legitimate operations.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Security Rules & CRUD Operations)
- **Collection-level access** — read/write permissions per collection (POIs, users, tiles, etc.)
- **Document-level access** — rules restrict access to specific documents (e.g., user can only read their own profile)
- **Field-level rules** — rules restrict write access to specific fields (e.g., `creator_id` is read-only after creation)
- **Subcollection access** — rules for nested collections
- **CRUD validation** — create, read, update, delete operations respect security rules
- **Unauthenticated access** — requests without valid auth token are denied
- **Authenticated access** — valid tokens are accepted and scoped correctly
- **Admin SDK access** — admin operations bypass rules for testing
- **Emulator-based rule testing** — rule verification in Cloud Functions Emulator

### ❌ Out of Scope (User-Facing Workflows)
- **User workflows** — tested in `[AUTH]`, `[POI]`, `[NOTIF]`, `[REPORT]` suites
- **Business logic** — tested in functional suites
- **Data consistency** — some aspects tested in `[DATA]`, but business consistency in functional suites

---

## Test Cases

---

### TC-FB-001 — POIs collection: read access allowed, write restricted to admin

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Firestore Emulator running.
- At least one POI document in `POIs` collection.
- Firestore rules loaded in emulator.

**Steps:**
1. **Unauthenticated read**: Query `POIs` collection without auth token. Verify the read is **DENIED**.
2. **Authenticated read**: Query `POIs` with a valid user token. Verify the read is **ALLOWED** and returns POI data.
3. **Authenticated write (user)**: Attempt to create/update a POI as a non-admin user. Verify the write is **DENIED**.
4. **Admin write**: Use Firebase Admin SDK to create/update a POI. Verify the write is **ALLOWED**.
5. **Admin read**: Query `POIs` with Admin SDK. Verify all documents are readable.

**Expected result:**
- Unauthenticated users cannot read POIs.
- Authenticated users can read POIs.
- Non-admin users cannot create/modify POIs.
- Admin SDK can perform all CRUD operations.

---

### TC-FB-002 — users collection: each user can read/write only their own profile

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Two test users created (user A and user B).
- Both users logged in and have valid ID tokens.

**Steps:**
1. **User A reads own profile**: Query `users/{userA_id}` with user A's token. Verify read is **ALLOWED**.
2. **User A reads user B's profile**: Query `users/{userB_id}` with user A's token. Verify read is **ALLOWED** (public profile) or **DENIED** (private).
3. **User A writes own profile**: Update `users/{userA_id}/display_name` with user A's token. Verify write is **ALLOWED**.
4. **User A writes user B's profile**: Attempt to update `users/{userB_id}/display_name` with user A's token. Verify write is **DENIED**.
5. **User A attempts to change user B's `uid` field**: Attempt write. Verify write is **DENIED** (immutable field).

**Expected result:**
- Users can read their own profile.
- Users cannot modify other users' profiles.
- Immutable fields (`uid`, `created_t`) cannot be modified after creation.

---

### TC-FB-003 — Field validation: POI fields cannot be modified after creation (immutable fields)

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- A POI document exists in Firestore.

**Steps:**
1. Attempt to modify the immutable field `uid` of an existing POI. Verify the write is **DENIED**.
2. Attempt to modify the immutable field `created_t`. Verify the write is **DENIED**.
3. Attempt to modify a mutable field (e.g., `status`). Verify the write is **ALLOWED**.
4. Verify the immutable field rejection includes a clear error message: "Immutable field cannot be modified."

**Expected result:**
- Immutable fields cannot be modified.
- Mutable fields can be modified.
- Error messages are clear.

---

### TC-FB-004 — Firestore indexes: required indexes exist and queries succeed

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Firestore indexes defined in `firestore.indexes.json`.
- Firebase emulator configured to apply indexes.

**Steps:**
1. Run `firebase firestore:indexes` and verify all expected indexes are listed.
2. Execute a composite query (e.g., `POIs where status == 'handled' and created_t > <timestamp>`).
3. Verify the query succeeds (index exists, no "needs index" error).
4. Repeat for other documented composite queries (e.g., users query, tiles query).

**Expected result:**
- All required indexes exist.
- Composite queries execute without "needs index" errors.
- Query results are correct.

---

### TC-FB-005 — Collection quotas: Firestore write limit respected (per-document)

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- A document that will be updated multiple times rapidly.
- Firestore quota simulator or actual quota enforcement enabled.

**Steps:**
1. Attempt to write to the same document 6 times in 1 second (exceeds ~5 writes/sec/doc limit).
2. Verify the 6th write is **REJECTED** with a quota error (429 or similar).
3. Wait 1 second and retry the 6th write.
4. Verify the retry succeeds.
5. Confirm that distributed writes to **different** documents succeed without quota issues.

**Expected result:**
- Per-document write quota is enforced.
- Rapid writes to the same document are throttled.
- Writes to different documents are not affected.
- Retry after backoff succeeds.

---

### TC-FB-006 — Firestore transactions: atomic multi-document writes

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- A Cloud Function using `db.runTransaction()` exists.

**Steps:**
1. Call the Cloud Function with test data for a multi-document transaction.
2. Verify the transaction either completely succeeds (all writes applied) or completely fails (no partial writes).
3. Simulate a conflict: have another process attempt to write to one of the transaction's documents mid-transaction.
4. Verify the transaction either retries or fails cleanly (no data corruption).

**Expected result:**
- Transactions are atomic (all or nothing).
- Conflicts are handled gracefully (retry or fail).
- No partial writes in case of failure.

---

### TC-FB-007 — Subcollections: access control inherited from parent

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- A subcollection exists (e.g., `users/{uid}/subscriptions`).
- Firestore rules define access control on subcollections.

**Steps:**
1. **User A reads own subscriptions**: Query `users/{userA_id}/subscriptions` with user A's token. Verify read is **ALLOWED**.
2. **User A reads user B's subscriptions**: Query `users/{userB_id}/subscriptions` with user A's token. Verify read is **DENIED**.
3. **User A writes to own subscriptions**: Create a subscription doc in own subcollection. Verify write is **ALLOWED**.
4. **User A writes to user B's subscriptions**: Attempt to create a subscription in user B's subcollection. Verify write is **DENIED**.

**Expected result:**
- Subcollection access control is enforced.
- Users can only access their own subcollections.
- Cross-user access is denied.

---

### TC-FB-008 — POIs_attic collection: archived POIs accessible but not modifiable by normal users

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Archived POIs exist in `POIs_attic` collection.

**Steps:**
1. **Authenticated user reads archived POI**: Query `POIs_attic` with a user token. Verify read is **ALLOWED** (public history).
2. **Authenticated user modifies archived POI**: Attempt to update a document in `POIs_attic`. Verify write is **DENIED**.
3. **Admin modifies archived POI**: Use Admin SDK to update `POIs_attic`. Verify write is **ALLOWED**.

**Expected result:**
- Archived POIs are readable by all authenticated users.
- Archived POIs cannot be modified by non-admin users.
- Admin SDK can modify archived POIs.

---

### TC-FB-009 — Deprecated collections blocked (poi_tile, tile_ids)

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Deprecated collections `poi_tile` and `tile_ids` may exist but are marked for deletion.
- Firestore rules either deny access or are silent on deprecated collections.

**Steps:**
1. Attempt to query `poi_tile` collection. Verify read is **DENIED** or returns empty.
2. Attempt to query `tile_ids` collection. Verify read is **DENIED** or returns empty.
3. Confirm no new code writes to these collections (checked via code review).

**Expected result:**
- Deprecated collections are not accessible.
- No data leakage from deprecated collections.
- New code does not reference deprecated collections.

---

### TC-FB-010 — Custom claims: user roles enforced in security rules

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Firebase custom claims configured for user roles (e.g., `admin: true`, `moderator: true`).
- Security rules check custom claims (e.g., `request.auth.token.admin == true`).

**Steps:**
1. Create a normal user (no custom claims).
2. Create an admin user (custom claim: `admin: true`).
3. **Normal user attempts admin operation**: Query an admin-only collection with normal user token. Verify read is **DENIED**.
4. **Admin performs admin operation**: Query admin-only collection with admin token. Verify read is **ALLOWED**.
5. Update custom claims on a user and verify the change takes effect immediately (or requires token refresh).

**Expected result:**
- Custom claims are checked in security rules.
- Role-based access control works correctly.
- Users without required claims are denied access.
