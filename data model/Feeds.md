# Feeds and Events

The "feed" concept covers two related-but-distinct stores. Both sit in the path between a domain change (a POI is created, a user gives feedback) and a push notification on a user's phone.

For raw JSON shape sketches, see [mdd daen-scout.md](mdd%20daen-scout.md).

---

## Two stores, two roles

The system splits responsibilities along the same CQRS-style line as the [POI lifecycle](POI%20lifecycle.md):

| Store | Path | Role |
|---|---|---|
| **Tile feed** | `rtdb:/feeds/{feedId}` | **Event source.** Append-only log of domain events (`poi-new`, `poi-update`, …) per geographic tile, with a reverse index of subscriptions listening to that tile. |
| **News roll** | `fs:users/{uid}/news_roll` | **Per-user notification queue.** Aggregated, delivery-ready items consumed by the Hermes worker and turned into Expo push notifications. |

The tile feed is the **fan-out point** (one event → many subscribers). The news roll is the **fan-in point** (many events touching a user's POIs → one digest notification). Both feed into the user's phone, but via different paths.

---

## Tile feed — `rtdb:/feeds/{feedId}`

### Identity

`feedId` is a tile coordinate string like `"1|23|45"`, derived from `POI.tileIdFromLatLon()`. Tiles are not pre-registered; a feed exists exactly when the first event is appended into it.

### Shape

```text
rtdb:/feeds/{feedId}/
├── logs/
│   └── {logId}: {event, params, at, status?}
└── subs/
    └── {subsId}: {trigger, opts}
```

| Field | Type | Description |
|---|---|---|
| `logs.{logId}` | object | An event log entry. Keyed by `push()` UID. |
| `logs.{logId}.event` | string | Event type: `"poi-new"`, `"poi-update"`, … |
| `logs.{logId}.params` | object | Event-specific parameters (see [Event payloads](#event-payloads)). |
| `logs.{logId}.at` | number | Event timestamp (ms). |
| `logs.{logId}.status` | string | Optional status (`"new"`, etc.). Validated by rules but rarely read in code — see open questions. |
| `subs.{subsId}` | object | Listener registration. Mirror of [Subscriptions](Subscriptions.md). |
| `subs.{subsId}.trigger` | string | Copied from the subscription. |
| `subs.{subsId}.opts` | string \| object | Copied from the subscription. |

**Sources:** [`db-triggers/feeds.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/db-triggers/feeds.js), [`db-triggers/subs.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/db-triggers/subs.js) (lines ~12–23, ~115–125), [`database.rules.json`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/database.rules.json) (lines ~47–59).

### Event payloads

The set of event types is open-ended (any string is allowed) but the codebase produces these:

| `event` | Triggered by | Typical `params` |
|---|---|---|
| `poi-new` | POI document created | `{code, poi, lat, lon}` |
| `poi-update` | POI status/score change, feedback applied | `{code, poi, ...}` (subset of the above) |

Events are minimal by design — subscribers do their own filtering against `params.code` (substring match) and `params.lat/lon` (haversine distance). See [Subscriptions § Filters](Subscriptions.md#filters).

### Lifecycle

1. **Append.** A POI trigger (`poi-poiCreated`, `poi-poiUpdated`) calls `persist.addPOIlogitem()` to write `rtdb:/feeds/{tileId}/logs/{eventId}`.
2. **Fan-out.** The append fires `feeds-notifySubs`, which:
   - Reads `feeds/{tileId}/subs/*` (the listener index).
   - Performs the coarse trigger/opts match (substring on `params.code`).
   - Copies the matching event into `subs/{subsId}/logs/{eventId}` for each listener.
3. **Subscription evaluation.** That append re-fires `subs-triggerSubsAlerts` for each affected subscription, which applies the fine geographic filter and sends the push (see [Subscriptions § Lifecycle](Subscriptions.md#lifecycle)).
4. **Cleanup.** Currently no archival or pruning — see open questions.

### Triggers

| Trigger | Path | Action | File |
|---|---|---|---|
| `notifySubs` | `rtdb:/feeds/{feedId}/logs/{logId}` (create) | Fan-out to subscribed subs | `db-triggers/feeds.js` (~14–36) |

---

## News roll — `fs:users/{uid}/news_roll`

### Identity

A Firestore subcollection under each user document. Each document is one notification-candidate item destined for that user.

### Shape

| Field | Type | Description |
|---|---|---|
| `at` | number | Event timestamp (ms). |
| `status` | string | `"new"` (pending) → `"notified"` (sent to Expo). |
| `source` | string | Origin: `"poi-update"`, `"poi_lifecycle"`, `"feedback"`. Used for filtering / debugging. |
| `news` | array | Optional. Aggregate items (see below). |
| `news[].poi` | string | POI UID. |
| `news[].plus` | number | Count of `PLUS` ("also seen") feedbacks on the POI. |
| `news[].tup` | number | Count of `THUMBUP` votes. |
| `news[].notseen` | number | Count of `NOTSEEN` reports. |

**Sources:** [`db-triggers/poi.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/db-triggers/poi.js) (lines ~65–82), [`workers/daen_workers/hermes.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/workers/daen_workers/hermes.js) (lines ~11–73), [`lib/daen-firebase/fb-user.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/lib/daen-firebase/fb-user.js) (lines ~51–59).

### Lifecycle

1. **Write.** When a creator's POI is touched (status change, feedback applied), `poi-poiUpdated` extracts the new entries from the POI's `log_roll` and appends one `news_roll` item per affected POI: `{at, news, source, status: "new"}`.
2. **Poll.** The Hermes worker periodically scans `firestore.collectionGroup('news_roll').where('status', '==', 'new')` (see [Worker System](../backend/Worker%20System.md) for the polling cadence and the planned event-driven migration in DEBT-004).
3. **Aggregate.** For each user, Hermes consolidates pending items into a single summed digest (totals of `plus`/`tup`/`notseen` across affected POIs).
4. **Notify.** Hermes sends a single Expo push per user (not per item) and stores the receipt ticket. The `hermes-cleaner` worker is scheduled to retrieve receipts later (implementation incomplete — see open questions).
5. **Mark.** Each delivered item's `status` flips to `"notified"`. It stays in Firestore as historical record.

### Why polled (not event-driven)?

Polling lets Hermes coalesce multiple feedback hits on a user's POIs into a single notification, instead of paging the user once per feedback. A direct `onDocumentCreated` trigger on `news_roll` would over-notify. Event-driven dispatch with a debounce window is on the roadmap (DEBT-004 in [`daen-fb-workers/CLAUDE.md`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/CLAUDE.md)).

---

## End-to-end: from POI change to push notification

Two parallel paths, both originating from a POI write:

```
                        ┌─────────────────────────┐
                        │  POI created or updated │
                        └────────────┬────────────┘
                                     │
                ┌────────────────────┴────────────────────┐
                │                                         │
                ▼                                         ▼
   ┌──────────────────────────┐               ┌──────────────────────────┐
   │ Path A — Tile feed       │               │ Path B — Creator news    │
   │ (fan-out to subscribers) │               │ (fan-in to creator)      │
   └────────────┬─────────────┘               └────────────┬─────────────┘
                │                                          │
   feeds/{tile}/logs.push                       users/{creatorId}/news_roll.add
                │                                          │
   notifySubs trigger                           Hermes worker (polled)
                │                                          │
   subs/{subsId}/logs.append                    aggregate per user
                │                                          │
   triggerSubsAlerts                            Expo push (digest)
                │                                          │
   filter pass → Expo push                      mark status: "notified"
   (per-subscription)
```

Path A is **subscriber-facing**: people who asked to be alerted about a tile. Path B is **creator-facing**: the user who reported the POI gets an activity digest.

---

## Cross-references

| To | Via | Purpose |
|---|---|---|
| POI | `feeds/{feedId}/logs.{logId}.params.poi`, `news_roll.{}.news[].poi` | Events reference the POI that caused them. |
| Subscription | `feeds/{feedId}/subs/{subsId}` (reverse index), `subscription.feeds[]` (forward) | Subs register on feeds; feeds fan out to subs. |
| User | `users/{uid}/news_roll` (subcollection) | Per-user notification queue. |
| User | `users/{uid}.feeds[]` (Firestore field) | Per-user feedback history (double-feedback prevention) — distinct from news roll, despite the field name. |

---

## Open questions

1. **`logs.{}.status` field.** Validated by RTDB rules and described in `mdd daen-scout.md` but not read or written by `notifySubs` or `triggerSubsAlerts`. Possibly aspirational (per-event read state) or a remnant.
2. **`source` usage.** Only ever written by news-roll producers, never read in the surveyed code. Is it consumed by client-side filtering (e.g. to choose an icon)? Or pure debugging?
3. **`alerts_log` subcollection.** Referenced in `fb-user.js` (`ALERTLOG` constant, `addUserAlert()` helper) but no caller in current triggers/workers. Relationship to `news_roll` — sibling? predecessor? — unclear.
4. **Multiple feedbacks per update.** When several feedbacks land on a POI in the 30-second debounce window, `poi-poiUpdated` writes a single `news_roll` item with a `news[]` array. The boundary conditions (overlapping windows, feedbacks during a tile rebuild) are not clearly tested in the surveyed code.
5. **Hermes receipt cleanup.** `hermes-cleaner` is referenced as the receipt-retrieval worker, but its implementation is not in the files surveyed. Confirm whether Expo receipts are actually retrieved or merely scheduled.
6. **Feed retention.** `rtdb:/feeds/{tileId}/logs` grows monotonically. There is no cleanup worker; old events accumulate. Acceptable while the absolute volume is small, but worth tracking.
