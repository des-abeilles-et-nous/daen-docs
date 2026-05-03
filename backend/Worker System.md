# Worker System

Backend execution engine for daen-fb-workers. All asynchronous business logic runs through a two-tier task queue dispatched to named workers.

---

## Architecture

### Two-tier queue

| Queue | RTDB path | Purpose |
|---|---|---|
| Buffer | `rtdb:/buffer` | Immediate, latency-sensitive operations (no throttling) |
| Tasks | `rtdb:/tasks` | Heavy Firestore writes, throttled via adaptive `tasksRunner` |

`bufferProcessor` fires on every new buffer item. `scheduled-tasksRunner` fires every minute and processes up to `maxBatchSize` tasks (adaptive, capped at 24).

### Worker dispatch

Both queues dispatch to the same shared `workers` registry. A task record contains `{ worker, options, status, at }`. The dispatcher calls `workers[worker]({ ...options, self: taskId })`.

---

## The Pantheon Pattern

Three workers — **Hermes**, **Hera**, and **Lachesis** — are **polling workers**, not event-driven. They run on a continuous loop maintained by two mechanisms:

1. **Zeus** (hourly): scans `rtdb:/tasks` and re-queues any pantheon member that is missing
2. **Self-scheduling**: each pantheon worker calls `addUniqueWorkerFrom(self)` at the end of its run

This means these workers are effectively always-on background processes, not triggered by specific events. The absence of an incoming trigger edge is intentional.

`scheduled-hypervisor` runs Zeus hourly (at :13) as the safety net.

---

## Named Workers

### Zeus — Pantheon Watchdog

Invoked hourly by `scheduled-hypervisor`. Scans `rtdb:/tasks` for scheduled entries and re-queues any missing pantheon worker (`lachesis`, `hermes`, `hera`). Also schedules `resetter` for stuck-task auto-recovery.

**Not** a polling worker itself — relies entirely on the Cloud Scheduler.

---

### Hermes — Notification Delivery

Polling worker (pantheon member). Scans the `news_roll` Firestore collection group for pending notifications and sends them via Expo push.

**Flow:**
1. Query `news_roll` subcollections (`status == 'new'`) across all users
2. Consolidate per user: count `plus`, `tup`, `notseen`, affected POIs
3. Build push payload via `hermesNotifMessage()`
4. Send via Expo Server SDK (`bulkSendExpoMessages`)
5. Schedule `hermes-cleaner` for Expo receipt retrieval
6. Self-reschedule

**What writes to `news_roll`:**
- `poi-poiUpdated` trigger — on new POI activity, writes to `users/{creatorId}/news_roll`
- `lachesis` worker — writes creator news when feedback scores are applied

---

### Lachesis — Feedback Batch Processor

Polling worker (pantheon member). Reads all pending feedback records from `rtdb:/feedbacks`, groups them by tile and POI, applies score updates, and records feeder activity.

**Flow:**
1. Read all `rtdb:/feedbacks` where `status == 'new'`
2. Group by tile → POI
3. `persist.updatePOIFromFeedbacks()` per tile
4. `persist.recordUserFeedback()` per feeder
5. Delete consumed feedback records from RTDB
6. Self-reschedule

**Contrast with Feme:** Lachesis is a batch poller. Feme processes a single feedback immediately on-demand with a duplicate-detection guard.

---

### Hera — Tile Integrity Sweep

Polling worker (pantheon member). Scans all `tiled_views` documents and ensures every tile with active POIs has a clotho run scheduled. Cleans up empty tiles.

**Flow:**
1. Load all `tiled_views`
2. Check `rtdb:/tasks` for already-scheduled `clotho` workers
3. Tiles with POIs but no pending clotho → schedule clotho with `launch_buffer` delay
4. Tiles with no POIs and no subcollections → delete

---

### Clotho — Per-tile POI Recalculation

Event-driven per tile. Applies business rules to all POIs in a tile, updates the `tiled_views` materialized view, and archives expired/disbelieved POIs.

**Triggered by:** `poi-poiCreated/Updated/Deleted`, `tiles-poiTileUpdated` (via `_clotho` flag), Hera (for tiles with no pending run). Self-reschedules for next cycle.

**Business rules applied:**
- `fading` — time-based visibility decay
- `opacity` — derived from fading × credibility
- `credibility` — community feedback score
- POI status transitions: `new` → `ageing` → `locked` → `archived`/`disbelieved`
- Moves expired/disbelieved POIs to `POIs_attic` (permanent archive)

**Options:** `tile` (required), `since` (partial update from timestamp), `full` (force full scan).

---

### Feme — Single Feedback Processor

On-demand, per-feedback-event worker. Immediate counterpart to Lachesis.

Checks for duplicate feedback (same user, same type, same POI) before applying. Updates POI scores and records feeder activity. Does not self-reschedule — not a pantheon member.

---

## Worker Responsibility Map

| Worker | Trigger model | Reads | Writes |
|---|---|---|---|
| Zeus | Cloud Scheduler (hourly) | `rtdb:/tasks` | `rtdb:/tasks` |
| Hermes | Polling (pantheon) | `fs:users/*/news_roll` | Push notifications, `rtdb:/tasks` |
| Lachesis | Polling (pantheon) | `rtdb:/feedbacks` | `fs:POIs`, `fs:users`, `rtdb:/tasks` |
| Hera | Polling (pantheon) | `fs:tiled_views`, `rtdb:/tasks` | `rtdb:/tasks`, `fs:tiled_views` (delete) |
| Clotho | Event-driven per tile | `fs:POIs`, `fs:tiled_views` | `fs:tiled_views`, `fs:POIs_attic`, `rtdb:/tasks` |
| Feme | On-demand per feedback | `fs:POIs`, `rtdb:/feedbacks` | `fs:POIs`, `fs:users` |

---

## Architectural Assessment

This section captures a deliberate trade-off analysis of the worker system against market standards and GCP-native alternatives. It exists so future contributors understand the design constraints and known improvement paths before changing the system.

### What holds up well

- **Single-responsibility workers** — each worker does one thing, is idempotent, and has a clear data contract. Matches standard worker/consumer patterns.
- **Two-tier queue separation** — buffer (latency) vs. tasks (throughput) maps directly to priority queue patterns used in production systems.
- **Adaptive throttling** (`maxRunnable = ceil(lastLaunched × 1.1)`) — pragmatic solution to Firestore write quota management that prevents cascading retries.
- **Lachesis batching** — processing all pending feedbacks in one invocation is more cost-efficient than N separate invocations.
- **30s trigger lag (clotho)** — batches rapid consecutive POI edits into a single tile rebuild, reducing unnecessary Firestore writes.

### The hidden invariant: per-tile write serialization

`tiled_views` is a denormalized materialized view of POIs grouped by geographic tile — each tile document is read by every nearby mobile client and written by Clotho (POI lifecycle), Lachesis (feedback scores), and Hera (tile cleanup). Read-side this is excellent: one tile fetch returns N POIs. Write-side it concentrates all activity for an area onto a single document, where Firestore's per-document write quota (~5/sec burst, 1/sec sustained) becomes the binding constraint.

The polling cycle currently provides per-tile write serialization as a side effect: by running `tasksRunner` once per minute and processing all pending work in one pass, every dirty tile receives at most one write per cycle regardless of how many feedbacks landed on it. **This is the actual function of the queue, not throughput throttling.** Any redesign that moves Lachesis to event-driven dispatch must preserve per-tile coalescing, otherwise active areas will exceed the per-document write rate.

The 30s trigger lag on Clotho is the same pattern at finer grain. Note that **nothing currently serializes Clotho writes against Lachesis writes on the same tile** — that race exists today, but is rare because POI lifecycle changes and feedback bursts seldom coincide on the same tile.

A second cost: Clotho and the `poi-tiles` triggers **dual-write to both `tiled_views` (current) and `poi_tiles` (legacy)** for backward compatibility with old client versions. Every tile update therefore costs two Firestore writes and doubles the contention surface on hot tiles.

### Known divergences from market standards

| Aspect | Current | Market standard | Tracked |
|---|---|---|---|
| Queue backend | RTDB (home-grown) | Cloud Tasks | DEBT-003 |
| Recurring jobs | Self-scheduling + Zeus watchdog | Cloud Scheduler directly | DEBT-003 |
| Hermes trigger | Polling on a schedule | `onDocumentCreated` event trigger | DEBT-004 |
| Lachesis trigger | Polling on a schedule | Event-driven with per-tile debouncing¹ | DEBT-004 |
| `tiled_views` writers (Clotho + Lachesis + Hera) | Implicit serialization via cycle | Single tile reconciler with named-task dedup | DEBT-004 |
| Legacy `poi_tiles` dual-write | Active for old client compat | Retire after client version sunset | DEBT-005 |
| Throttling | Custom adaptive algorithm | Managed queue rate limits | DEBT-003 |
| Worker design | Single-responsibility, idempotent | ✓ Matches standard | — |
| Priority tiers | Buffer / tasks split | ✓ Matches standard | — |

¹ Naive `onValueCreated` would break the per-tile write serialization invariant. See "The hidden invariant" above.

### Cost profile

**Cloud Scheduler** ($0.10/job/month, 3 free) is not a meaningful cost. Replacing Zeus + self-scheduling with direct Cloud Scheduler jobs costs ~$0.30/month.

**The actual cost inefficiency is the polling pattern.** Hermes and Lachesis each pay for a Firestore/RTDB read on every cycle regardless of whether there is work to do. For a sparse-write app like a nature-sighting reporter, this means paying to find nothing most of the time. Event-driven triggers fire only on actual writes — zero cost at idle.

**The legacy `poi_tiles` dual-write doubles cost on every tile update.** Retiring this once the supported client floor allows would halve the write contention on every active tile and is independent of any architectural migration.

**Where the current system saves money** relative to naive alternatives: the `concurrency: 1` constraint prevents write contention retries; the cycle-based dispatch serializes per-tile writes on `tiled_views`; batching reduces invocation count. The queue design is sound — the trigger model for polling workers is the weak spot, but replacing it requires preserving the per-tile serialization invariant, not just changing the trigger mechanism.

### Remediation priority

1. **DEBT-005** (retire `poi_tiles` dual-write): cheapest, lowest risk, halves tile-write surface independently of any architectural change. Pre-requisite check: minimum supported client version.
2. **DEBT-004 (Hermes case)**: self-contained, no contention concerns. Switch to `onDocumentCreated` on `news_roll`. No dependency on DEBT-003.
3. **DEBT-006** (Lachesis delete-before-update bug): trivial fix, do before any Lachesis refactor.
4. **DEBT-004 (Lachesis case)**: requires per-tile debouncing. Two paths — either extend `addUniqueWorkerFrom` with per-tile keys (stays in current architecture), or migrate to Cloud Tasks named tasks (couples with DEBT-003). The Cloud Tasks path also opens the door to unifying Clotho + Lachesis into one per-tile reconciler.
5. **DEBT-003** (RTDB queue → Cloud Tasks): largest migration, but solves Lachesis-tile-reconciler cleanly and eliminates Zeus + self-scheduling fragility.

See `daen-fb-workers/CLAUDE.md` for the full DEBT entries with implementation steps.

---

## Adding a New Worker

1. Create `functions/workers/daen_workers/{name}.js`
2. Export from `functions/workers/daen_workers/index.js`
3. Document in `functions/workers/daen_workers/CLAUDE.md`
4. If always-on: add to the `pantheon` array in `zeus.js`
5. If event-driven: schedule from the appropriate trigger in `functions/db-triggers/`
