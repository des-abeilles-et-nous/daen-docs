# User Model

The user object stores profile data, identity references, and links to a user's POIs and subscriptions. It is the glue between Firebase Auth (identity) and the application's business state.

For raw JSON shape sketches, see [mdd daen-scout.md](mdd%20daen-scout.md) (Firestore section, `users` collection).

---

## Storage

| Location | Path | Purpose |
|---|---|---|
| Firestore | `users/{uid}` | Profile document, one per authenticated user. Key is the Firebase Auth `uid`. |
| Firestore subcollection | `users/{uid}/news_roll` | User-facing notification queue (see [Feeds](Feeds.md)) |
| Firestore subcollection | `users/{uid}/alerts_log` | Historical alerts log (referenced in code; see open questions) |

The user document is the only canonical source for profile data. Authentication metadata lives in Firebase Auth and is not duplicated here beyond `uid` and a few transcoded fields.

---

## Field shape

| Field | Type | Required | Description |
|---|---|---|---|
| `uid` | string | yes | Firebase Auth UID. Document key. |
| `email` | string | optional | Login email, copied from Auth provider. |
| `display_name` | string | optional | Name from OAuth provider. Rarely updated after creation. |
| `picture` | string | optional | Profile picture URL (OAuth or app-uploaded). |
| `pseudo` | string | optional | App-level display name (auto-generated from `public_id`, user-editable). Used as `creator_pseudo` denormalised onto POIs. |
| `public_id` | string | optional | Unique public identifier. Deprecated, not actively used. |
| `num_id` | number | optional | Sequential numeric ID, assigned at creation via `sequences/users` counter. |
| `created_t` | number | optional | Profile creation timestamp (ms). |
| `modified_t` | number | optional | Last profile modification timestamp (ms). Set via `serverTimestamp()`. |
| `isBeekeeper` | boolean | optional | Has active hives/apiaries. |
| `bkpr` | string | optional | Beekeeping activity kind, LOV-bound (e.g. `"hobbyist"`, `"professional"`). |
| `isHunter` | boolean | optional | Hornet hunter flag. |
| `isSponsor` | boolean | optional | Sponsor flag. Origin/usage unclear (see open questions). |
| `isRegistered` | boolean | optional | Set during Auth transcoding but never written by code (see open questions). |
| `refLocation` | object | optional | Reference position used for distance-based alerts. |
| `refLocation.latitude` | number | conditional | Latitude. |
| `refLocation.longitude` | number | conditional | Longitude. |
| `refLocation.isoCountryCode` | string | conditional | ISO 3166-1 country code. |
| `refLocation.postalCode` | string | conditional | Postal/zip code. |
| `pois` | string[] | optional | POI UIDs created by this user. Used to bulk-update `creator_pseudo` on POIs when `pseudo` changes. |
| `subs` | string[] | optional | RTDB subscription IDs the user owns. Removal triggers cascade delete of `rtdb:/subs/{subsId}`. |
| `feeds` | object[] | optional | Per-feedback log used to prevent double-feedback. Each item: `{at, poi, feed}` where `feed` ∈ `{PLUS, HANDLED, NOTSEEN, THUMBUP}`. |
| `alerts` | map | optional | User's alert subscriptions, keyed by alert type code. Each entry: `{status, subsid}` where `subsid` references `rtdb:/subs/{subsid}`. |

**Sources:**
- Field declarations: [mdd daen-scout.md](mdd%20daen-scout.md) (lines ~78–190)
- Auth-side transcoding: [`daen-scout/api/Auth.js`](https://github.com/des-abeilles-et-nous/daen-scout/blob/main/api/Auth.js) (`updateUser`, lines ~138–150)
- Backend triggers: [`daen-fb-workers/functions/db-triggers/users.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/db-triggers/users.js)

---

## Lifecycle

### 1. Creation

**Currently broken in Cloud Functions v2** — the v1 `onUserCreated` Auth trigger does not exist in v2. There is no active server-side handler that auto-creates the Firestore user document when a Firebase Auth user signs in for the first time. The `initPublicId` helper exists in [`users.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/db-triggers/users.js) (lines ~11–81) but is not currently wired to a trigger.

In practice, user documents are created on first profile read/write from the client (`Auth.updateUser` with `merge: true`). `num_id` and `pseudo` assignment via the sequences counter is currently **not happening on creation** — this is part of the v1→v2 migration gap.

### 2. Profile updates

Client edits in the daen-scout `MyAccount` screen call `Auth.updateUser(user)`:

1. Firestore `users/{uid}` set with `{merge: true}` and `modified_t: serverTimestamp()`.
2. `onUserChange` trigger fires (`users.js`, lines ~87–118):
   - If `pseudo` changed → `bulkPOIUpdate` updates `creator_pseudo` on every POI in `user.pois[]`.
   - If a `subsId` was removed from `subs[]` → corresponding `rtdb:/subs/{subsId}` is deleted (cascade).

### 3. POI linkage

When a user creates a POI in daen-scout:

1. POI document written to `fs:POIs/{poiId}` (with `creator_id = user.uid`).
2. `addPOI2creator()` → `addPOI2user()` appends `poiId` to `user.pois[]` ([`fb-user.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/lib/daen-firebase/fb-user.js), lines ~15–35).

This link enables the `pseudo`-change cascade above and lets admin tools enumerate a user's reports.

### 4. Subscription linkage

When a user creates a location-based alert:

1. `createUserSubscription()` writes to `rtdb:/subs/{subsId}` (see [Subscriptions](Subscriptions.md)).
2. `addSubs2user()` appends `subsId` to `user.subs[]` ([`fb-user.js`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/functions/lib/daen-firebase/fb-user.js), lines ~154–165).

Removing a `subsId` from `user.subs[]` is the canonical way to delete a subscription — the `onUserChange` trigger handles RTDB cleanup.

### 5. Notification reception

The Firestore subcollection `users/{uid}/news_roll` is the user's notification queue. It is written by `poi-poiUpdated` (creator activity feed) and read by the Hermes worker, which dispatches Expo push notifications and marks items as `notified`. See [Feeds](Feeds.md) for the full flow.

### 6. Deletion

**Currently incomplete** — see DEBT-002 in [`daen-fb-workers/CLAUDE.md`](https://github.com/des-abeilles-et-nous/daen-fb-workers/blob/main/CLAUDE.md) and [`daen-scout/CLAUDE.md`](https://github.com/des-abeilles-et-nous/daen-scout/blob/main/CLAUDE.md).

Today, `deleteUserAccount()` in daen-scout deletes only the Auth user. The Firestore `users/{uid}` document is orphaned, along with its `news_roll`, `alerts_log`, RTDB `subs/*`, and any feedbacks under `rtdb:/feedbacks` keyed by uid. Planned remediation: switch `delUserProfile` from the missing v2 `onUserDeleted` trigger to an `onDocumentDeleted` Firestore trigger and have the client delete the Firestore doc **before** the Auth user.

---

## Triggers

| Trigger | Path | Action | File |
|---|---|---|---|
| `onUserChange` | `users/{uid}` (write) | Cascades pseudo to POIs, deletes removed subs | `db-triggers/users.js` (~87–118) |

---

## Cross-references

| To | Via | Purpose |
|---|---|---|
| POI | `user.pois[]` (Firestore POI UIDs) | Bulk pseudo update on profile change |
| POI | `POI.creator_id` (denormalised back) | Reverse lookup; `creator_pseudo` cached on POI for read-time display |
| Subscription | `user.subs[]` (RTDB subscription IDs) | Owns subs; removal cascades to `rtdb:/subs/{subsId}` deletion |
| Subscription | `user.alerts[code].subsid` | Per-alert-type pointer into RTDB subs |
| Feeds | `users/{uid}/news_roll` subcollection | User-facing notification queue (see [Feeds](Feeds.md)) |

---

## Open questions

1. **`alerts` map activation flow.** The `alerts` field exists in the schema but the trigger that sets `alerts.{code}.status` and creates the corresponding `subsid` is not visible in the source code surveyed. Likely implemented client-side; needs client-code audit to confirm.
2. **`isRegistered`, `isSponsor` semantics.** Both are transcoded in `Auth.js` but never explicitly written by application code. Are they computed downstream, or remnants of a prior design?
3. **`news_roll` shape inconsistency.** `mdd daen-scout.md` describes `news_roll` as an inline field on the user document; the actual implementation is a Firestore subcollection. The doc should be reconciled — see [Feeds](Feeds.md) for the implemented shape.
4. **`alerts_log` subcollection.** Referenced in `fb-user.js` (`ALERTLOG` constant, `addUserAlert()` helper) but no caller found in current triggers/workers. May be dead code or pending feature.
5. **User creation flow under v2.** No active trigger creates the user document or assigns `num_id`/`pseudo`. Either the client is expected to do this on first sign-in, or it is silently broken. Confirm with daen-scout `Auth.js` flow.
