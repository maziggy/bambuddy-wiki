# Authentication

Bambuddy includes an optional authentication system that allows you to secure your instance with user accounts and group-based permissions. This feature is completely optional and can be enabled or disabled at any time.

## Overview

When enabled, authentication provides:

- **User Accounts**: Create multiple users with unique credentials
- **Group-Based Permissions**: 80+ granular permissions organized by feature
- **Customizable Groups**: Create custom groups or use default system groups
- **Secure Authentication**: JWT tokens with password hashing using PBKDF2
- **User Activity Tracking**: See who uploaded archives, library files, queued prints, and started prints
- **Advanced Auth via Email**: Optional SMTP-based user onboarding and self-service password resets

## Groups & Permissions

### Default Groups

Bambuddy comes with three default system groups:

| Group | Description | Permissions |
|-------|-------------|-------------|
| **Administrators** | Full access to all features | All permissions |
| **Operators** | Control printers and manage content | Printer control, queue, archives, projects, library |
| **Viewers** | Read-only access | View printers, archives, queue, projects |

### Permission Categories

Permissions follow a `resource:action` pattern. Categories include:

- **Printers**: read, create, update, delete, control, files, ams_rfid, clear_plate
- **Archives**: read, create, update_own, update_all, delete_own, delete_all, reprint_own, reprint_all
- **Queue**: read, create, update_own, update_all, delete_own, delete_all, reorder
- **Library**: read, upload, update_own, update_all, delete_own, delete_all
- **Projects**: read, create, update, delete
- **Inventory**: read, create, update, delete, view_assignments
- **Cloud**: auth (login/logout and read/write cloud profiles)
- **Settings**: read, update, backup, restore
- **Users/Groups**: read, create, update, delete
- And many more...

!!! tip "Cloud Profiles Are Per-User"
    When authentication is enabled, each user has their own independent Bambu Cloud login. User A logging into Cloud does not affect User B's session. To use Cloud Profiles, a user needs the `cloud:auth` permission — this single permission covers login, logout, and all cloud profile operations (reading, creating, editing, deleting presets). The `settings:read` permission is NOT required for cloud profiles.

!!! tip "Separating Inventory Access from AMS Assignments"
    The `inventory:view_assignments` permission controls whether spool-to-AMS-slot assignment data is visible on the Printers page. This is separate from `inventory:read`, which controls access to the full Inventory page. Grant only `inventory:view_assignments` to let users see what's loaded in each AMS slot without exposing the full spool inventory.

### Ownership-Based Permissions

For archives, queue items, and library files, permissions are split into "own" and "all" variants:

| Permission Type | Description |
|-----------------|-------------|
| `*_own` | User can only modify items they created |
| `*_all` | User can modify any item (includes `*_own` capability) |

**Examples:**

- `archives:delete_own` - Delete only archives you uploaded
- `archives:delete_all` - Delete any archive
- `queue:update_own` - Edit only queue items you added
- `library:update_all` - Rename/move any library file

**Default Group Assignments:**

| Group | Permissions |
|-------|-------------|
| Administrators | All `*_all` permissions (full access) |
| Operators | All `*_own` permissions (own items only) |
| Viewers | No update/delete permissions (read-only) |

**Ownerless Items:**

Items created before authentication was enabled (or by deleted users) have no owner. These "ownerless" items require `*_all` permission to modify.

### Users in Multiple Groups

Users can belong to multiple groups. Permissions are **additive** - a user has all permissions from all their groups combined.

## Enabling Authentication

### First-Time Setup

1. Navigate to any page in Bambuddy
2. You'll be redirected to the **Setup Page** if authentication is not configured
3. Choose to **Enable Authentication**
4. Create your admin account:
   - Enter a username
   - Enter a password (minimum 6 characters)
5. Click **Enable Authentication**

The first user is automatically added to the **Administrators** group.

### From Settings (When Already Running)

1. Go to **Settings** → **Users** tab
2. Click **Activate Authentication**
3. You'll be redirected to the Setup Page
4. Complete the setup as described above

## Managing Users

### Creating Users

#### Standard Mode

1. Log in as a user with `users:create` permission
2. Go to **Settings** → **Users** tab
3. Click **Add User**
4. Fill in:
   - Username
   - Password (minimum 6 characters)
   - Confirm Password
   - Groups (select one or more)
5. Click **Create**

#### With Advanced Auth (Email)

When [Advanced Auth via Email](#advanced-auth-via-email) is enabled:

1. Go to **Settings** → **Users** tab
2. Click **Add User**
3. Fill in:
   - Username
   - Email address
   - Groups (select one or more)
4. Click **Create** — the system generates a secure random password and emails it to the user automatically

No one besides the new user sees the password, making this inherently more secure than manually assigning passwords.

### Editing Users

1. Go to **Settings** → **Users**
2. Click the edit icon next to a user
3. Modify username, password, or group assignments
4. Click **Save**

### Deleting Users

1. Go to **Settings** → **Users**
2. Click the delete icon next to a user
3. If the user has created any archives, queue items, or library files, you'll be asked what to do:
   - **Delete user AND their items** - Removes the user and all content they created
   - **Delete user, keep items** - Removes the user but keeps their content (items become "ownerless")
4. Confirm deletion

Note: You cannot delete yourself or the last administrator. Ownerless items require `*_all` permission to modify.

## Managing Groups

### Viewing Groups

1. Go to **Settings** → **Users** → **Groups** tab
2. View all groups with their permission counts

### Creating Custom Groups

1. Go to **Settings** → **Users** → **Groups** tab
2. Click **Add Group** — this opens the full-page group editor
3. Enter group name and description
4. Use the permission grid to select permissions:
   - **Search**: Filter permissions by name using the search bar
   - **Select All / Clear All**: Bulk-select or deselect all permissions
   - **Category checkboxes**: Toggle all permissions in a category at once
   - Each category card shows a count badge (e.g., "5/7") for selected permissions
5. Click **Save**

### Editing Groups

1. Click the edit icon next to a group — this opens the full-page group editor
2. Modify name, description, or permissions
3. Click **Save**

Note: System groups (Administrators, Operators, Viewers) cannot be deleted.

### Adding Users to Groups

1. Go to **Settings** → **Users** → **Groups** tab
2. Click on a group to view details
3. Click **Add User** and select a user
4. Or edit a user and select their groups

## Changing Your Password

Any authenticated user can change their own password:

1. Click the **Key icon** in the sidebar (next to logout)
2. Enter your current password
3. Enter your new password
4. Confirm the new password
5. Click **Change Password**

## Forgot Password

### With Advanced Auth (Email)

If [Advanced Auth via Email](#advanced-auth-via-email) is enabled, users can reset their own password:

1. Click **"Forgot your password?"** on the login page
2. Enter your username or email address
3. A new secure random password is emailed to you automatically
4. Log in with the new password and change it if desired

Admins can also trigger a password reset from User Management with a single click — the new password is emailed to the user.

### Without Advanced Auth

If email-based auth is not enabled:

1. Contact your Bambuddy administrator
2. They can reset your password in User Management
3. Log in with the temporary password and change it

## Disabling Authentication

If you need to disable authentication:

1. Log in as an administrator
2. Go to **Settings** → **Users** tab
3. Click **Disable Authentication**
4. Confirm the action

**Warning**: Disabling authentication removes access control. All features become accessible without login.

## Advanced Auth via Email

Advanced Authentication adds SMTP-based email integration for streamlined user onboarding and self-service password management. This is an optional feature that can be enabled or disabled independently of basic authentication.

### Setting Up SMTP

1. Go to **Settings** → **Email** tab
2. Configure your SMTP server:
   - **SMTP Host** — Your mail server (e.g., `smtp.gmail.com`)
   - **SMTP Port** — Typically `587` (TLS) or `465` (SSL)
   - **Username** — SMTP login (if authentication is required)
   - **Password** — SMTP password or app-specific password
   - **From Address** — Sender email shown in outgoing messages
   - **External URL** — Your Bambuddy instance URL (used in email links)
3. Enable **Advanced Authentication**
4. Use the **Test Email** button to verify your configuration

### How It Works

Once enabled:

- **User creation**: Admins enter a username and email address. The system generates a secure random password and emails it directly to the user. No one else sees the password.
- **Admin password reset**: In User Management, admins can click a reset button to generate a new password and email it to the user — one click, no manual entry.
- **Self-service reset**: Users can click "Forgot your password?" on the login screen to receive a new password via email without contacting an admin.
- **Email validation**: The system validates email addresses since email is the sole mechanism for password delivery.
- **Case-insensitive login**: Usernames and email addresses are not case-sensitive when logging in.

### Email Templates

Bambuddy includes customizable notification templates for:

- **Welcome Email** — Sent when a new user account is created
- **Password Reset** — Sent when a password is reset (by admin or self-service)
- **User Print Started** — Sent when a user's print job begins
- **User Print Completed** — Sent when a user's print job finishes successfully
- **User Print Failed** — Sent when a user's print job fails
- **User Print Stopped** — Sent when a user's print job is cancelled

Templates can be edited in **Settings** → **Email** → **Templates**.

### Per-User Email Notifications

When Advanced Auth is enabled, individual users can opt in to email notifications for their own print jobs. This is separate from the global notification system — it only emails the user who submitted the print.

#### Enabling User Notifications

1. Go to **Settings** → **Notifications** tab
2. Enable **User Notifications**
3. SMTP must be configured (see [Setting Up SMTP](#setting-up-smtp))

#### Managing Your Preferences

1. Click **Notifications** in the sidebar (visible when User Notifications are enabled)
2. Toggle notifications for each event type:
   - **Print Job Starts** — Email when your print begins
   - **Print Job Finishes** — Email when your print completes successfully
   - **Print Errors** — Email when your print fails
   - **Print Job Stops** — Email when your print is cancelled
3. Click **Save**

!!! info "Requires Email Address"
    Users must have an email address on their account to receive notifications. The email address can be set by an administrator in User Management.

!!! tip "Permission Required"
    The `notifications:user_email` permission is required to access the Notifications page. Administrators and Operators have this by default. Viewers do not.

### Enabling/Disabling

Advanced Auth can be toggled on or off at any time without affecting basic authentication or existing user accounts. When disabled, user creation and password resets revert to the standard manual workflow.

---

## LDAP Authentication

Bambuddy supports LDAP/Active Directory authentication, allowing users to log in with their directory credentials. LDAP users coexist with local accounts — the local admin remains as a fallback when the LDAP server is unreachable.

### Setting Up LDAP

1. Go to **Settings** → **Authentication** → **LDAP** tab
2. Configure your LDAP server:
   - **Server URL** — `ldaps://ldap.example.com:636` for LDAPS or `ldap://ldap.example.com:389` for StartTLS
   - **Security** — StartTLS (upgrades plain connection to TLS) or LDAPS (TLS from the start)
   - **Bind DN** — Service account DN for searching users (e.g., `cn=admin,dc=example,dc=com`)
   - **Bind Password** — Service account password
   - **Search Base** — Where to search for users (e.g., `dc=example,dc=com`)
   - **User Search Filter** — LDAP filter to find users. `{username}` is replaced with the login name
     - Active Directory: `(sAMAccountName={username})`
     - OpenLDAP: `(uid={username})`
3. Click **Save**, then **Test Connection** to verify
4. Click **Enable** to activate LDAP authentication

!!! warning "TLS Required"
    Plaintext LDAP is not supported. All connections use either StartTLS or LDAPS to encrypt credentials in transit.

### User Provisioning

- **Auto-provision** — When enabled, a BamBuddy account is automatically created on first successful LDAP login. When disabled, an admin must pre-create the user in BamBuddy.
- **Email sync** — The user's email address from LDAP is synced on each login.
- **Auth source** — LDAP users are tagged with `auth_source=ldap` and shown with an "LDAP" badge in user management.

### Group Mapping

LDAP groups can be mapped to BamBuddy groups for automatic role assignment. The mapping is configured as a JSON object in the LDAP settings:

```json
{
  "cn=PrintFarm_Admins,ou=groups,dc=example,dc=com": "Administrators",
  "cn=PrintFarm_Operators,ou=groups,dc=example,dc=com": "Operators",
  "cn=PrintFarm_Viewers,ou=groups,dc=example,dc=com": "Viewers"
}
```

- Keys are LDAP group DNs (case-insensitive matching)
- Values are BamBuddy group names
- Both Active Directory groups (`memberOf` attribute) and POSIX groups (`memberUid` attribute) are supported
- Group membership is synced on every login

!!! tip "No Mapping? No Problem"
    If no group mapping is configured, LDAP users are created without any group. Admins can manually assign groups in BamBuddy afterward.

### Password Management

LDAP users cannot change their password through BamBuddy — passwords are managed by the LDAP server. The following features are automatically disabled for LDAP users:

- Change Password button (hidden in UI)
- Forgot Password (skipped for LDAP emails)
- Admin Password Reset (blocked with error message)

### Local Admin Fallback

Local accounts always work regardless of LDAP status. If the LDAP server is unreachable, LDAP authentication fails gracefully and falls back to local authentication. This ensures administrators can always access BamBuddy.

!!! info "Same Username"
    If a local user and an LDAP user have the same username, the local account takes priority. LDAP will not override an existing local account.

---

## Two-Factor Authentication (2FA)

Bambuddy supports per-user opt-in two-factor authentication via either **TOTP** (time-based one-time password, compatible with Google Authenticator, Authy, 2FAS, and any standard TOTP app) or **Email OTP** (a 6-digit code delivered to the user's registered email). Each user independently decides whether to enable 2FA and which method to use.

2FA is configured from **Settings → Authentication → Two-Factor Auth**. The tab header shows a green bullet when 2FA is active for your account.

### Enabling TOTP

1. Settings → Authentication → Two-Factor Auth → **Set up TOTP**
2. Scan the displayed QR code with your authenticator app, or paste the secret manually
3. Enter the 6-digit code your app currently shows to confirm
4. Bambuddy displays **10 single-use backup codes** — save them in a password manager or print them. They are shown **only once**
5. On next login, after entering username + password, Bambuddy prompts for the TOTP code

### Enabling Email OTP

1. Settings → Authentication → Two-Factor Auth → **Set up Email OTP**
2. Bambuddy emails a 6-digit setup code to your registered address
3. Enter the code to confirm; email 2FA is now active
4. Requires [Advanced Auth via Email](#advanced-auth-via-email) to be configured (for SMTP delivery)

### Backup Codes

Each enrolled TOTP user receives 10 backup codes. Any backup code can substitute for the TOTP code once — after use it is consumed. Regenerate a fresh set any time from Settings → Two-Factor Auth (invalidates all prior codes).

### Security Properties

- **Brute-force protection** — per-user **and** per-IP rate limits on every verify attempt
- **Replay protection** — a TOTP code cannot be accepted twice within the same 30-second window, on any of: setup, enable, verify, disable, backup-code regenerate
- **Single-use challenge** — the pre-auth token issued between password and 2FA verification is a DB-backed single-use token bound to the browser via an HttpOnly cookie. It cannot be replayed on a different browser or reused after a successful verify
- **Constant-time backup-code check** — all stored hashes are iterated on every attempt so the matching position in the list does not leak through timing
- **Silent-replacement guard** — replacing an active TOTP requires verifying the current code first

### Admin Reset

An administrator with `users:update` can disable any user's 2FA (TOTP, Email OTP, and backup codes) from **Settings → Users → Edit user**. Disabling also bumps the user's `password_changed_at`, invalidating any JWT issued before the reset.

---

## Single Sign-On (OIDC / SSO)

Bambuddy integrates with any standards-compliant OpenID Connect provider — **PocketID**, **Authentik**, **Keycloak**, **Google Workspace**, **Azure AD**, etc. The login page renders an SSO button for every enabled provider.

OIDC providers are configured from **Settings → Authentication → SSO / OIDC** (admin only). The tab header shows a green bullet when at least one provider is enabled.

### Adding a Provider

1. Create a client on your identity provider with redirect URI `https://<your-bambuddy>/api/v1/auth/oidc/callback`
2. Settings → Authentication → SSO / OIDC → **Add Provider**
3. Fill in:
    - **Name** — display name shown on the login page
    - **Issuer URL** — must start with `https://` (HTTP is rejected; private/loopback/link-local IPs are rejected to prevent SSRF)
    - **Client ID / Client Secret** — from the IdP. The client secret is encrypted at rest
    - **Scopes** — default `openid email profile` (the `openid` scope is required)
    - **Icon URL** (optional) — HTTPS only, rendered on the login page
4. Toggle **Enabled** — the SSO button appears on the login page immediately

### Account Linking & Auto-Provisioning

Two independent toggles per provider:

| Toggle | Default | Effect |
|--------|:-------:|--------|
| **Auto-create users** | Off | On first successful SSO login, create a new BamBuddy account for the verified email. Off → unknown emails are rejected |
| **Auto-link existing accounts** | Off | On first successful SSO login where the verified email matches an existing local user, link the two accounts. Off → admins must pre-link manually to prevent silent takeover by an attacker-controlled IdP |
| **Email Claim** | `email` | JWT claim used as the user's email identity. Set to `preferred_username` or `upn` for Azure Entra ID. Custom claims bypass the `email_verified` check entirely |
| **Require Email Verified** | On | Only accept the email claim if the provider marks it as verified (`email_verified: true`). Disable only when the provider never sends this flag (e.g. Azure Entra ID) or when using a custom Email Claim |

Auto-link is gated by an additional check: if the target user already has any OIDC link, a second IdP cannot auto-link to the same account.

> **Security constraint:** Auto-link requires both **Require Email Verified** to be on and **Email Claim** set to `email`. Using a custom claim (e.g. `preferred_username`) or disabling the verified-flag check automatically blocks auto-linking — this is enforced at the UI and database level. The reasoning: auto-linking based on an unverified or non-standard claim could allow an attacker-controlled IdP to silently take over a local account.

### Security Properties

- **PKCE (S256)** on every authorization request — safe for public clients without a secret
- **`email_verified` enforcement** — the IdP must explicitly mark the email as verified; unverified claims are ignored
- **Issuer / `aud` / `nonce` validation** on every callback — replayed or cross-client ID tokens are rejected
- **State single-use** — the OIDC `state` is a DB-backed single-use token that is atomically consumed on callback
- **Discovery-document SSRF hardening** — every URL pulled from the provider's `/.well-known/openid-configuration` (authorization_endpoint, token_endpoint, jwks_uri) is validated for scheme (http(s) only) **and** for private/loopback/link-local host IPs, so a compromised IdP cannot redirect the server's outbound calls at `169.254.169.254`, RFC1918, or loopback
- **`/oidc/authorize` rate-limited** per-IP to prevent discovery-document request amplification
- **OIDC users blocked from local password change / reset** — credentials live at the IdP, not in BamBuddy

## Email Claim

Defines which JWT claim from the provider's ID token is used as the user's email identity. Default is `email` (works with most providers). For **Azure Entra ID**, set this to `preferred_username` or `upn` — Entra ID always populates these with the user's UPN (e.g. `user@contoso.com`) but does not send an `email_verified` flag.

Only use claim names you fully trust. Custom claims bypass the `email_verified` check entirely (see below).

---

## Require Email Verified

When enabled (default), BamBuddy only accepts the email claim if the provider explicitly marks it as verified (`email_verified: true` in the ID token). This prevents unverified addresses from being used as identity anchors.

Disable this only when:

- The provider never sends `email_verified` (e.g. Azure Entra ID), **or**
- You are using a custom **Email Claim** (e.g. `preferred_username`) — in that case the verified-flag check is skipped automatically regardless of this setting.

> **Note:** If **Auto-link existing accounts** is enabled, `Require Email Verified` must be on and **Email Claim** must be `email`. This is enforced at the UI and database level to prevent account takeover.

### Trailing-Slash Tolerance

Both the admin-supplied issuer URL and the `issuer` claim returned by the discovery document are normalised (trailing slashes stripped) before the PyJWT `iss` check, so a provider that disagrees with itself by one byte (e.g. stores `https://idp/` but returns `https://idp` in the discovery doc) still works.

---

## Security Details

### Password Storage

Passwords are never stored in plain text. Bambuddy uses PBKDF2-SHA256 hashing with a secure salt for password storage.

### Token Authentication

- Bambuddy uses JWT (JSON Web Tokens) for authentication
- Tokens expire after 7 days
- Tokens are stored in the browser's localStorage
- Each API request includes the token for validation

### Best Practices

1. **Use Strong Passwords**: Choose passwords with at least 8 characters, mixing letters, numbers, and symbols
2. **Limit Admin Access**: Only add users to Administrators group when necessary
3. **Create Custom Groups**: Define groups matching your team's needs
4. **Use Least Privilege**: Give users only the permissions they need
5. **Regular Password Changes**: Consider changing passwords periodically
6. **Logout on Shared Devices**: Always log out when using shared computers

## User Activity Tracking

When authentication is enabled, Bambuddy tracks who performs key actions:

### What's Tracked

| Activity | Where It Shows |
|----------|----------------|
| **Archive uploads** | Archive cards show "Uploaded by {username}" |
| **Library file uploads** | File cards show "Uploaded by {username}" |
| **Queue additions** | Queue items show who added the print job |
| **Print starts** | Printer cards show "Started by {username}" during active prints |

### How It Works

- User tracking is automatic when logged in
- Information displays on cards and list items
- When auth is disabled, tracking fields are hidden
- Historical data is preserved even if the user is later deleted

### Privacy Note

User activity tracking helps teams understand who is using the system. If you prefer anonymous operation, simply disable authentication.

---

## Backup & Restore

User accounts and groups are included in backups:

- Enable **Include Users** and **Include Groups** options when creating a backup
- Passwords are NOT included in backups for security
- When restoring users, temporary passwords are generated
- Administrators must share these temporary passwords with users
- Users should change their passwords after restoration
- Group assignments are preserved during restore

## Troubleshooting

### Forgot Admin Password

If you forget your admin password and cannot log in:

1. Stop the Bambuddy service
2. Access the database directly
3. Delete the users table entries
4. Restart Bambuddy
5. Re-run the setup process

### Session Expired

If you see "Session expired" or get redirected to login:

- Your JWT token has expired (after 7 days)
- Simply log in again to continue

### Cannot Access a Feature

If a button or feature is disabled:

- Hover over it to see what permission is required
- Ask an administrator to add you to a group with that permission
- Or create a custom group with the needed permissions

### Cannot Access Settings

If you cannot access the Settings page:

- You need `settings:read` permission
- Ask an administrator to add you to a group with settings access
- Operators group has settings access by default
