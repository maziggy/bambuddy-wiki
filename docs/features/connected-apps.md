---
title: Connected Apps
description: Let external applications sign people in with their Bambuddy account ("Sign in with Bambuddy")
keywords:
  - connected apps
  - sign in with bambuddy
  - single sign-on
  - sso
  - oauth
  - pkce
  - client id
  - client secret
  - integration
---

# Connected Apps

A connected app is an external application that lets people sign in with their Bambuddy account instead of a separate password. When the app is opened from Bambuddy's sidebar, sign-in happens without the user doing anything.

The app never receives the user's password or Bambuddy session. It receives the user's name, email address, groups and permissions, once per sign-in, and decides from those what the person may do in the app.

!!! note "Requires authentication"
    Connected apps only work while Bambuddy authentication is enabled. With authentication off there are no accounts to sign in with, so apps must use their own login.

---

## :material-link-variant: Adding an App

You need the `settings:update` permission (administrators have it).

1. Go to **Settings** > **API Keys** > **Connected Apps**.
2. Enter a name and the app's **callback URL**. The app's documentation gives this URL; it must match exactly, including the port and path.
3. Click **Add app**.
4. Copy the **client ID** and **client secret** into the app's settings.

!!! warning "The secret is shown once"
    The client secret is displayed only right after adding the app (or creating a new secret). If it is lost, click **New secret** and paste the new one into the app. The old secret stops working immediately.

To show the app in Bambuddy's sidebar, add it as an [external link](external-links.md).

### What users see

The first time someone signs in to an app, Bambuddy asks them once:

> *Order Desk wants to sign you in. Signed in to Bambuddy as martin.*

They choose **Allow** or **Cancel**. After that, sign-in is automatic. If someone isn't logged in to Bambuddy yet, they see Bambuddy's normal login page first, including two-factor authentication, OIDC and LDAP if configured.

### Managing apps

| Action | Effect |
|---|---|
| **Disable** | Nobody can sign in through the app until it is enabled again. |
| **New secret** | Replaces the secret. The app stops working until it has the new one. |
| **Delete** | Removes the app and everyone's consent. Adding it again asks everyone again. |

Removing a user's account or disabling it also stops their sign-in to every app.

---

## :material-code-braces: For App Developers

Bambuddy implements the OAuth 2.0 authorization-code flow with PKCE (RFC 7636), limited to what an app on the same network needs to identify a user. It does not issue access tokens: the result of a sign-in is the user's identity, and the app keeps its own session. To call Bambuddy's API, the app uses an [API key](api-keys.md).

### 1. Check that sign-in is available

`GET /api/v1/auth/status` returns `auth_enabled`. If it is `false`, don't start the flow; use your own login.

### 2. Send the browser to Bambuddy

```
GET {bambuddy}/connect/authorize
    ?client_id=bba_...
    &redirect_uri={your registered callback URL}
    &state={random value you check on return}
    &code_challenge={base64url(SHA-256(code_verifier)), no padding}
    &code_challenge_method=S256
```

Only `S256` is accepted. `redirect_uri` must be exactly the registered URL.

### 3. Handle the callback

Bambuddy sends the browser to your callback URL with either:

- `?code=...&state=...` on success, or
- `?error=access_denied&state=...` if the user clicked Cancel.

Check that `state` matches what you sent. If the request itself was invalid (unknown app, disabled app, wrong callback URL), Bambuddy shows an error page and does **not** redirect.

### 4. Exchange the code (server side)

```http
POST {bambuddy}/api/v1/connect/token
Content-Type: application/json

{
  "grant_type": "authorization_code",
  "code": "...",
  "redirect_uri": "{your registered callback URL}",
  "client_id": "bba_...",
  "client_secret": "bbs_...",
  "code_verifier": "..."
}
```

Response:

```json
{
  "user": {
    "id": 7,
    "username": "martin",
    "email": "martin@example.com",
    "is_admin": false,
    "groups": ["Operators"],
    "permissions": ["queue:create", "queue:read_all", "..."]
  },
  "issued_at": "2026-09-26T10:00:00Z"
}
```

Errors:

| Status | `detail` | Meaning |
|---|---|---|
| 401 | `{"error": "invalid_client"}` | Unknown client ID, wrong secret, or app disabled |
| 400 | `{"error": "invalid_grant"}` | Code unknown, expired, already used, issued to another app, callback URL or verifier mismatch, or the user was disabled |
| 400 | `{"error": "auth_disabled"}` | Bambuddy authentication is off |
| 429 | | Too many failed exchanges; wait and retry |

### Rules a code follows

- Valid for **60 seconds**, usable **once**. A failed exchange spends it.
- Bound to the app it was issued to, its registered callback URL and the PKCE challenge.
- Redeemable only with the app's client secret, which never leaves the app's server.
- Bambuddy stores only a hash of the code and of the secret.

---

## :material-help-circle: Troubleshooting

**"Can't sign you in": this sign-in request isn't valid**
:   The app is not registered, is disabled, or sent a callback URL that differs from its registration. Compare the URL character by character, including `http`/`https`, the port and any trailing `/`.

**The app says the exchange failed with `invalid_client`**
:   The secret in the app is wrong or was replaced. Create a new secret and paste it into the app.

**Sign-in inside Bambuddy's sidebar shows the login page every time**
:   Bambuddy's login lasts for the browser tab. Tick **Remember Me** at login to keep it across tabs.
