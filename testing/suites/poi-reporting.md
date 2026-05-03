<!--
  Suite: POI Reporting & Feedback  (testing/suites/poi-reporting.md)
  ──────────────────────────────────────────────────────────────────
  PURPOSE : Functional Test (FUNC) suite
  ORIENTATION: User-Facing / Domain Behavior — tests user feedback submission workflows
  
  These tests validate end-to-end feedback submission workflows (user submits feedback
  on POIs, feedback counters are updated, workers process feedback correctly).
-->

# Suite: POI Reporting & Feedback

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 👤 **Functional Test (FUNC)** — User-Facing / Domain Behavior
> 
> This suite validates **user feedback submission workflows and feedback processing**, not infrastructure.
> Focus: feedback submission, counter updates, worker processing, feedback aggregation.
>
> **Jira plan section:** POI Reporting & Feedback — prefix `REPORT`
> **TC ID prefix:** `TC-REPORT`
> **Jira task type:** Story or Tâche
> **Reference docs:**
> - [`data model/`](../../data%20model/) — feedback schema, POI counters
> - [`backend/Worker System.md`](../../backend/Worker%20System.md) — Lachesis worker

This suite validates that users can submit feedback on POIs, that feedback is recorded correctly, and that workers aggregate feedback into POI counter updates.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Feedback Workflows)
- **Feedback submission** — user submits "also seen", "not seen", "thumbup", or "handled" feedback on a POI
- **Feedback types** — all feedback types (PLUS, HANDLED, NOTSEEN, THUMBUP) are correctly recorded
- **Feedback counters** — POI counters (plus, notseen, tup, handled) are updated after feedback
- **Counter aggregation** — workers aggregate feedback into counters correctly
- **Duplicate feedback prevention** — user cannot submit the same feedback type twice on the same POI
- **Feedback UI feedback** — user sees confirmation after submitting feedback
- **Feedback history** — user's feedback history is tracked and visible

### ❌ Out of Scope (Infrastructure & Advanced Features)
- **Worker implementation details** — tested in `[WORKER]`
- **Cross-collection consistency** — tested in `[DATA]`
- **Notification triggers** — tested in `[NOTIF]`

---

## Test Cases

---

### TC-REPORT-001 — User submits "also seen" feedback on a POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, fb-workers, firebase

**Preconditions:**
- User is logged in.
- At least one POI exists on the map.
- POI detail screen is accessible.

**Steps:**
1. Navigate to a POI on the map.
2. Open the POI detail screen.
3. Tap the "+ Also seen" button.
4. Verify a confirmation message: "Feedback submitted".
5. Wait 2 seconds for the worker to process.
6. Verify the POI counter for "also seen" increments (UI updates or reload to verify).
7. Verify the feedback is recorded in `rtdb:/feedbacks/` and in the user's profile.

**Expected result:**
- Feedback is submitted.
- Confirmation is shown to the user.
- POI "plus" counter increments.
- Feedback is recorded in the backend.

---

### TC-REPORT-002 — User submits "handled" feedback on a POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, fb-workers, firebase

**Preconditions:**
- User is logged in.
- POI exists (especially a bee-related POI like "nest" or "swarm").

**Steps:**
1. Open a POI detail screen.
2. Tap the "✓ Handled" button.
3. Optionally select a sub-type (e.g., "nest destroyed", "swarm passed").
4. Verify feedback is submitted.
5. Wait for worker processing.
6. Verify the POI counter for "handled" increments.
7. Verify the POI status may change if threshold is reached.

**Expected result:**
- "Handled" feedback is recorded.
- Counter increments.
- Confirmation is shown.

---

### TC-REPORT-003 — User submits "not seen" feedback on a POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, fb-workers, firebase

**Preconditions:**
- User is logged in.
- POI exists on map.

**Steps:**
1. Open POI detail screen.
2. Tap "✗ Not seen" button.
3. Verify feedback is submitted.
4. Wait for processing.
5. Verify the POI "notseen" counter increments.

**Expected result:**
- "Not seen" feedback is recorded and counter updates.

---

### TC-REPORT-004 — User submits "thumbup" feedback on a POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, fb-workers, firebase

**Preconditions:**
- User is logged in.
- POI exists.

**Steps:**
1. Open POI detail screen.
2. Tap "👍 Thumbup" button.
3. Verify feedback submitted.
4. Wait for processing.
5. Verify the POI "tup" (thumbup) counter increments.

**Expected result:**
- Thumbup feedback is recorded and counter updates.

---

### TC-REPORT-005 — User cannot submit duplicate feedback on same POI

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- User already submitted feedback on a POI (e.g., "+1").

**Steps:**
1. Open the same POI detail screen.
2. Attempt to submit the same feedback type again (e.g., tap "+ Also seen" again).
3. Verify the action is prevented with a message: "You already voted this".
4. Verify the button is disabled or shows the user's previous feedback.

**Expected result:**
- Duplicate feedback is prevented.
- User is informed they already voted.

---

### TC-REPORT-006 — User can submit different feedback types on the same POI

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User is logged in.
- POI exists.

**Steps:**
1. Submit "+ Also seen" feedback on a POI.
2. Verify the button shows user already voted.
3. Attempt to submit "👍 Thumbup" feedback on the same POI.
4. Verify the thumbup is submitted (different type, so allowed).
5. Verify both counters increment.

**Expected result:**
- User can submit multiple feedback types on the same POI.
- Each counter increments independently.

---

### TC-REPORT-007 — Feedback counter aggregation: multiple users' feedback updates POI counter

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- Multiple test user accounts available.
- Same POI visible to all users.

**Steps:**
1. **User A** submits "+ Also seen" feedback on POI X.
   - Verify User A's counter increments.
2. **User B** (different user) submits "+ Also seen" feedback on the same POI X.
   - Verify the POI counter increments again (User A's vote + User B's vote).
3. **User C** submits "+ Also seen" feedback.
   - Verify counter increments to 3.
4. Reload POI detail and verify final counter shows 3.

**Expected result:**
- Feedback from multiple users aggregates correctly.
- POI counter reflects all user votes.
- No race conditions or lost votes.

---

### TC-REPORT-008 — Feedback history: user can view their feedback history

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has submitted feedback on multiple POIs.

**Steps:**
1. Navigate to user profile or "Feedback history" screen.
2. Verify a list of all feedback submitted by the user is displayed.
3. For each feedback entry, verify:
   - POI name or ID
   - Feedback type (also seen, handled, etc.)
   - Timestamp of feedback submission
4. Tap a feedback entry to view POI details.
5. Verify the POI is displayed correctly.

**Expected result:**
- User can view their feedback history.
- All entries are correctly displayed.
- Feedback metadata (type, timestamp) is accurate.

---

### TC-REPORT-009 — POI status changes based on feedback thresholds

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- POI exists with status "new" or "active".
- Feedback threshold defined (e.g., 5 "handled" votes = status → "handled").

**Steps:**
1. Submit "handled" feedback 5 times (from different test users or manually).
2. Wait for worker to process all feedback.
3. Reload the POI detail screen.
4. Verify the POI status changed to "handled".
5. Verify the UI reflects the status change (e.g., color change, status badge).

**Expected result:**
- POI status changes automatically when feedback threshold is reached.
- UI updates to reflect new status.

---

### TC-REPORT-010 — Feedback submission fails gracefully with offline app

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User is logged in.
- App is online initially.

**Steps:**
1. Go offline (disconnect network or enable airplane mode).
2. Attempt to submit feedback on a POI.
3. Verify the app either:
   - Queues the feedback locally and shows "Queued" status, or
   - Shows "Cannot submit: offline" error
4. Go back online.
5. If feedback was queued, verify it's automatically submitted.
6. Verify the POI counter increments.

**Expected result:**
- App handles offline gracefully.
- Feedback either queues or shows a clear error.
- No app crash.
