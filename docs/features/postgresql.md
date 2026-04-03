---
title: PostgreSQL Support
description: Optional external PostgreSQL database for larger setups
---

# PostgreSQL Support

Bambuddy supports an optional external PostgreSQL database as an alternative to the built-in SQLite. SQLite remains the default and requires zero configuration.

---

## :material-help-circle: When to Use PostgreSQL

SQLite works great for most users. Consider PostgreSQL if you:

- Run a **large print farm** (10+ printers) with high write concurrency
- Want a **dedicated database server** shared across services
- Need **point-in-time recovery** or streaming replication
- Already have PostgreSQL infrastructure (e.g., for other services)

!!! tip "SQLite is Fine for Most Users"
    SQLite with WAL mode handles concurrent reads + single writer well. Bambuddy tunes busy_timeout and synchronous mode automatically. If you're not hitting performance issues, there's no reason to switch.

---

## :rocket: Setup

### 1. Install PostgreSQL

Skip this step if you already have a PostgreSQL server running.

=== ":material-docker: Docker (Recommended)"

    Create a `docker-compose.yml` for PostgreSQL (or add to an existing one):

    ```yaml
    services:
      postgres:
        image: postgres:16-alpine
        container_name: bambuddy-db
        restart: unless-stopped
        environment:
          POSTGRES_USER: bambuddy
          POSTGRES_PASSWORD: your-secure-password
          POSTGRES_DB: bambuddy
        volumes:
          - pgdata:/var/lib/postgresql/data
        ports:
          - "5432:5432"

    volumes:
      pgdata:
    ```

    Start it:

    ```bash
    docker compose up -d
    ```

    That's it — the database and user are created automatically from the environment variables.

=== ":material-ubuntu: Debian / Ubuntu"

    ```bash
    # Install PostgreSQL
    sudo apt update && sudo apt install -y postgresql postgresql-client

    # Create user and database
    sudo -u postgres psql -c "CREATE USER bambuddy WITH PASSWORD 'your-secure-password';"
    sudo -u postgres psql -c "CREATE DATABASE bambuddy OWNER bambuddy;"
    ```

    PostgreSQL starts automatically after installation. Verify it's running:

    ```bash
    sudo systemctl status postgresql
    ```

    !!! note "Remote Access"
        By default, PostgreSQL only accepts local connections. To allow connections from another host (e.g., Bambuddy on a different machine), edit `/etc/postgresql/*/main/pg_hba.conf` and add:

        ```
        host    bambuddy    bambuddy    192.168.0.0/16    scram-sha-256
        ```

        And in `/etc/postgresql/*/main/postgresql.conf`, set:

        ```
        listen_addresses = '*'
        ```

        Then restart: `sudo systemctl restart postgresql`

=== ":material-fedora: Fedora / RHEL"

    ```bash
    # Install PostgreSQL
    sudo dnf install -y postgresql-server postgresql
    sudo postgresql-setup --initdb
    sudo systemctl enable --now postgresql

    # Create user and database
    sudo -u postgres psql -c "CREATE USER bambuddy WITH PASSWORD 'your-secure-password';"
    sudo -u postgres psql -c "CREATE DATABASE bambuddy OWNER bambuddy;"
    ```

=== ":material-server: Existing Server"

    If you already have a PostgreSQL server, create the user and database:

    ```bash
    psql -U postgres -c "CREATE USER bambuddy WITH PASSWORD 'your-secure-password';"
    psql -U postgres -c "CREATE DATABASE bambuddy OWNER bambuddy;"
    ```

    Or from inside a Docker container:

    ```bash
    docker exec -it your-postgres-container psql -U postgres -d postgres \
      -c "CREATE USER bambuddy WITH PASSWORD 'your-secure-password';"
    docker exec -it your-postgres-container psql -U postgres -d postgres \
      -c "CREATE DATABASE bambuddy OWNER bambuddy;"
    ```

### 2. Configure Bambuddy

Set the `DATABASE_URL` environment variable:

```
DATABASE_URL=postgresql+asyncpg://bambuddy:your-secure-password@db-host:5432/bambuddy
```

=== ":material-docker: Docker"

    Add to `docker-compose.override.yml` (create if it doesn't exist):

    ```yaml
    services:
      bambuddy:
        environment:
          - DATABASE_URL=postgresql+asyncpg://bambuddy:your-password@db-host:5432/bambuddy
    ```

    Then restart:

    ```bash
    docker compose up -d
    ```

=== ":material-linux: Systemd"

    Add to your `.env` file:

    ```bash
    DATABASE_URL=postgresql+asyncpg://bambuddy:your-password@db-host:5432/bambuddy
    ```

    Then restart:

    ```bash
    sudo systemctl restart bambuddy
    ```

### 3. Start Bambuddy

Bambuddy automatically creates all tables on first startup. No manual schema setup needed.

---

## :material-backup-restore: Migrating from SQLite

To migrate your existing data from SQLite to PostgreSQL:

1. **Create a backup** from your current SQLite install (Settings > Backup & Restore > Create Backup)
2. **Configure** the `DATABASE_URL` as described above
3. **Restart** Bambuddy (it creates empty tables on the new database)
4. **Restore** the backup ZIP through Settings > Backup & Restore > Restore Backup

Bambuddy automatically handles the cross-database import:

- Converts SQLite integer booleans (0/1) to PostgreSQL native booleans
- Parses SQLite datetime strings into proper datetime objects
- Fills default values for columns added after the backup was created
- Handles foreign key constraints and duplicate detection
- Resets PostgreSQL sequences to match imported data

---

## :material-swap-horizontal: Portable Backups

Backups are always in portable SQLite format, regardless of which database backend you use. This means:

- A backup from a **PostgreSQL** install can be restored on a **SQLite** install
- A backup from a **SQLite** install can be restored on a **PostgreSQL** install
- Backups work across different Bambuddy versions (forward-compatible)

---

## :material-database-check: Health Diagnostics

The support bundle (Settings > System > Support Bundle) automatically detects the database backend and reports:

| Metric | SQLite | PostgreSQL |
|--------|--------|------------|
| Backend type | `sqlite` | `postgresql` |
| Version | Journal mode | PostgreSQL version |
| Database size | File size + WAL size | `pg_database_size()` |
| Integrity check | `PRAGMA quick_check` | Connection test |

---

## :material-text-search: Full-Text Search

Archive search works on both backends with the best available engine:

| Backend | Technology | Features |
|---------|-----------|----------|
| SQLite | FTS5 virtual table | Prefix matching, ranking by relevance |
| PostgreSQL | tsvector + GIN index | Prefix matching, language-aware stemming |

The search experience is identical from the user's perspective.

---

## :material-cog: Connection Settings

| Setting | SQLite | PostgreSQL |
|---------|--------|------------|
| Pool size | 20 connections | 10 connections |
| Max overflow | 200 | 20 |
| WAL mode | Enabled | N/A |
| Busy timeout | 15 seconds | N/A (uses pool) |

These are tuned automatically — no configuration needed.

---

## :material-frequently-asked-questions: FAQ

**Can I switch back to SQLite?**
:   Yes. Remove the `DATABASE_URL` environment variable and restart. Bambuddy falls back to the built-in SQLite database. Your PostgreSQL data remains untouched — create a backup first if you want to keep it.

**Do I need `pg_dump` installed on the Bambuddy host?**
:   No. Bambuddy's backup system exports data using pure Python (SQLAlchemy + sqlite3), so no external PostgreSQL tools are required on the Bambuddy server.

**What PostgreSQL version is supported?**
:   PostgreSQL 14 or newer is recommended. The `asyncpg` driver supports PostgreSQL 9.5+, but Bambuddy uses features like `GIN` indexes that work best on modern versions.

**Can I share a PostgreSQL server with other applications?**
:   Yes. Create a separate database and user for Bambuddy. It won't interfere with other databases on the same server.

**What happens if the PostgreSQL server goes down?**
:   Bambuddy will show connection errors until the server is back. No data is lost — PostgreSQL is ACID-compliant. Bambuddy automatically reconnects when the server recovers.
