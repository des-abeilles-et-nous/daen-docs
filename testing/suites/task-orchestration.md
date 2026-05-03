<!--
  Suite: Cloud Functions & Task Orchestration  (testing/suites/task-orchestration.md)
  ──────────────────────────────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests worker system mechanics, not user features
  
  These tests validate that Cloud Functions, task queues, and worker orchestration
  systems function correctly without testing the user-facing POI or feedback workflows
  that depend on them.
-->

# Suite: Cloud Functions & Task Orchestration

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **worker execution mechanics and task orchestration**, not user workflows.
> Focus: Cloud Function triggers, task queue execution, retry logic, error handling, per-tile serialization.
>
> **Jira plan section:** Cloud Functions & Task Orchestration — prefix `WORKER`
> **TC ID prefix:** `TC-WORKER`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`backend/Worker System.md`](../../backend/Worker%20System.md)
> - [`data model/POI lifecycle.md`](../../data%20model/POI%20lifecycle.md)

This suite validates that the worker system (Pantheon workers + task queue) correctly orchestrates background tasks, respects the per-tile write serialization invariant, and handles errors and retries.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Worker Mechanics & Orchestration)
- **Cloud Function triggers** — HTTP triggers, Pub/Sub triggers, Firestore triggers, RTDB triggers
- **Task queue execution** — buffer (immediate), tasks (scheduled), polling cycle, de-duplication
- **Per-tile write serialization** — worker queue ensures POI updates to a tile are serialized, not raced
- **Retry logic** — failed tasks are re-queued, exponential backoff
- **Error handling** — function exceptions are caught, logged, moved to error queue
- **Worker lifecycle** — Zeus watchdog restarts dead workers, Hermes/Lachesis/Hera self-schedule next runs
- **Emulator-based testing** — Cloud Functions Emulator, RTDB emulator, Firestore emulator

### ❌ Out of Scope (User-Facing Workflows)
- **POI feedback processing end-to-end** — tested in `[REPORT]` (functional)
- **Notification delivery** — tested in `[NOTIF]` (functional)
- **POI lifecycle** — tested in `[POI]` (functional)

---

## Test Cases

---

### TC-WORKER-001 — HTTP trigger: Cloud Function invocation and response

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Cloud Functions Emulator running locally (`firebase emulators:start`).
- At least one HTTP-triggered function deployed (e.g., `onHealthCheck`, `onTestHook`).

**Steps:**
1. Invoke an HTTP-triggered function via `curl` to `http://localhost:5001/<project>/us-central1/<functionName>`.
2. Verify the function responds with a 200 status code.
3. Check function logs to confirm execution.
4. Invoke with invalid parameters and verify error handling (e.g., 400 Bad Request).

**Expected result:**
- HTTP trigger invokes the function correctly.
- Response code and body are as expected.
- Function logs show execution details.

---

### TC-WORKER-002 — Pub/Sub trigger: Cloud Function invoked from message

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Cloud Functions Emulator running.
- Pub/Sub emulator running.
- A Cloud Function subscribed to a Pub/Sub topic (e.g., `onPOIFeedbackPublished`).

**Steps:**
1. Publish a test message to the Pub/Sub topic via `gcloud pubsub topics publish <topic> --message="test"`.
2. Wait 2 seconds for the message to be processed.
3. Verify the Cloud Function executed by checking function logs.
4. Confirm the function processed the message payload correctly.

**Expected result:**
- Pub/Sub message triggers the Cloud Function.
- Function logs show message reception and processing.
- Message payload is correctly parsed.

---

### TC-WORKER-003 — Firestore trigger: function invoked on document write

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Cloud Functions Emulator running.
- Firestore emulator running.
- A Firestore trigger function deployed (e.g., `onPOICreated`).

**Steps:**
1. Write a test document to Firestore via the emulator: `firebase firestore:set <collection>/<doc> <data>`.
2. Wait 2 seconds for the trigger to fire.
3. Check function logs to confirm execution.
4. Verify the function read and processed the document correctly.

**Expected result:**
- Firestore write trigger invokes the function.
- Function logs show trigger execution and document data.

---

### TC-WORKER-004 — Task queue: immediate buffer execution

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Cloud Functions Emulator and RTDB Emulator running.
- Task queue worker (e.g., Zeus watchdog) is running or manually triggered.

**Steps:**
1. Write a task to `rtdb:/buffer/<uuid>` with `{"worker": "hermes", "at": 0, "options": {...}, "status": "new"}`.
2. Manually trigger the buffer polling cycle (or wait for automatic execution).
3. Verify the task status changed from `new` → `running` → `complete`.
4. Check function logs to confirm the worker executed.
5. Verify the side effect (e.g., a document was created, updated).

**Expected result:**
- Buffered task is picked up immediately.
- Worker executes and completes successfully.
- Task status progresses through lifecycle.
- Side effects are visible in Firestore/RTDB.

---

### TC-WORKER-005 — Task queue: scheduled tasks execution with polling

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- RTDB Emulator running.
- Polling cycle configured to run every 10 seconds (or manually triggered).

**Steps:**
1. Write a task to `rtdb:/tasks/<uuid>` with `{"at": <future-timestamp>, "worker": "lachesis", ...}`.
2. Verify the task is NOT executed immediately (status remains `new`).
3. Wait for the polling cycle to execute (or manually trigger it).
4. Verify the task is now executed (status changed to `complete`).
5. Confirm the side effect occurred.

**Expected result:**
- Scheduled tasks are NOT executed before their scheduled time.
- Polling cycle picks up tasks at the correct time.
- Task lifecycle is `new` → `running` → `complete`.

---

### TC-WORKER-006 — Error handling: failed task logged and moved to error queue

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Cloud Functions Emulator running.
- A worker function that can be made to fail (e.g., invalid input triggers an error).

**Steps:**
1. Write a task with invalid options that will cause the worker to throw an error.
2. Trigger the polling cycle (or buffer processing).
3. Verify the task status is now `error`.
4. Check `rtdb:/logs/error/<uuid>` to confirm the error log entry exists.
5. Verify the error message contains useful debugging information (stack trace, input, timestamp).

**Expected result:**
- Failed task status is set to `error`.
- Error log entry is created with full context.
- Task is moved to error queue for inspection/retry.

---

### TC-WORKER-007 — Per-tile write serialization: POI updates to same tile are queued, not raced

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Cloud Functions Emulator and Firestore Emulator running.
- Lachesis worker (tile update) implemented and testable.
- `tiled_views/{tileId}` document exists in Firestore.

**Steps:**
1. Create 3 feedback entries for POIs in the same tile (same `tileId`).
2. Write 3 tasks to the buffer, each updating the same tile (e.g., 3 Lachesis tasks for tileId=`12:34`).
3. Trigger buffer processing with concurrency enabled (e.g., parallel execution).
4. Verify that:
   - All 3 tasks complete successfully (no write conflicts).
   - The tile document was updated exactly 3 times (or consolidated if debounced).
   - No "document update conflict" errors occur.
5. Inspect the final tile state to confirm all 3 feedback updates were incorporated.

**Expected result:**
- Per-tile tasks are serialized (executed one at a time for the same tile).
- No write conflicts or data loss.
- All updates are applied correctly.
- Worker queue respects the per-tile write serialization invariant.

---

### TC-WORKER-008 — Retry logic: failed task is re-queued with exponential backoff

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Cloud Functions Emulator running.
- A worker function with retry logic implemented.
- A way to simulate transient failures (e.g., mock external API to fail once, then succeed).

**Steps:**
1. Write a task that will fail on first execution (e.g., external API call fails).
2. Trigger polling cycle — task executes and fails, status set to `error`.
3. Verify the task is re-queued with a delay: `rtdb:/tasks/<uuid>` still exists with `at` set to future timestamp.
4. Wait for the exponential backoff delay (e.g., 5 seconds for retry 1, 10 seconds for retry 2).
5. Mock the external API to succeed on retry.
6. Trigger polling cycle again — task executes and succeeds.
7. Verify final status is `complete`.

**Expected result:**
- Failed task is automatically re-queued.
- Retry delay increases with each attempt (exponential backoff).
- Task eventually succeeds if the underlying issue is transient.

---

### TC-WORKER-009 — Zeus watchdog: restarts dead worker or self-scheduled worker

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Zeus watchdog Cloud Function deployed.
- Pantheon workers (Hermes, Lachesis, Hera) have self-scheduling hooks.

**Steps:**
1. Simulate a worker crash: manually delete or mark a worker as dead (e.g., set a `dead` flag in RTDB).
2. Trigger Zeus watchdog to run.
3. Verify it detects the dead worker.
4. Verify it re-invokes the worker (or schedules a Cloud Scheduler job to restart it).
5. Confirm the worker is running again and processing tasks.

**Expected result:**
- Zeus watchdog detects dead workers.
- Watchdog restarts them without manual intervention.
- Workers resume processing tasks after restart.

---

### TC-WORKER-010 — Worker self-scheduling: worker schedules next run via Cloud Scheduler

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-fb-workers, gcp

**Preconditions:**
- Cloud Scheduler Emulator or actual Cloud Scheduler in `dev` project.
- A Pantheon worker (Hermes, Lachesis, or Hera) that self-schedules.

**Steps:**
1. Invoke a Pantheon worker manually (e.g., via HTTP trigger or Pub/Sub).
2. Worker executes and at the end self-schedules the next run.
3. Verify a Cloud Scheduler job is created or updated with the correct:
   - Schedule time (next execution timestamp)
   - Function target
   - Message payload (worker options)
4. Wait for the scheduled time and verify the worker is automatically invoked again.

**Expected result:**
- Worker successfully self-schedules its next run.
- Cloud Scheduler job is created with correct configuration.
- Worker is automatically invoked at the scheduled time.
- Cycle repeats without manual re-invocation.
