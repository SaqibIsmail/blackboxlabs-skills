---
name: postgres-patterns
description: PostgreSQL database patterns for query optimization, schema design, indexing, and data types. Use when designing PostgreSQL schemas, indexes, or when a query is too slow.
metadata:
  origin: ECC
---

> **Provenance note (not part of the original file):** originally copied verbatim from
> `affaan-m/ECC`, `skills/postgres-patterns/SKILL.md` (MIT), fetched 2026-09-19, then
> **filtered for this project** (2026-09-19) against `blackboxlabs`'s own locked stack
> (`AGENTS.md`'s decision register). The original file describes itself as "Based on
> Supabase best practices" and includes an RLS policy example built on Supabase's own
> `(SELECT auth.uid())` helper — that function only exists inside Supabase's auth
> schema, and this project uses Auth.js (NextAuth) with raw `pg` + `redis` (no Supabase,
> no ORM) and enforces authorization at the application layer with CASL (see
> `docs/auth.md`), not via database-level Row Level Security. The Supabase-specific RLS
> code block and the matching "Implementing Row Level Security" activation trigger were
> removed for that reason, with a short caveat left in their place in case RLS is ever
> adopted here. Everything else — index type selection, data type choices, generic query
> patterns (composite/covering/partial indexes, UPSERT, cursor pagination, queue
> processing via `FOR UPDATE SKIP LOCKED`), the anti-pattern diagnostic queries, and the
> configuration template — is stack-agnostic Postgres guidance and was kept as-is. This
> file is project-specific now, not the generic upstream copy — regenerate it (don't
> hand-edit further piecemeal) if the upstream source or this project's locked stack
> changes.

# PostgreSQL Patterns

Quick reference for PostgreSQL best practices. For detailed guidance, use the `database-reviewer` agent.

## When to Activate

- Writing SQL queries or migrations
- Designing database schemas
- Troubleshooting slow queries
- Setting up connection pooling

## Quick Reference

### Index Cheat Sheet

| Query Pattern | Index Type | Example |
|--------------|------------|---------|
| `WHERE col = value` | B-tree (default) | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | Composite | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| `WHERE tsv @@ query` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| Time-series ranges | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### Data Type Quick Reference

| Use Case | Correct Type | Avoid |
|----------|-------------|-------|
| IDs | `bigint` | `int`, random UUID |
| Strings | `text` | `varchar(255)` |
| Timestamps | `timestamptz` | `timestamp` |
| Money | `numeric(10,2)` | `float` |
| Flags | `boolean` | `varchar`, `int` |

### Common Patterns

**Composite Index Order:**
```sql
-- Equality columns first, then range columns
CREATE INDEX idx ON orders (status, created_at);
-- Works for: WHERE status = 'pending' AND created_at > '2024-01-01'
```

**Covering Index:**
```sql
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
-- Avoids table lookup for SELECT email, name, created_at
```

**Partial Index:**
```sql
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;
-- Smaller index, only includes active users
```

**Row Level Security — not used here:**
The upstream pattern showed an RLS policy built on Supabase's `(SELECT auth.uid())`
helper, which only exists inside Supabase's own auth schema. This project doesn't use
Supabase auth (it uses Auth.js/NextAuth) and authorization is enforced at the
application layer with CASL (see `docs/auth.md`), not via Postgres RLS — so that example
was removed rather than adapted. If DB-level RLS is ever introduced here, the
current-user id would need to come from a session variable the app sets per connection
(e.g. `SET LOCAL app.user_id = ...`), with policies reading it via `current_setting()`,
not `auth.uid()`.

**UPSERT:**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (123, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value;
```

**Cursor Pagination:**
```sql
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
-- O(1) vs OFFSET which is O(n)
```

**Queue Processing:**
```sql
UPDATE jobs SET status = 'processing'
WHERE id = (
  SELECT id FROM jobs WHERE status = 'pending'
  ORDER BY created_at LIMIT 1
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

### Anti-Pattern Detection

```sql
-- Find unindexed foreign keys
SELECT conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
  );

-- Find slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;

-- Check table bloat
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### Configuration Template

```sql
-- Connection limits (adjust for RAM)
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- Timeouts
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';
ALTER SYSTEM SET statement_timeout = '30s';

-- Monitoring
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Security defaults
REVOKE ALL ON SCHEMA public FROM public;

SELECT pg_reload_conf();
```

## Related

- Agent: `database-reviewer` - Full database review workflow
- Skill: `clickhouse-io` - ClickHouse analytics patterns
- Skill: `backend-patterns` - API and backend patterns

---

*Based on Supabase Agent Skills (credit: Supabase team) (MIT License)*
