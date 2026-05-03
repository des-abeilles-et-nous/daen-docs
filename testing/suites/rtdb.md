<!--
  Suite: Realtime Database  (testing/suites/rtdb.md)
  ─────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests RTDB operations, not user features
  
  These tests validate Realtime Database read/write operations, listener behavior,
  and connection state handling without testing the user workflows that depend on RTDB.
-->

# Suite: Realtime Database

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **RTDB read/write operations and real-time listener behavior**, not user workflows.
> Focus: path structure validation, listener triggers, connection state, data persistence.
>
> **Jira plan section:** Realtime Database — prefix `RTDB`
> **TC ID prefix:** `TC-RTDB`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`data model/mdd daen-scout.md`](../../data%20model/mdd%20daen-scout.md) — RTDB structure

This suite validates that the Firebase Realtime Database (RTDB) is correctly structured and that read/write operations, listeners, and real-time synchronization work correctly.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (RTDB Operations & Listeners)
- **Path structure** — required root keys exist (`buffer`, `tasks`, `logs`, `feedbacks`, `subs`)
- **Read operations** — reading from RTDB paths returns correct data
- **Write operations** — writing to RTDB persists data correctly
- **Listener registration** — registering listeners on paths triggers callbacks
- **Real-time updates** — data updates trigger listener callbacks in real-time
- **Connection state** — listeners detect connection/disconnection
- **Offline behavior** — writes queue when offline and persist when reconnected
- **Emulator-based testing** — RTDB Emulator operations

### ❌ Out of Scope (User-Facing Workflows)
- **Task queue semantics** — semantics tested in `[WORKER]`
- **Feedback processing** — user workflows tested in `[REPORT]`
- **Subscription behavior** — subscription workflows tested in `[NOTIF]`

---

## Test Cases

---

### TC-RTDB-001 — RTDB root structure: required paths exist

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB Emulator running.
- RTDB initialized with expected root keys.

**Steps:**
1. Read the RTDB root: `db.ref("/").once("value")`.
2. Verify all expected top-level keys exist: `buffer`, `tasks`, `logs`, `feedbacks`, `subs`.
3. Verify no unexpected root-level keys that might indicate stale data.
4. For each expected path:
   - Verify it's either empty or contains valid child nodes.
   - Verify no null or corrupted values.

**Expected result:**
- All expected root paths exist.
- No unexpected paths.
- RTDB structure matches documentation.

---

### TC-RTDB-002 — Read operation: retrieve data from RTDB path

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Data written to RTDB paths.

**Steps:**
1. Write test data to `rtdb:/buffer/<uuid>`: `{"worker": "test", "at": 0, ...}`.
2. Read the data back: `db.ref("buffer/<uuid>").once("value")`.
3. Verify the returned data matches what was written.
4. Read from a non-existent path and verify it returns `null` (not an error).

**Expected result:**
- Data reads return correct values.
- Non-existent paths return `null` gracefully.

---

### TC-RTDB-003 — Write operation: persist data to RTDB

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB Emulator running.

**Steps:**
1. Write data to a new path: `db.ref("buffer/test-1").set({...})`.
2. Verify the write completes without error.
3. Read the data back and verify it persists.
4. Update the data: `db.ref("buffer/test-1").update({...})`.
5. Read again and verify the update was applied.
6. Delete the data: `db.ref("buffer/test-1").remove()`.
7. Read again and verify the data is gone (returns `null`).

**Expected result:**
- Write operations succeed and persist.
- Update operations modify data in-place.
- Delete operations remove data.

---

### TC-RTDB-004 — Listener: on value change, callback is triggered

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB Emulator running.
- Client code with listener registration available.

**Steps:**
1. Register a listener on a path: `db.ref("buffer").on("value", (snap) => {...})`.
2. Write data to the path from another client/process.
3. Verify the listener callback is triggered with the updated data.
4. Verify the callback receives the correct `DataSnapshot`.
5. Unregister the listener: `db.ref("buffer").off("value")`.
6. Write more data and verify the callback is NOT triggered (listener removed).

**Expected result:**
- Listeners trigger when data changes.
- Listeners receive correct `DataSnapshot` objects.
- Unregistering listeners stops callbacks.

---

### TC-RTDB-005 — Real-time sync: multiple clients see same data immediately

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Two RTDB clients connected (e.g., two browser tabs or two processes).
- Both have listeners on the same path.

**Steps:**
1. Client A writes data to `rtdb:/tasks/<uuid>`.
2. Verify Client B's listener is triggered within 100ms (real-time sync).
3. Verify Client B receives the same data that Client A wrote.
4. Client B modifies the data.
5. Verify Client A's listener is triggered immediately.
6. Verify both clients see the same final state.

**Expected result:**
- Data changes propagate to all connected clients in real-time.
- No data divergence between clients.

---

### TC-RTDB-006 — Connection state: listener detects online/offline transitions

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB client with `.info/connected` listener registered.

**Steps:**
1. Register a listener on `.info/connected`.
2. Verify the listener triggers with `true` (initially connected).
3. Simulate offline by stopping network (e.g., firewall rule, network simulation).
4. Verify the listener triggers with `false` (disconnected).
5. Restore network.
6. Verify the listener triggers with `true` (reconnected).

**Expected result:**
- Connection state listener detects online/offline transitions.
- Client gracefully handles disconnections.

---

### TC-RTDB-007 — Offline writes: writes queue when offline and persist on reconnect

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB client with offline persistence enabled.

**Steps:**
1. Go offline (disconnect from network).
2. Attempt a write: `db.ref("buffer/offline-test").set({...})`.
3. Verify the write completes locally (callback fires) but hasn't reached the server.
4. Go online (restore network).
5. Verify the queued write is automatically sent to the server.
6. From another client, verify the data was persisted.

**Expected result:**
- Writes queue when offline.
- Queued writes persist on reconnection.
- No data loss during offline/online transitions.

---

### TC-RTDB-008 — Transactions: multi-step atomic updates

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB client with transaction support.

**Steps:**
1. Define a transaction that reads, modifies, and writes back: `db.ref("counter").transaction((current) => current + 1)`.
2. Run the transaction 5 times concurrently from different clients.
3. Verify the counter increased by exactly 5 (no race conditions).
4. Simulate a conflict: have one transaction attempt to write to a path that another transaction is writing to.
5. Verify one transaction succeeds and the other retries.
6. Verify the final state is consistent (no partial updates).

**Expected result:**
- Transactions are atomic.
- Concurrent transactions are serialized.
- No data loss or corruption from concurrent updates.

---

### TC-RTDB-009 — Security rules enforcement: unauthenticated read denied

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB with security rules configured to deny unauthenticated access.

**Steps:**
1. Attempt to read from RTDB without authentication: `db.ref("/").once("value")`.
2. Verify the read is **REJECTED** with a permission error.
3. Authenticate with a valid token.
4. Attempt the same read.
5. Verify the read is **ALLOWED** (if the user has permission).

**Expected result:**
- Unauthenticated reads are denied.
- Authenticated reads are allowed (if rules permit).

---

### TC-RTDB-010 — Index optimization: queries on RTDB paths are fast

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P3
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Large dataset in RTDB (e.g., 1000+ task documents).

**Steps:**
1. Query tasks by status: `db.ref("tasks").orderByChild("status").equalTo("new").once("value")`.
2. Measure query time (should be < 100ms even with 1000+ documents).
3. Verify the query returns only documents matching the filter.
4. Repeat for other common queries (e.g., order by timestamp).

**Expected result:**
- Queries execute quickly even on large datasets.
- Filtering works correctly.
