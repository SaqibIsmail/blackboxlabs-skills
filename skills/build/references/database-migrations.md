---
name: database-migrations
description: PostgreSQL database migration best practices for schema changes, data migrations, rollbacks, and zero-downtime deployments with hand-written SQL (Flyway). Use when writing a schema or data migration, planning a rollback, or aiming for zero-downtime deployment.
metadata:
  origin: ECC
---

> **Provenance note (not part of the original file):** originally copied verbatim from
> `affaan-m/ECC`, `skills/database-migrations/SKILL.md` (MIT), fetched 2026-09-19, then
> **filtered for this project** (2026-09-19) against `blackboxlabs`'s own locked stack
> (`AGENTS.md`'s decision register: raw `pg`, Flyway migrations, no ORM). The Prisma,
> Drizzle, Kysely, Django, and golang-migrate sections were removed entirely — none of
> those tools are used here, and their command syntax and schema-file formats don't
> apply to Flyway's plain versioned `.sql` files. The original frontmatter also named
> MySQL and TypeORM, but the source file never actually had dedicated sections for
> either, so that mention was dropped along with the rest of the ORM list rather than
> filtered piecemeal. The PostgreSQL-specific safety guidance (safe column additions,
> concurrent index creation, avoiding full-table locks, separating schema from data
> migrations, the expand-contract pattern) was kept as-is: it applies directly to
> hand-written Flyway SQL migrations, which is exactly what this project writes. One
> inline Django-specific aside (`SeparateDatabaseAndState`) was removed from the
> "Removing a Column Safely" example for the same reason. A few notes were added
> (marked inline) tying the generic guidance to how Flyway is actually wired up here —
> see `docs/database.md` (`V<N>__description.sql` under `migrations/postgres/`,
> forward-only, a `-- ROLLBACK:` comment documenting the undo rather than an executable
> down migration, no `executeInTransaction` override in `flyway/postgres.toml`). This
> file is project-specific now, not the generic upstream copy — regenerate it (don't
> hand-edit further piecemeal) if the upstream source or this project's locked stack
> changes.

# Database Migration Patterns

Safe, reversible database schema changes for production systems.

## When to Activate

- Creating or altering database tables
- Adding/removing columns or indexes
- Running data migrations (backfill, transform)
- Planning zero-downtime schema changes
- Setting up migration tooling for a new project

## Core Principles

1. **Every change is a migration** — never alter production databases manually
2. **Migrations are forward-only in production** — rollbacks use new forward migrations. This matches how Flyway is used here: `docs/database.md` documents undo SQL in a `-- ROLLBACK:` comment at the top of the file, not an executable down migration that runs automatically.
3. **Schema and data migrations are separate** — never mix DDL and DML in one migration
4. **Test migrations against production-sized data** — a migration that works on 100 rows may lock on 10M
5. **Migrations are immutable once deployed** — never edit a migration that has run in production. Already a hard rule in root `AGENTS.md` ("New migration / schema change" → `docs/database.md`; "immutability enforced by CI").

## Migration Safety Checklist

Before applying any migration:

- [ ] Migration has both UP and DOWN (or is explicitly marked irreversible) — for this project, the UP is the `V<N>__*.sql` file itself and the "DOWN" is its `-- ROLLBACK:` comment (see `docs/database.md`)
- [ ] No full table locks on large tables (use concurrent operations)
- [ ] New columns have defaults or are nullable (never add NOT NULL without default)
- [ ] Indexes created concurrently (not inline with CREATE TABLE for existing tables)
- [ ] Data backfill is a separate migration from schema change
- [ ] Tested against a copy of production data
- [ ] Rollback plan documented

## PostgreSQL Patterns

These apply directly to this project's hand-written Flyway SQL migrations
(`migrations/postgres/V<N>__*.sql`) — there is no ORM generating SQL here, so getting
the raw SQL right in the migration file *is* the whole job.

### Adding a Column Safely

```sql
-- GOOD: Nullable column, no lock
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- GOOD: Column with default (Postgres 11+ is instant, no rewrite)
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- BAD: NOT NULL without default on existing table (requires full rewrite)
ALTER TABLE users ADD COLUMN role TEXT NOT NULL;
-- This locks the table and rewrites every row
```

### Adding an Index Without Downtime

```sql
-- BAD: Blocks writes on large tables
CREATE INDEX idx_users_email ON users (email);

-- GOOD: Non-blocking, allows concurrent writes
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);

-- Note: CONCURRENTLY cannot run inside a transaction block
```

**Flyway-specific gotcha:** Flyway wraps each SQL migration for Postgres in a single
transaction by default, and that will fail on `CREATE INDEX CONCURRENTLY` with a
"cannot run inside a transaction block" error. `flyway/postgres.toml` in this project
doesn't currently override that, so a migration needing `CONCURRENTLY` will need
Flyway's per-script `-- flyway:executeInTransaction=false` pragma (or an equivalent
config change) — check the installed Flyway edition/version supports it before relying
on it, and confirm against a real run rather than assuming.

### Renaming a Column (Zero-Downtime)

Never rename directly in production. Use the expand-contract pattern:

```sql
-- Step 1: Add new column (migration 001)
ALTER TABLE users ADD COLUMN display_name TEXT;

-- Step 2: Backfill data (migration 002, data migration)
UPDATE users SET display_name = username WHERE display_name IS NULL;

-- Step 3: Update application code to read/write both columns
-- Deploy application changes

-- Step 4: Stop writing to old column, drop it (migration 003)
ALTER TABLE users DROP COLUMN username;
```

Each numbered step here is its own `V<N>__*.sql` file in this project's convention —
never combine them into one migration.

### Removing a Column Safely

```sql
-- Step 1: Remove all application references to the column
-- Step 2: Deploy application without the column reference
-- Step 3: Drop column in next migration
ALTER TABLE orders DROP COLUMN legacy_status;
```

### Large Data Migrations

```sql
-- BAD: Updates all rows in one transaction (locks table)
UPDATE users SET normalized_email = LOWER(email);

-- GOOD: Batch update with progress
DO $$
DECLARE
  batch_size INT := 10000;
  rows_updated INT;
BEGIN
  LOOP
    UPDATE users
    SET normalized_email = LOWER(email)
    WHERE id IN (
      SELECT id FROM users
      WHERE normalized_email IS NULL
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % rows', rows_updated;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

Per Core Principle 3 above, this kind of batched backfill belongs in its own data
migration file, separate from whatever DDL introduced the column it's populating.

## Zero-Downtime Migration Strategy

For critical production changes, follow the expand-contract pattern:

```
Phase 1: EXPAND
  - Add new column/table (nullable or with default)
  - Deploy: app writes to BOTH old and new
  - Backfill existing data

Phase 2: MIGRATE
  - Deploy: app reads from NEW, writes to BOTH
  - Verify data consistency

Phase 3: CONTRACT
  - Deploy: app only uses NEW
  - Drop old column/table in separate migration
```

### Timeline Example

```
Day 1: Migration adds new_status column (nullable)
Day 1: Deploy app v2 — writes to both status and new_status
Day 2: Run backfill migration for existing rows
Day 3: Deploy app v3 — reads from new_status only
Day 7: Migration drops old status column
```

## Anti-Patterns

| Anti-Pattern | Why It Fails | Better Approach |
|-------------|-------------|-----------------|
| Manual SQL in production | No audit trail, unrepeatable | Always use migration files |
| Editing deployed migrations | Causes drift between environments | Create new migration instead |
| NOT NULL without default | Locks table, rewrites all rows | Add nullable, backfill, then add constraint |
| Inline index on large table | Blocks writes during build | CREATE INDEX CONCURRENTLY |
| Schema + data in one migration | Hard to rollback, long transactions | Separate migrations |
| Dropping column before removing code | Application errors on missing column | Remove code first, drop column next deploy |
