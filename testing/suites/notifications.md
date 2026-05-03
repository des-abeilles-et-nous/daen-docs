<!--
  Suite: Notifications & Follow-up  (testing/suites/notifications.md)
  ───────────────────────────────────────────────────────────────────
  PURPOSE : Functional Test (FUNC) suite
  ORIENTATION: User-Facing / Domain Behavior — tests notification subscriptions and delivery
  
  These tests validate end-to-end notification workflows: user subscribes to alerts,
  new POIs or feedback trigger notifications, and users receive push notifications.
-->

# Suite: Notifications & Follow-up

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 👤 **Functional Test (FUNC)** — User-Facing / Domain Behavior
> 
> This suite validates **notification subscription and delivery workflows**, not infrastructure.
> Focus: subscription creation, notification triggers, push notification delivery, user engagement.
>
> **Jira plan section:** Notifications & Follow-up — prefix `NOTIF`
> **TC ID prefix:** `TC-NOTIF`
> **Jira task type:** Story or Tâche
> **Reference docs:**
> - [`data model/`](../../data%20model/) — subscription, alert schemas
> - [`backend/Worker System.md`](../../backend/Worker%20System.md)

This suite validates that users can subscribe to alerts (e.g., bee-related observations), that notifications are triggered when relevant POIs are created, and that push notifications are delivered via Expo.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (Notification Workflows)
- **Subscription creation** — user subscribes to an alert (e.g., "hornets in my area")
- **Subscription types** — subscriptions for different alert types (blooming, hornet, nest, etc.)
- **Geospatial filtering** — subscriptions filtered by user's reference location and radius
- **Notification trigger** — new POI matching subscription criteria triggers a notification
- **Feedback trigger** — significant feedback on an existing POI triggers a notification
- **Push notification delivery** — Expo sends push notification to user's device
- **Notification content** — notification title, body, and deep link are correct
- **User engagement** — user can tap notification and be taken to relevant POI
- **Subscription management** — user can view, enable/disable, or delete subscriptions

### ❌ Out of Scope (Infrastructure & Worker Details)
- **Worker implementation** — tested in `[WORKER]`
- **Pub/Sub mechanism** — tested in `[WORKER]`
- **Security rules** — tested in `[FB]`

---

## Test Cases

---

### TC-NOTIF-001 — User creates a subscription to "hornet" alerts in their area

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase, expo-build

**Preconditions:**
- User is logged in.
- User has set a reference location (home location for alerts).
- Push notifications enabled on the device.

**Steps:**
1. Navigate to "Alerts" or "Subscriptions" screen.
2. Tap "Create new alert".
3. Select alert type: "hornet".
4. Set the radius (e.g., "5 km from home").
5. Set notification preferences (e.g., "all hornet sightings", "only verified", etc.).
6. Tap "Create subscription" button.
7. Verify a confirmation message: "Alert created".
8. Verify the subscription appears in the subscriptions list.
9. Verify a subscription document was created in Firestore `users/{uid}/subscriptions/` or in `rtdb:/subs/`.

**Expected result:**
- Subscription is created with correct parameters.
- Subscription is stored in Firestore/RTDB.
- User sees the subscription in their list.

---

### TC-NOTIF-002 — Notification triggered: new POI matches user's subscription

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase, expo-build

**Preconditions:**
- User has an active "hornet" subscription.
- User's reference location is set (e.g., Paris).

**Steps:**
1. From a test account or admin, create a new POI:
   - Category: "hornet"
   - Location: within 5 km of the user's reference location
2. Submit the POI.
3. Wait for the worker to process and trigger notifications (~5-10 seconds).
4. Verify the user receives a push notification on their device.
5. Verify the notification contains:
   - Title: "Hornet alert"
   - Body: POI category and location (e.g., "Hornet sighting near Eiffel Tower")
   - Notification icon and sound
6. Tap the notification and verify it opens the POI detail screen.

**Expected result:**
- Push notification is triggered.
- Notification contains correct information.
- Tapping notification opens the relevant POI.

---

### TC-NOTIF-003 — Notification NOT triggered: new POI outside subscription radius

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User has a "hornet" subscription with 5 km radius from Paris.

**Steps:**
1. Create a new hornet POI in Lyon (>100 km from Paris).
2. Wait for worker processing.
3. Verify the user does NOT receive a notification (or receives one marked as "outside radius").

**Expected result:**
- Notifications respect the geospatial radius filter.
- Users don't receive irrelevant notifications.

---

### TC-NOTIF-004 — Notification NOT triggered: new POI doesn't match subscription type

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User has a "hornet" subscription only (no "blooming" subscription).

**Steps:**
1. Create a new "blooming" POI within the user's subscription radius.
2. Wait for processing.
3. Verify the user does NOT receive a notification.

**Expected result:**
- Notifications respect the alert type filter.
- Users receive alerts only for subscribed types.

---

### TC-NOTIF-005 — User views their active subscriptions

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has created multiple subscriptions (e.g., hornet, nest, blooming).

**Steps:**
1. Navigate to "Alerts" or "Subscriptions" screen.
2. Verify a list of all active subscriptions is displayed.
3. For each subscription, verify:
   - Alert type (hornet, nest, blooming, etc.)
   - Radius and reference location
   - Status (active, paused, etc.)
   - Number of recent notifications (if available)
4. Tap on a subscription to view details.

**Expected result:**
- User can view all their subscriptions.
- Subscription details are complete and accurate.

---

### TC-NOTIF-006 — User disables a subscription temporarily

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has an active subscription.

**Steps:**
1. Open the subscription details.
2. Tap the "Disable" or toggle the "Enable/Disable" switch.
3. Verify the subscription status changes to "inactive" or "paused".
4. Create a new POI matching the (now-disabled) subscription.
5. Wait for processing.
6. Verify the user does NOT receive a notification.
7. Re-enable the subscription.
8. Create another matching POI.
9. Verify the user receives a notification again.

**Expected result:**
- User can disable/enable subscriptions.
- Disabled subscriptions don't trigger notifications.
- Re-enabling subscriptions restores functionality.

---

### TC-NOTIF-007 — User deletes a subscription

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has a subscription.

**Steps:**
1. Open subscription details.
2. Tap "Delete subscription" button.
3. Confirm deletion.
4. Verify the subscription is removed from the list.
5. Create a matching POI.
6. Wait for processing.
7. Verify the user does NOT receive a notification (subscription is gone).

**Expected result:**
- User can delete subscriptions.
- Deleted subscriptions no longer trigger notifications.

---

### TC-NOTIF-008 — Notification triggered: significant feedback on existing POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User subscribed to "hornet" alerts.
- An existing hornet POI was created by another user.

**Steps:**
1. Have multiple test users submit feedback on the existing POI (e.g., "✓ Handled").
2. After a certain threshold (e.g., 3 confirmations), a notification may be triggered.
3. Wait for processing.
4. Verify if a notification is sent (depends on app's follow-up policy).
5. If notification is sent, verify it informs the user about the update to the POI.

**Expected result:**
- Significant activity on POIs triggers follow-up notifications.
- Users are informed about POI updates relevant to their interests.

---

### TC-NOTIF-009 — Notification content: deep link correctly opens the POI

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, daen-fb-workers, firebase, expo-build

**Preconditions:**
- User receives a notification (from previous test).

**Steps:**
1. Receive a notification on the device.
2. Tap the notification.
3. Verify the app opens (if closed) or comes to foreground (if in background).
4. Verify the app navigates directly to the relevant POI detail screen.
5. Verify the POI ID in the URL or app state matches the notified POI.

**Expected result:**
- Deep link correctly identifies and opens the relevant POI.
- User can immediately see the details of the notified POI.

---

### TC-NOTIF-010 — Notification history: user can see recent notifications

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has received multiple notifications.

**Steps:**
1. Navigate to "Notification history" or "Alerts" screen.
2. Verify a list of recent notifications is displayed (last 30 days or configurable).
3. For each notification, verify:
   - Title and body
   - Timestamp
   - Associated POI (or alert type)
   - Read/unread status
4. Tap a notification to open the associated POI.

**Expected result:**
- User can view notification history.
- Notification metadata is complete.
- User can navigate to POI from history.

---

### TC-NOTIF-011 — Notification opt-out: user can disable push notifications globally

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has subscriptions with push notifications enabled.

**Steps:**
1. Navigate to Settings → Notifications.
2. Find the toggle for "Push notifications".
3. Tap to disable push notifications globally.
4. Verify the setting is saved.
5. Create a new POI matching user's subscription.
6. Wait for processing.
7. Verify the user does NOT receive a push notification (even though subscription is active).
8. Re-enable push notifications.
9. Verify notifications are sent again.

**Expected result:**
- User can disable push notifications globally.
- Subscriptions remain active but don't send notifications.
- Re-enabling restores notification delivery.

---

### TC-NOTIF-012 — Notification frequency: user can set notification frequency (e.g., daily digest)

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P3
**Component:** daen-scout, daen-fb-workers, firebase

**Preconditions:**
- User has subscriptions.

**Steps:**
1. Open notification settings.
2. Set notification frequency: "Daily digest" (instead of immediate notifications).
3. Save settings.
4. Create 3 matching POIs in quick succession.
5. Verify the user receives only ONE notification (a daily digest), not 3 individual ones.
6. Verify the digest notification lists all 3 POIs.

**Expected result:**
- User can set notification frequency.
- Multiple POIs are aggregated into a digest notification.
- Reduces notification noise while keeping user informed.
