# POI Lifecycle

End-to-end flow of a POI from user creation in daen-scout through backend processing in daen-fb-workers to map display and push notification. See `POI lifecycle.drawio` for the visual diagram.

---

## Stage 1 — Creation (daen-scout)

User reports a sighting on the map.

1. `ReportMap` calls `POIstore.addPOI()`
2. `POIstore._enrichPOI()` normalises and validates the POI data
3. `FirebaseBackend.js` writes the POI document to `fs:POIs/{poiId}`
4. `fb-user.js` calls `addPOI2creator()` → `addPOI2user()` to link the POI to the creator's profile

**Data written:** `fs:POIs/{poiId}` with `status: 'new'`, `views[]` (pre-calculated tile set), `creator_id`, `created_t`

---

## Stage 2 — Firestore triggers queue tile work (daen-fb-workers)

Three Cloud Function triggers watch the `POIs` collection:

| Trigger | Fires on | Action |
|---|---|---|
| `poi-poiCreated` | New document | Schedules clotho for each tile in `views[]` |
| `poi-poiUpdated` | Any field change | Schedules clotho for affected tiles; also writes to `news_roll` on new log items |
| `poi-poiDeleted` | Document deletion | Removes POI from creator's profile; schedules clotho for previously associated tiles |

Tasks are pushed to `rtdb:/tasks` with a 30-second lag (`config.trigger.poi.lagAfterUpdate`) to batch rapid consecutive updates into a single tile rebuild.

`poi-poiUpdated` also handles two additional concerns independently:
- **Creator news feed** — appends new log items to `users/{creatorId}/news_roll` so the creator sees activity on their POI
- **Handled notification** — sends a direct push alert to the creator when `status` transitions to `handled` by a different user

---

## Stage 3 — Tile materialisation (Clotho)

Clotho is a per-tile worker that applies business rules to every POI in a tile and updates the `tiled_views` materialized view.

**For each POI in the tile:**
- Calculates `fading` (time-based decay), `credibility` (feedback score), `opacity` (fading × credibility)
- Applies status transitions:

```
new → ageing → locked → archived / disbelieved
```

| Condition | Outcome |
|---|---|
| Age > `maxAge` OR fading > threshold | Moved to `POIs_attic`, removed from tile |
| Credibility below threshold | `disbelieved` → moved to `POIs_attic` |
| Opacity below `lockThreshold` | `locked` (still visible, faded) |
| Otherwise | `ageing`, opacity updated |

**Writes:**
- `fs:tiled_views/{tileId}` — updated `poi_lists` for the tile (map clustering layer)
- `fs:POIs_attic/{poiId}` — archived/disbelieved POIs
- `rtdb:/tasks` — self-reschedule for next cycle

Clotho also maintains backward compatibility with the legacy `poi_tiles` collection for older client versions.

---

## Stage 4 — Tile update triggers map sync and feed fanout

When `tiled_views/{id}` is written, the `tiles-poiTileUpdated` trigger fires:

1. **Feed fanout** — pushes `poi-new` events to `rtdb:/feeds/{tileId}/logs` for new POIs appearing in `poi_lists`
2. **RTDB sync** — the RTDB write propagates to connected mobile clients, updating the map live without polling
3. **Clotho scheduling** — re-schedules a clotho if the `_clotho` flag is set on the tile document

**Hera** runs as a background sweep (hourly via Zeus) to catch any tiles that missed a clotho run — it scans all tiles and schedules clotho for any with POIs but no pending task.

---

## Stage 5 — Subscription notification pipeline

`feeds-notifySubs` and `subs-triggerSubsAlerts` form a two-stage fanout:

1. **`feeds-notifySubs`** — on each new `rtdb:/feeds/{feedId}/logs` item, reads all registered subscribers for that feed and copies matching events into `rtdb:/subs/{subsId}/logs` (filtering by `trigger` type and optional `opts.code`)

2. **`subs-triggerSubsAlerts`** — on any write to `rtdb:/subs/{subsId}`, evaluates filter criteria (geolocation distance, `firstof` mode, wildcard) and sends a direct push alert via `notifs.sendUserAlert()` if criteria are met

Subscription `mode` controls lifecycle: `once` (auto-deletes after first trigger), `firstof` (tracks fired subtypes), `all` (recurring).

---

## Stage 6 — Aggregated notifications (Hermes)

For activity that accumulates over time (likes, views, confirmations on a creator's POI), Hermes handles batched notification delivery:

1. `poi-poiUpdated` writes new log items to `users/{creatorId}/news_roll`
2. Hermes polls `news_roll` (collection group, `status == 'new'`) continuously
3. Aggregates per user: total `plus`, `tup`, `notseen`, list of affected POIs
4. Sends one consolidated push notification per user via Expo SDK
5. Schedules `hermes-cleaner` for receipt retrieval

Hermes is a polling worker kept permanently scheduled by Zeus — see [Worker System](../backend/Worker System.md).

---

## Full cross-repo flow

```
daen-scout                    daen-fb-workers                     daen-scout
──────────                    ───────────────                     ──────────
User reports POI
  → POIstore.addPOI()
    → fs:POIs/{id}
                         → poi-poiCreated trigger
                              → rtdb:/tasks (clotho)
                         → Clotho worker
                              → fs:tiled_views/{tile}
                         → tiles-poiTileUpdated
                              → rtdb:/feeds/{tile}/logs
                              → rtdb (RTDB sync)        → Map updates live
                         → feeds-notifySubs
                              → rtdb:/subs/{sub}/logs
                         → subs-triggerSubsAlerts
                              → Expo push              → Subscription alert
                         → poi-poiUpdated (on activity)
                              → users/{id}/news_roll
                         → Hermes (polling)
                              → Expo push              → Creator activity digest
```

---

## Key data stores

| Store | Path | Role |
|---|---|---|
| Firestore | `POIs/{id}` | Canonical POI business object |
| Firestore | `tiled_views/{tileId}` | Materialized clustering layer for map |
| Firestore | `POIs_attic/{id}` | Permanent archive of expired/disbelieved POIs |
| Firestore | `users/{uid}/news_roll` | Pending notifications for Hermes |
| RTDB | `/buffer`, `/tasks` | Worker task queues |
| RTDB | `/feeds/{tileId}/logs` | Raw feed events per tile |
| RTDB | `/subs/{subsId}/logs` | Per-subscriber filtered events |
| RTDB | `/feedbacks` | Pending feedback records for Lachesis |
