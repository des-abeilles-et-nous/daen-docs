<!--
  Suite: Data Integrity & Schema Validation  (testing/suites/data-integrity.md)
  ──────────────────────────────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests data validation, not user features
  
  These tests validate that Firestore data conforms to schema, that cross-collection
  consistency is maintained, and that data migrations don't corrupt state.
-->

# Suite: Data Integrity & Schema Validation

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **data schema compliance and cross-collection consistency**, not user workflows.
> Focus: field-level schema validation, type enforcement, cross-collection invariants, migration integrity.
>
> **Jira plan section:** Data Integrity & Schema Validation — prefix `DATA`
> **TC ID prefix:** `TC-DATA`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`data model/`](../../data%20model/) — all schema definitions

This suite validates that all data stored in Firestore conforms to documented schemas, that types are correct, and that cross-collection invariants are maintained.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Data Schema & Integrity)
- **Field-level validation** — required fields present, correct types, value constraints
- **Collection schema compliance** — every document conforms to documented schema
- **Type enforcement** — strings are strings, numbers are numbers, arrays have correct item types
- **Enum validation** — status fields contain only allowed values (e.g., `handled|disbelieved|locked|archived`)
- **Cross-collection invariants** — POIs in `POIs` have corresponding entries in `tiled_views`, users referenced in POIs exist, etc.
- **Timestamp validation** — timestamps are Unix milliseconds, creation < modification < now
- **Data migration integrity** — schema changes don't corrupt existing documents
- **Null/undefined fields** — required fields are never null, optional fields are either omitted or null (not both)

### ❌ Out of Scope (User-Facing Workflows)
- **Feature behavior** — tested in functional suites
- **Business logic correctness** — tested in functional suites (e.g., "POI counter is correct" is `[POI]`, not `[DATA]`)

---

## Test Cases

---

### TC-DATA-001 — POI schema: required fields present and correctly typed

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POI documents exist in `POIs` collection.
- Schema definition available in `data model/mdd daen-scout.md`.

**Steps:**
1. Query all POI documents: `db.collection("POIs").get()`.
2. For each POI, verify required fields exist: `uid`, `code`, `lat`, `lon`, `created_t`, `creator_id`, `status`.
3. Verify field types:
   - `uid`: string
   - `lat`, `lon`, `alt`: number
   - `code`: string (non-empty)
   - `created_t`, `m_t`: number (Unix timestamp)
   - `status`: string in `["handled", "disbelieved", "locked", "archived"]`
   - `creator_pseudo`: string or omitted (optional)
4. Verify optional fields are either omitted or non-null (not undefined).

**Expected result:**
- All POIs have required fields.
- All field types are correct.
- No POIs have missing required data.

---

### TC-DATA-002 — user schema: required fields present and correctly typed

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- User documents exist in `users` collection.
- Schema definition available.

**Steps:**
1. Query all user documents.
2. For each user, verify required fields: `uid`, `email`, `display_name`, `created_t`.
3. Verify optional fields: `pseudo`, `picture`, `isBeekeeper`, `isHunter`, `refLocation`.
4. Verify field types:
   - `uid`: string (UUID format)
   - `email`: string (email format)
   - `created_t`, `modified_t`: number (Unix timestamp)
   - `isBeekeeper`, `isHunter`: boolean or omitted
   - `refLocation`: object with `latitude`, `longitude`, `isoCountryCode` (all numbers/strings)

**Expected result:**
- All users have required fields.
- All field types are correct.
- Email addresses are valid format.

---

### TC-DATA-003 — tile schema: required fields present and map array valid

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- Tile documents exist in `tiled_views` collection.

**Steps:**
1. Query all tile documents.
2. For each tile, verify required fields: `id`, `a` (arc size), `poi_lists`, `updated_t`.
3. Verify field types:
   - `id`: string (tile ID)
   - `a`: number (arc size in degrees)
   - `poi_lists`: object with string keys (category codes) and array values
   - Each item in `poi_lists[category]` is a POIview object with: `uid`, `lat`, `lon`, `code`, `status`
4. Verify all POIs referenced in `poi_lists` actually exist in the `POIs` collection (cross-collection check).

**Expected result:**
- All tiles have required fields.
- POI lists are correctly structured.
- All referenced POIs exist.

---

### TC-DATA-004 — Cross-collection invariant: POIs have corresponding tiled_views entries

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POIs and tiled_views collections populated.
- Tiles computed based on POI coordinates.

**Steps:**
1. For each POI in `POIs` collection:
   - Compute its tile ID from lat/lon (using the tile algorithm).
   - Query the corresponding `tiled_views/{tileId}` document.
   - Verify the POI appears in that tile's `poi_lists[code]` array.
2. For each tile in `tiled_views`:
   - For each POI listed in the tile's `poi_lists`:
     - Query `POIs/{uid}` and verify the document exists.
     - Verify the stored POI data matches the tiled_view version (same `lat`, `lon`, `status`).

**Expected result:**
- Every active POI appears in exactly one tile (the tile containing its coordinates).
- Every POI in a tile exists in the POIs collection.
- POI data in tiles matches the canonical POI document.

---

### TC-DATA-005 — Enum validation: status fields contain only allowed values

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P1
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POI and tile documents populated.

**Steps:**
1. Query all POI documents and check `status` field values.
2. Verify all statuses are in the allowed set: `["handled", "disbelieved", "locked", "archived"]`.
3. Verify no POI has status `null`, `undefined`, or an unexpected value like `"invalid"`.
4. Repeat for other enum-like fields (e.g., feedback types: `PLUS`, `HANDLED`, `NOTSEEN`, `THUMBUP`).

**Expected result:**
- All enum fields contain only documented allowed values.
- No invalid enum values.

---

### TC-DATA-006 — Timestamp validation: creation < modification < now

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POI and user documents with timestamps.

**Steps:**
1. For each document with `created_t` and `m_t` (modified time):
   - Verify `created_t <= m_t`.
   - Verify both `created_t` and `m_t` are less than or equal to the current time.
2. Detect any documents with future timestamps (which would indicate clock skew or data corruption).
3. Check timestamp format: all timestamps should be Unix milliseconds (13 digits), not seconds.

**Expected result:**
- All timestamps are in correct order: creation ≤ modification ≤ now.
- No future-dated documents.
- All timestamps are in milliseconds.

---

### TC-DATA-007 — Data migration: old schema fields are absent or deprecated gracefully

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, staging (after migration)
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- A data migration has been performed (e.g., field rename, structure change).
- Old and new field names documented.

**Steps:**
1. Query documents after migration.
2. Verify the new field exists and is populated.
3. Verify the old field is absent (or marked as deprecated).
4. Verify no documents have both old and new fields (which would indicate incomplete migration).
5. Spot-check data values: verify old data was correctly transformed into new format.

**Expected result:**
- Migration is complete and consistent.
- All documents use the new schema.
- No duplicate or conflicting old/new data.

---

### TC-DATA-008 — sequences collection: counters are monotonically increasing

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- `sequences` collection exists with counter documents.

**Steps:**
1. Read the current counter value from `sequences/{seq_name}`.
2. Create a new document that uses the next counter value (e.g., new POI with `num_id = counter + 1`).
3. Increment the counter in `sequences/{seq_name}`.
4. Verify the counter value increased by exactly 1.
5. Create another document and verify it uses the incremented counter.
6. Query all documents using this counter and verify no gaps (sequential).

**Expected result:**
- Counters are monotonically increasing.
- No gaps in sequence.
- No duplicate counter values.

---

### TC-DATA-009 — Numeric fields: lat/lon within valid ranges

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POI documents with lat/lon coordinates.

**Steps:**
1. For each POI, verify:
   - `lat` is in range `[-90, 90]`.
   - `lon` is in range `[-180, 180]`.
   - Both are numbers (not strings or null).
2. Detect any POIs with invalid coordinates (which would break geospatial queries).

**Expected result:**
- All coordinates are within valid geographic ranges.
- No invalid coordinate values.

---

### TC-DATA-010 — Referential integrity: users referenced in POIs exist

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev, sandbox, staging
**Priority:** P2
**Component:** daen-fb-workers, firebase

**Preconditions:**
- POIs and users collections populated.

**Steps:**
1. For each POI:
   - Read `creator_id` field.
   - Query `users/{creator_id}` to verify the user exists.
   - Verify the user's `created_t <= POI.created_t` (creator existed before POI was created).
2. Collect any broken references (POIs with `creator_id` pointing to non-existent users).

**Expected result:**
- All POIs reference valid users.
- No broken cross-collection references.
- Creator creation time is before POI creation time.
