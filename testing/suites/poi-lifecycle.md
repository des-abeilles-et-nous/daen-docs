<!--
  Suite: POI Lifecycle & Backend  (testing/suites/poi-lifecycle.md)
  ───────────────────────────────────────────────────────────────
  PURPOSE : Functional Test (FUNC) suite
  ORIENTATION: User-Facing / Domain Behavior — tests POI creation, modification, and lifecycle
  
  These tests validate end-to-end POI creation workflow (user reports a POI), POI
  modifications (status changes, updates), and POI lifecycle state transitions.
-->

# Suite: POI Lifecycle & Backend

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 👤 **Functional Test (FUNC)** — User-Facing / Domain Behavior
> 
> This suite validates **POI creation and lifecycle workflows**, not infrastructure.
> Focus: POI creation, status transitions (active → handled → archived), map display, search.
>
> **Jira plan section:** POI Lifecycle & Backend — prefix `POI`
> **TC ID prefix:** `TC-POI`
> **Jira task type:** Story or Tâche
> **Reference docs:**
> - [`data model/POI lifecycle.md`](../../data%20model/POI%20lifecycle.md)
> - [`backend/Worker System.md`](../../backend/Worker%20System.md)

This suite validates that users can create POIs, that POIs are correctly displayed on the map, and that POI lifecycle state transitions work correctly.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (POI Lifecycle & Display)
- **POI creation** — user reports a new POI (bee-related observation)
- **POI metadata** — POI fields (code, location, description) are recorded correctly
- **POI display on map** — POIs appear on the map at the correct location
- **POI search/filtering** — users can search or filter POIs by category
- **POI status transitions** — POI status changes from active → handled/disbelieved → archived
- **Map clustering** — POIs are clustered on the map when zoomed out
- **POI details** — opening a POI shows all details (category, location, feedback counters)
- **POI creator attribution** — POI shows creator's pseudo and profile

### ❌ Out of Scope (Infrastructure & Feedback)
- **Worker processing** — tested in `[WORKER]`
- **Tile materialization** — tested in `[FB]` and `[DATA]`
- **Feedback aggregation** — tested in `[REPORT]`

---

## Test Cases

---

### TC-POI-001 — User creates a new POI (reports a beekeeping observation)

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User is logged in.
- User is on the map screen.
- User's location is enabled (or can manually select location).

**Steps:**
1. Tap the "+" (new POI) button on the map.
2. Select a POI category (e.g., "nest", "swarm", "hornet", "beekeeper").
3. Optionally add a description (e.g., "Active nest in oak tree").
4. Select or confirm the location (should be user's current location).
5. Tap "Create POI" button.
6. Verify a confirmation message: "POI created successfully".
7. Verify the new POI appears on the map at the selected location.
8. Verify a document was created in Firestore `POIs` collection with the correct data.

**Expected result:**
- POI is created with correct metadata.
- POI appears on the map.
- Creator ID is set to the logged-in user.

---

### TC-POI-002 — New POI appears in correct tile on map

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User has created a POI.

**Steps:**
1. Open the map and zoom to the tile containing the POI.
2. Verify the POI is visible on the map.
3. Verify the POI appears in the correct grid tile (based on coordinates).
4. Pan to an adjacent tile and verify the POI is not displayed (in that tile).
5. Zoom out to see multiple tiles.
6. Verify POIs are correctly distributed across tiles.
7. Zoom in and verify individual POIs are clustered or displayed based on zoom level.

**Expected result:**
- POI appears in the correct tile on the map.
- POI is not displayed in other tiles.
- Clustering works correctly at different zoom levels.

---

### TC-POI-003 — POI search: user finds POI by category or name

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- Multiple POIs exist on the map.

**Steps:**
1. Open the map or POI list screen.
2. Tap the search or filter button.
3. Filter by category (e.g., "nest", "swarm").
4. Verify only POIs of that category are displayed.
5. Use search by location name (e.g., "Paris", "rue de").
6. Verify POIs in that area are shown.
7. Clear filters and verify all POIs are displayed again.

**Expected result:**
- Search/filter functionality works correctly.
- Only matching POIs are displayed.

---

### TC-POI-004 — POI detail: all fields are displayed correctly

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- POI exists.

**Steps:**
1. Open POI detail screen.
2. Verify all information is displayed:
   - Category (code) with icon/color
   - Location (address or coordinates)
   - Description (if provided)
   - Feedback counters: also seen (+), handled (✓), not seen (✗), thumbup (👍)
   - Creator name (pseudo)
   - Creation date/time
   - Last update date/time
3. Verify the creator's profile picture is displayed (if available).
4. Tap the creator's name to open their profile.

**Expected result:**
- All POI fields are correctly displayed.
- Links to creator profile work.

---

### TC-POI-005 — POI status transition: active POI → handled

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- A POI exists with status "active".
- Threshold for status change defined (e.g., 5 "handled" votes).

**Steps:**
1. Gather feedback from 5 test users or accounts: each submits "✓ Handled" feedback.
2. Wait for worker to process feedback and update POI status.
3. Reload the POI detail screen.
4. Verify the status changed to "handled".
5. Verify the UI updated (e.g., status badge, color change, location on map).
6. Verify the POI can still be viewed but is marked as handled.

**Expected result:**
- POI status changes automatically.
- UI reflects the new status.

---

### TC-POI-006 — POI status transition: active POI → disbelieved

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- A POI exists with status "active".

**Steps:**
1. Gather feedback: multiple users submit "✗ Not seen" votes.
2. Wait for processing.
3. Verify the POI status changed to "disbelieved" (if threshold reached).
4. Verify the UI shows "disbelieved" status.

**Expected result:**
- POI status transitions correctly based on feedback.

---

### TC-POI-007 — POI status transition: handled/disbelieved → archived

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- A POI with status "handled" or "disbelieved" exists.
- Time-based archival is configured (e.g., auto-archive after 30 days).

**Steps:**
1. Wait for the time-based archival condition to trigger (or manually trigger in test).
2. Verify the POI status changed to "archived".
3. Verify the POI is still viewable but marked as archived.
4. Verify the POI is moved from `POIs` to `POIs_attic` collection in Firestore.
5. Verify the POI is no longer displayed on the map (or shown separately as archived).

**Expected result:**
- POI is archived after appropriate time.
- Archived POI is moved to `POIs_attic`.
- Map no longer shows archived POIs.

---

### TC-POI-008 — POI update: user can edit POI description

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- POI exists and creator is the logged-in user.

**Steps:**
1. Open POI detail screen (must be creator to edit).
2. Tap "Edit" button.
3. Modify the description.
4. Tap "Save" button.
5. Verify the description is updated.
6. Reload the POI detail and verify the change persists.
7. Verify `m_t` (modified timestamp) is updated.

**Expected result:**
- POI description can be edited by creator.
- Changes persist.
- Last modified time is updated.

---

### TC-POI-009 — POI immutable fields: user cannot edit creator_id or creation time

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- POI exists.

**Steps:**
1. Open POI edit screen (if creator).
2. Verify `creator_id` field is not editable (hidden or disabled).
3. Verify `created_t` field is not editable (hidden or disabled).
4. Verify other immutable fields are protected.

**Expected result:**
- Immutable fields cannot be edited.
- Security rules prevent modification attempts.

---

### TC-POI-010 — POI creation with precise location: coordinates are accurate

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User can enable precise location (GPS or map selection).

**Steps:**
1. Create a POI at a specific, known location (e.g., a test landmark).
2. Specify the exact coordinates (e.g., 48.8566, 2.3522 for Eiffel Tower).
3. Verify the POI is created with those exact coordinates.
4. Query the POI in Firestore and verify `lat` and `lon` match.
5. Display the POI on a map and verify it's at the correct location.
6. Use geospatial distance calculation to verify accuracy (should be within 1 meter).

**Expected result:**
- POI coordinates are accurate.
- POI appears at the correct location on the map.
