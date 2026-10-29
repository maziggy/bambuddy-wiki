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

!!! info "Part of a bigger picture"
    On a large farm the database is the *second* thing to fix — camera streaming
    usually dominates by an order of magnitude. See
    [Running a Large Farm](../reference/farm-sizing.md).

---

## :rocket: Setup

### 1. Install PostgreSQL

Skip this step if you already have a PostgreSQL server running. It must be **PostgreSQL 13 or newer**; Bambuddy does not start on 12 or older.

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
| Pool size | 20 connections | 20 connections |
| Max overflow | 200 | 80 |
| Pre-ping / recycle | N/A | Enabled / 1800s |
| WAL mode | Enabled | N/A |
| Busy timeout | 15 seconds | N/A (uses pool) |

The defaults suit most installs — no configuration needed. `pool_size + max_overflow`
is the ceiling on concurrent database connections (per app worker process).

### Tuning for large printer farms

On large farms (many dozens of printers), the default 80-connection ceiling can be
raised. Each of these is optional and overrides the default when set:

| Variable | Default (PostgreSQL) | Purpose |
|----------|----------------------|---------|
| `DB_POOL_SIZE` | `20` | Base connections kept open |
| `DB_MAX_OVERFLOW` | `60` (`80` before 1.2.5.7) | Extra connections opened on demand |
| `DB_POOL_TIMEOUT` | `30` | Seconds a request waits for a free connection before erroring |
| `DB_POOL_RECYCLE` | `1800` | Seconds before a pooled connection is recycled |
| `DB_POOL_USE_LIFO` | `true` | Reuse the most-recently-returned connection so a bursty farm keeps a small hot set busy and lets excess overflow connections age out via `DB_POOL_RECYCLE` instead of churning the whole pool. Set `false` for strict round-robin (FIFO) |
| `DB_POOL_PRE_PING` | `true` | Check each connection with a cheap ping before handing it out, so connections dropped by the server or a proxy are replaced instead of failing the request. Set `false` to skip it, for a database on the same host or LAN only (see the note below) |

!!! note "Rollback ratio on PostgreSQL monitoring"
    Bambuddy ends most of its database sessions with a `ROLLBACK` instead of a `COMMIT`:
    read-only sessions are closed without committing, and with `DB_POOL_PRE_PING` on every
    pooled session is also preceded by an empty `BEGIN; ROLLBACK;` from the connection
    ping. `pg_stat_database` therefore shows a high `xact_rollback` share (about 80 % on a
    small install), and monitoring that alerts on the rollback ratio (for example Netdata's
    `postgres_db_transactions_rollback_ratio`, which warns above 2 %) stays raised. No
    statement has failed. `DB_POOL_PRE_PING=false` roughly halves the count; the rest is
    normal read-only session cleanup, so raise or scope the alert threshold for the
    Bambuddy database. **The trade-off:** with the check off, a connection the server
    dropped (PostgreSQL restart, firewall or NAT idle timeout) is no longer replaced quietly;
    it fails the first request that picks it up. `DB_POOL_RECYCLE` only limits how old a
    connection gets. Turn it off only for a database on the same host or LAN, and keep it on
    behind a firewall, NAT or PgBouncer.

```bash
# Example for a ~100-printer farm
DB_POOL_SIZE=20
DB_MAX_OVERFLOW=180   # 200-connection ceiling
```

!!! warning "Size PostgreSQL `max_connections` to match"
    PostgreSQL must allow at least `DB_POOL_SIZE + DB_MAX_OVERFLOW` connections
    **per app worker process**, plus headroom for admin/backup tools. Raise
    `max_connections` in `postgresql.conf` accordingly (and note each connection
    costs memory server-side).

    The default ceiling is `20 + 60 = 80`, which fits a stock PostgreSQL:
    it ships `max_connections = 100` and reserves 3 of those for superusers,
    leaving 97. Up to 1.2.5.6 the default was `20 + 80 = 100`, already over
    that budget; on those versions set `DB_MAX_OVERFLOW=60`. When you raise the
    pool, raise the server with it:

    ```conf
    # postgresql.conf
    max_connections = 200
    ```

    Bambuddy checks this at startup and logs a warning naming both numbers when
    the pool could ask for more than the server allows.

!!! info "Which error means what"
    The two failure modes look similar and have opposite fixes:

    | Error | Meaning | Fix |
    |-------|---------|-----|
    | `asyncpg ... TooManyConnectionsError` | The pool's ceiling is **above** what the server allows, so it never queues — it asks the server, which refuses | Lower `DB_MAX_OVERFLOW` or raise `max_connections` |
    | `QueuePool limit ... connection timed out` | The pool itself is the bottleneck: too much concurrency, or connections held too long | Raise `DB_POOL_SIZE` / `DB_MAX_OVERFLOW`, or investigate what is holding connections |

    `TooManyConnectionsError` surfaces at whatever needed a connection next,
    which can be far from the real cause — a queue dispatch, a notification, a
    web request. `GET /api/v1/system/db-pool` reports both sides under
    `server_limits`, and so does the support bundle.

!!! tip "Watch the live pool"
    `GET /api/v1/system/db-pool` reports the resolved configuration plus live
    `checked_out` / `checked_in` / `overflow` gauges (requires the `system:read`
    permission). If `checked_out` sits pinned at the ceiling, connections are
    being held across slow work — raising the pool only buys time.

---

## :material-frequently-asked-questions: FAQ

**Can I switch back to SQLite?**
:   Yes. Remove the `DATABASE_URL` environment variable and restart. Bambuddy falls back to the built-in SQLite database. Your PostgreSQL data remains untouched — create a backup first if you want to keep it.

**Do I need `pg_dump` installed on the Bambuddy host?**
:   No. Bambuddy's backup system exports data using pure Python (SQLAlchemy + sqlite3), so no external PostgreSQL tools are required on the Bambuddy server.

**What PostgreSQL version is supported?**
:   PostgreSQL **13 or newer** is required; 14 or newer is recommended, and 16 is what the examples here and the Bambuddy Appliance use. Bambuddy is tested on 13 to 16. **PostgreSQL 12 and older are not supported:** Bambuddy does not start on them, because some of its startup migrations use column types those versions reject. PostgreSQL 12 itself has been out of support since November 2024. The `asyncpg` driver would connect to older servers, but that does not make them usable.

**Can I share a PostgreSQL server with other applications?**
:   Yes. Create a separate database and user for Bambuddy. It won't interfere with other databases on the same server.

**What happens if the PostgreSQL server goes down?**
:   Bambuddy will show connection errors until the server is back. No data is lost — PostgreSQL is ACID-compliant. Bambuddy automatically reconnects when the server recovers.
