<!--
  Suite: Authentication & User Sessions  (testing/suites/user-auth.md)
  ─────────────────────────────────────────────────────────────────────
  PURPOSE : Functional Test (FUNC) suite
  ORIENTATION: User-Facing / Domain Behavior — tests user authentication workflows
  
  These tests validate end-to-end user authentication workflows (signup, login, logout,
  social auth, password reset) and that user sessions are correctly established and managed.
-->

# Suite: Authentication & User Sessions

> ⚠️ **STATUS: WORK IN PROGRESS** — Under review, not finalized yet
>
> **Test Classification:** 👤 **Functional Test (FUNC)** — User-Facing / Domain Behavior
> 
> This suite validates **end-to-end user authentication workflows**, not infrastructure details.
> Focus: signup/login/logout workflows, social authentication, password reset, session management.
>
> **Jira plan section:** Authentication & User Sessions — prefix `AUTH`
> **TC ID prefix:** `TC-AUTH`
> **Jira task type:** Tâche or Story
> **Reference docs:**
> - [`data model/`](../../data%20model/) — user schema
> - [`ECOSYSTEM_CONTEXT.md` — Authentication](../../ECOSYSTEM_CONTEXT.md)

This suite validates that users can authenticate via email/password and social auth, that sessions are correctly established, and that password reset and logout workflows work as expected.

---

## Scope: What This Suite Tests (and What It Doesn't)

### ✅ In Scope (User Authentication & Sessions)
- **Email/password signup** — user registration with email and password
- **Email/password login** — user login with email/password credentials
- **Social authentication** — login via Facebook, Google (OAuth)
- **Session management** — Firebase Auth token generation, session persistence
- **Logout** — session termination and local state cleanup
- **Password reset** — email-based password reset workflow
- **Email verification** — email verification emails sent and links work
- **User profile creation** — user document created in Firestore after signup
- **Session persistence** — user stays logged in across app restarts
- **Invalid credentials** — login with wrong password/email shows appropriate error

### ❌ Out of Scope (Security & Infrastructure)
- **Security rules enforcement** — tested in `[FB]` (IT suite)
- **Token validation** — tested in `[FB]`
- **Rate limiting** — not covered (infrastructure concern)

---

## Test Cases

---

### TC-AUTH-001 — Email/password signup: new user can register

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- Beefree app running in `dev` or `staging` environment.
- Firebase Auth email/password provider enabled.

**Steps:**
1. Open the Beefree app and navigate to the signup screen.
2. Enter a new email address (not previously registered).
3. Enter a secure password.
4. Enter a display name (pseudo).
5. Tap "Sign up" button.
6. Verify the signup succeeds and the user is logged in.
7. Verify a user document was created in Firestore `users` collection with the correct data.

**Expected result:**
- User is registered with email/password.
- User is immediately logged in after signup.
- User profile document exists in Firestore.
- All required fields are populated (uid, email, display_name, created_t).

---

### TC-AUTH-002 — Email/password login: existing user can log in

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- A user account already exists.

**Steps:**
1. Close and restart the Beefree app (logout).
2. Navigate to the login screen.
3. Enter the email address of an existing user.
4. Enter the correct password.
5. Tap "Log in" button.
6. Verify the user is logged in and the home screen is displayed.

**Expected result:**
- User is logged in with correct credentials.
- No errors during login.
- User profile is accessible from the home screen.

---

### TC-AUTH-003 — Invalid password: login fails with appropriate error

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- A user account exists.

**Steps:**
1. Navigate to the login screen.
2. Enter the user's email address.
3. Enter an incorrect password.
4. Tap "Log in" button.
5. Verify the login fails with an error message.
6. Verify the error message is user-friendly: "Incorrect password" or "Invalid credentials" (not a stack trace).

**Expected result:**
- Login fails.
- Error message is clear and user-friendly.
- User is NOT logged in.

---

### TC-AUTH-004 — Non-existent email: login fails gracefully

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- No user exists with the test email.

**Steps:**
1. Navigate to the login screen.
2. Enter an email address that does not exist.
3. Enter any password.
4. Tap "Log in" button.
5. Verify the login fails with an error message (not a crash).
6. Error message should indicate "User not found" or "Invalid credentials".

**Expected result:**
- Login fails gracefully.
- No app crash.
- User-friendly error message.

---

### TC-AUTH-005 — Facebook login: user logs in via Facebook OAuth

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- Beefree app built for `dev` target.
- Facebook Auth configured in Firebase and app.
- Test Facebook app credentials available.

**Steps:**
1. Open the Beefree app.
2. Tap "Log in with Facebook" button.
3. Authorize the app to access Facebook profile (name, email, picture).
4. Verify the OAuth flow completes and user is logged in.
5. Verify a new user document is created in Firestore with Facebook profile data.
6. Verify the user's display_name and picture are populated from Facebook.

**Expected result:**
- Facebook login succeeds.
- User is logged in.
- User profile data is fetched from Facebook and stored.

---

### TC-AUTH-006 — Google login: user logs in via Google OAuth

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- Beefree app built for `dev` or `staging` target.
- Google Sign-In configured.
- Test Google credentials available.

**Steps:**
1. Open the Beefree app.
2. Tap "Log in with Google" button.
3. Complete the Google OAuth flow.
4. Verify the user is logged in.
5. Verify a user document is created with Google profile data.

**Expected result:**
- Google login succeeds.
- User profile populated with Google data.

---

### TC-AUTH-007 — Logout: user session is terminated and app resets

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- User is logged in.

**Steps:**
1. Navigate to account settings or menu.
2. Tap "Logout" or "Sign out" button.
3. Verify the user is logged out and returns to login screen.
4. Verify the user's profile is no longer accessible.
5. Verify local data (cached user profile, preferences) is cleared.
6. Attempt to navigate back without logging in — verify access is denied.

**Expected result:**
- User is logged out.
- Login screen is displayed.
- Local session data is cleared.

---

### TC-AUTH-008 — Session persistence: user stays logged in across app restarts

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P1
**Component:** daen-scout, firebase

**Preconditions:**
- User is logged in.

**Steps:**
1. User logs in and is on the home screen.
2. Close the Beefree app completely (kill the process).
3. Reopen the Beefree app.
4. Verify the user is still logged in (no login screen).
5. Verify the home screen is displayed.
6. Verify the user's profile data is accessible.

**Expected result:**
- User session persists across app restarts.
- User does not need to log in again.

---

### TC-AUTH-009 — Password reset: user can reset forgotten password via email

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User account exists with a valid email.
- Email service configured to send password reset emails.

**Steps:**
1. Navigate to login screen.
2. Tap "Forgot password?" link.
3. Enter the user's email address.
4. Tap "Send reset email" button.
5. Verify a confirmation message: "Password reset email sent."
6. Check email inbox for password reset email (may use test email service).
7. Click the reset link in the email.
8. Verify a password reset form appears in a browser or in-app.
9. Enter a new password and confirm it.
10. Tap "Reset password" button.
11. Verify password is reset and user can log in with the new password.

**Expected result:**
- Password reset email is sent.
- Reset link works and allows password change.
- User can log in with new password.

---

### TC-AUTH-010 — Email verification: new user receives verification email

**Type:** functional
**Scope:** system
**Jira type:** Story or Tâche
**Environment:** dev
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- Email verification enabled in Firebase Auth.

**Steps:**
1. Sign up a new user with email/password.
2. Verify the user is logged in but marked as "email unverified" internally.
3. Check email inbox for verification email.
4. Click the verification link in the email.
5. Verify the email is marked as verified in Firebase.
6. Verify the user's Firestore profile is updated with `email_verified: true`.

**Expected result:**
- Verification email is sent after signup.
- Verification link works.
- Email is marked as verified.

---

### TC-AUTH-011 — User profile fields: name, pseudo, picture are stored correctly

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev, staging
**Priority:** P2
**Component:** daen-scout, firebase

**Preconditions:**
- User has logged in via email or social auth.

**Steps:**
1. Open user profile screen.
2. Verify the display_name (full name) is displayed.
3. Verify the pseudo (username/nickname) is displayed.
4. Verify the profile picture (avatar) is displayed.
5. Edit the profile: change pseudo or upload a new picture.
6. Save changes.
7. Reload the profile screen.
8. Verify the changes persist.

**Expected result:**
- User profile fields are displayed correctly.
- Profile updates persist.
- Picture is correctly displayed (not broken).

---

### TC-AUTH-012 — Multi-account linking: user can link multiple social providers

**Type:** functional
**Scope:** repo
**Jira type:** Story or Tâche
**Environment:** dev
**Priority:** P3
**Component:** daen-scout, firebase

**Preconditions:**
- User logged in via one provider (e.g., Facebook).
- User account supports linking additional providers.

**Steps:**
1. Open account settings.
2. Navigate to "Connected accounts" or "Link accounts" section.
3. Tap "Link Google account" (or other provider).
4. Complete the OAuth flow for the second provider.
5. Verify the accounts are linked.
6. Log out.
7. Log in using the second provider (Google).
8. Verify the same user account is accessed (same profile data).

**Expected result:**
- Multiple providers can be linked to the same account.
- User can log in via either provider.
- Account data is consistent across providers.
