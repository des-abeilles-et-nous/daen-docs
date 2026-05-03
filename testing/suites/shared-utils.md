<!--
  Suite: Shared Utilities & Cross-Repo Helpers  (testing/suites/shared-utils.md)
  ────────────────────────────────────────────────────────────────────────────
  PURPOSE : Integration Test (IT) suite
  ORIENTATION: Technical / Infrastructure — tests utility code, not user features
  
  These tests validate that shared libraries and cross-repo utilities (Bit components)
  function correctly and are properly integrated across repos.
-->

# Suite: Shared Utilities & Cross-Repo Helpers

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 🔧 **Integration Test (IT)** — Infrastructure & Technical
> 
> This suite validates **shared utility functions and cross-repo library integration**, not user workflows.
> Focus: Bit component behavior, utility function correctness, cross-repo dependency resolution.
>
> **Jira plan section:** Shared Utilities & Cross-Repo Helpers — prefix `SHARED`
> **TC ID prefix:** `TC-SHARED`
> **Jira task type:** Tâche or IT
> **Reference docs:**
> - [`ECOSYSTEM_CONTEXT.md` — Shared Libraries](../../ECOSYSTEM_CONTEXT.md)
> - Bit component documentation: `bit.dev/desabeillesetnous/daen-js-shared`

This suite validates that shared utility functions and Bit components are correctly exported, imported, and used across repositories.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Shared Utilities & Library Integration)
- **Bit component export** — utility components are correctly tagged and exported from Bit
- **Bit component import** — utilities can be imported in other repos without errors
- **Function correctness** — shared utility functions produce correct outputs for given inputs
- **Type definitions** — TypeScript types exported by shared components are correct
- **Cross-repo dependency resolution** — shared utilities are found and linked correctly across repos
- **Library versioning** — shared component versions are pinned and consistent
- **Error handling** — shared utilities handle edge cases and error conditions gracefully

### ❌ Out of Scope (User-Facing Features)
- **Component rendering** (UI components tested in `[AUTH]`, `[POI]`, etc.)
- **User workflows** — tested in functional suites
- **Business logic** — tested in functional suites

---

## Test Cases

---

### TC-SHARED-001 — Bit component export: component is tagged and published

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- Bit workspace configured.
- Shared components authored (e.g., `desabeillesetnous.daen-js-shared/utils/date-utils`).

**Steps:**
1. Run `bit list` and verify shared components are listed.
2. Verify each component has a version number (e.g., `0.0.1`).
3. Check that components are published to Bit Cloud: `bit export --dry-run`.
4. Verify component metadata (description, dependencies, version) is correct.

**Expected result:**
- Shared components are properly tagged and exported.
- Component metadata is complete.
- Components are accessible via Bit Cloud.

---

### TC-SHARED-002 — Bit component import: component installs without errors in dependent repo

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- Shared component published to Bit.
- Package manager (npm/yarn) configured.

**Steps:**
1. In a dependent repo (e.g., daen-scout), install a shared component: `npm install @desabeillesetnous.daen-js-shared/utils`.
2. Verify the installation succeeds without errors.
3. Verify the component files are present in `node_modules/`.
4. Import the component in code: `import { utilFunc } from '@desabeillesetnous.daen-js-shared/utils'`.
5. Verify the import resolves without errors (no TypeScript errors, ESLint warnings).

**Expected result:**
- Shared components install without errors.
- Imports resolve correctly.
- No missing dependency issues.

---

### TC-SHARED-003 — Date utility functions: format, parse, compare work correctly

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, shared

**Preconditions:**
- Date utility component available (`daen-js-shared/utils/date-utils`).

**Steps:**
1. Test `formatDate()`:
   - Input: timestamp `1609459200000` (2021-01-01 00:00:00 UTC)
   - Expected output: `"2021-01-01"` (or configured format)
   - Verify date formatting with different locales (e.g., `en-US`, `fr-FR`)
2. Test `parseDate()`:
   - Input: `"2021-01-01"`
   - Expected output: timestamp `1609459200000`
   - Test edge cases: leap years, daylight saving time transitions
3. Test `isValidDate()`:
   - Valid input: `"2021-01-01"` → `true`
   - Invalid input: `"2021-13-01"`, `"invalid"` → `false`
4. Test `compareDates()`:
   - Compare two dates and verify ordering (before, equal, after)

**Expected result:**
- Date functions produce correct outputs for all test cases.
- Edge cases handled correctly.
- No timezone issues.

---

### TC-SHARED-004 — Geospatial utility functions: tile calculation, coordinate validation

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, shared

**Preconditions:**
- Geospatial utility component available.

**Steps:**
1. Test `calculateTileId()`:
   - Input: latitude `48.8566`, longitude `2.3522` (Paris)
   - Expected: tile ID string (e.g., `"48:2"`)
   - Test multiple coordinates and verify tile boundaries
2. Test `isValidCoordinate()`:
   - Valid: `(48.8566, 2.3522)` → `true`
   - Invalid: `(91, 2)` (lat > 90) → `false`
   - Invalid: `(48, 181)` (lon > 180) → `false`
3. Test `distanceBetween()`:
   - Two nearby points (< 1 km) → distance in meters
   - Verify Haversine formula correctness
4. Test `boundsFromCenter()`:
   - Center point and radius → bounding box
   - Verify bounding box contains the center

**Expected result:**
- Geospatial functions are accurate.
- Tile IDs are consistent.
- Coordinate validation works correctly.

---

### TC-SHARED-005 — Enum and constant definitions: values consistent across repos

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- Shared enums/constants component available (e.g., `daen-js-shared/constants/enums`).

**Steps:**
1. Query POI status enum from shared component.
2. Verify values match documentation: `["handled", "disbelieved", "locked", "archived"]`.
3. Import the enum in daen-scout and verify values are accessible.
4. Import the same enum in daen-fb-workers and verify values match (no divergence).
5. Test feedback type enum: `["PLUS", "HANDLED", "NOTSEEN", "THUMBUP"]`.
6. Verify no enum value collisions or ambiguities.

**Expected result:**
- Enums are correctly defined.
- Values are consistent across repos.
- No enum divergence.

---

### TC-SHARED-006 — TypeScript types: type definitions are correct and accessible

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, shared

**Preconditions:**
- Shared types component available (e.g., `daen-js-shared/types/models`).

**Steps:**
1. Import a shared type: `import type { POI } from '@desabeillesetnous.daen-js-shared/types'`.
2. Use the type in a variable declaration: `const poi: POI = {...}`.
3. Verify TypeScript compiler accepts the type (no type errors).
4. Test type narrowing: verify TypeScript can narrow POI types based on properties.
5. Test generic types: if shared types use generics (e.g., `Response<T>`), verify they work correctly.

**Expected result:**
- Types import without errors.
- TypeScript compilation succeeds.
- Types are correctly structured.

---

### TC-SHARED-007 — Validation utilities: input validation functions work correctly

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, shared

**Preconditions:**
- Validation utility component available.

**Steps:**
1. Test `isValidEmail()`:
   - Valid: `"user@example.com"` → `true`
   - Invalid: `"invalid"`, `"@example.com"`, `"user@"` → `false`
2. Test `isValidPseudo()`:
   - Valid: `"john_doe"`, `"user123"` → `true`
   - Invalid: `""`, `"user@invalid"` (special chars) → `false`
3. Test `sanitizeInput()`:
   - Input: `"<script>alert('xss')</script>"` → sanitized output without dangerous HTML
4. Test numeric validation: `isInteger()`, `isPositive()`, `isInRange()`

**Expected result:**
- Validation functions correctly identify valid/invalid inputs.
- No false positives or negatives.
- Sanitization prevents injection attacks.

---

### TC-SHARED-008 — Error handling utilities: custom errors work correctly

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, shared

**Preconditions:**
- Shared error class component available.

**Steps:**
1. Test custom error throwing: `throw new AppError("Invalid input", "VALIDATION_ERROR", 400)`.
2. Verify error code and message are accessible.
3. Test error serialization: convert error to JSON (for logging).
4. Test error handling in try/catch blocks.
5. Verify error stack trace is preserved.

**Expected result:**
- Custom errors throw and catch correctly.
- Error metadata (code, message, HTTP status) is accessible.
- Errors are serializable for logging.

---

### TC-SHARED-009 — Logging utilities: structured logging works across repos

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- Shared logging utility available.

**Steps:**
1. Use shared logger in daen-scout: `logger.info("event", { userId, action })`.
2. Verify log output includes structured fields (user ID, action).
3. Use same logger in daen-fb-workers.
4. Verify log format is consistent across repos.
5. Test log levels: `debug`, `info`, `warn`, `error`.
6. Verify error logging includes stack traces.

**Expected result:**
- Logging is consistent across repos.
- Structured logging preserves fields.
- Log levels work correctly.

---

### TC-SHARED-010 — Component version pinning: all repos use same version

**Type:** integration
**Scope:** repo
**Jira type:** Tâche or IT
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, fb-admin, shared

**Preconditions:**
- Multiple repos consuming shared components.

**Steps:**
1. Extract shared component version from `package.json` in daen-scout.
2. Extract same component version from `package.json` in daen-fb-workers.
3. Extract same component version from `package.json` in fb-admin.
4. Verify all versions match (no divergence).
5. If versions differ, identify the version mismatch and resolve it.
6. Run builds in all repos and verify no compatibility issues.

**Expected result:**
- All repos pin the same version of shared components.
- No version conflicts.
- Builds succeed across all repos.
