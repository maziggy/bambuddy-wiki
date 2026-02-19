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
- **Settings**: read, update, backup, restore
- **Users/Groups**: read, create, update, delete
- And many more...

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

Templates can be edited in **Settings** → **Email** → **Templates**.

### Enabling/Disabling

Advanced Auth can be toggled on or off at any time without affecting basic authentication or existing user accounts. When disabled, user creation and password resets revert to the standard manual workflow.

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
