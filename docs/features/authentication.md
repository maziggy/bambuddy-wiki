# Authentication

Bambuddy includes an optional authentication system that allows you to secure your instance with user accounts and role-based access control. This feature is completely optional and can be enabled or disabled at any time.

## Overview

When enabled, authentication provides:

- **User Accounts**: Create multiple users with unique credentials
- **Role-Based Access**: Admin and User roles with different permission levels
- **Secure Authentication**: JWT tokens with password hashing using PBKDF2

## Roles

### Admin Role

Admins have full access to all Bambuddy features:

- All printer controls (start, stop, pause, resume)
- Print queue management
- Archive and file management
- Settings configuration
- User management
- Smart plug controls
- All other features

### User Role

Users have limited access focused on day-to-day operations:

- Printer monitoring and basic controls
- Print queue management
- Archive viewing
- File manager access
- Project management

Users cannot access:

- Settings page
- User management
- System configuration changes

## Enabling Authentication

### First-Time Setup

1. Navigate to any page in Bambuddy
2. You'll be redirected to the **Setup Page** if authentication is not configured
3. Choose to **Enable Authentication**
4. Create your admin account:
   - Enter a username
   - Enter a password (minimum 6 characters)
5. Click **Enable Authentication**

### From Settings (When Already Running)

1. Go to **Settings** → **Users** tab
2. Click **Activate Authentication**
3. You'll be redirected to the Setup Page
4. Complete the setup as described above

## Managing Users

### Creating Users

1. Log in as an admin
2. Go to **Settings** → **Users** tab
3. Click **Manage Users**
4. Click **Add User**
5. Fill in:
   - Username
   - Password (minimum 6 characters)
   - Role (Admin or User)
6. Click **Create**

### Editing Users

1. Go to **Settings** → **Users** → **Manage Users**
2. Click the edit icon next to a user
3. Modify username, password, or role
4. Click **Save**

### Deleting Users

1. Go to **Settings** → **Users** → **Manage Users**
2. Click the delete icon next to a user
3. Confirm deletion

Note: You cannot delete your own account.

## Disabling Authentication

If you need to disable authentication:

1. Log in as an admin
2. Go to **Settings** → **Users** tab
3. Click **Disable Authentication**
4. Confirm the action

**Warning**: Disabling authentication will delete all user accounts. You can re-enable authentication later, but you'll need to create new accounts.

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
2. **Limit Admin Accounts**: Only give admin access to users who need to change settings
3. **Regular Password Changes**: Consider changing passwords periodically
4. **Logout on Shared Devices**: Always log out when using shared computers

## Backup & Restore

User accounts can be included in backups:

- Enable **Include Users** option when creating a backup
- Passwords are NOT included in backups for security
- When restoring users, temporary passwords are generated
- Administrators must share these temporary passwords with users
- Users should change their passwords after restoration

## Troubleshooting

### Forgot Password

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

### Cannot Access Settings

If you cannot access the Settings page:

- You may be logged in as a User (not Admin)
- Ask an admin to upgrade your role, or
- Log in with an admin account
