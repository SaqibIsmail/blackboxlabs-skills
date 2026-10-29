---
name: database-reviewer
description: PostgreSQL query performance, schema design, indexing, security, and connection/concurrency review guidance. Use when writing SQL, creating migrations, designing schemas, or troubleshooting database performance.
metadata:
  origin: ECC
---

> **Provenance note (not part of the original file):** originally copied verbatim from
> `affaan-m/ECC`, `agents/database-reviewer.md` (MIT), fetched 2026-09-19, then
> **filtered for this project** (2026-09-19) against `blackboxlabs`'s own locked stack
> (`AGENTS.md`'s decision register) — this project has no Supabase at all: it uses
> Auth.js (NextAuth) sessions and the project's own RBAC permission map for access
> control at the application layer, not Row Level Security, which is a Supabase/Postgres
> feature this project doesn't use. The RLS- and `(SELECT auth.uid())`-specific bullets
> and checklist items were removed entirely rather than caveated, since they don't apply
> at all here, not just partially. Everything else — query performance, indexing, data
> types, connection pooling, concurrency, and anti-patterns like `SELECT *`, N+1 queries,
> and OFFSET pagination — is real, stack-agnostic Postgres review guidance kept as-is.
> This file was also converted from an installable Claude Code agent into a plain
> reference doc: the `tools:`/`model:` frontmatter and the "Prompt Defense Baseline"
> boilerplate (agent-only prompt-injection defenses, not reference content) were
> dropped. This file is project-specific now, not the generic upstream copy —
> regenerate it (don't hand-edit further piecemeal) if the upstream source or this
> project's locked stack changes.

# Database Reviewer

PostgreSQL guidance for query optimization, schema design, security, and performance —
covering how to keep database code following best practices, prevent performance
issues, and maintain data integrity. Incorporates patterns from Supabase's
postgres-best-practices (credit: Supabase team).

## Core Responsibilities

1. **Query Performance** — Optimize queries, add proper indexes, prevent table scans
2. **Schema Design** — Design efficient schemas with proper data types and constraints
3. **Security** — Least privilege access, safe query patterns
4. **Connection Management** — Configure pooling, timeouts, limits
5. **Concurrency** — Prevent deadlocks, optimize locking strategies
6. **Monitoring** — Set up query analysis and performance tracking

## Diagnostic Commands

```bash
psql $DATABASE_URL
psql -c "SELECT query, mean_exec_time, calls FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;"
psql -c "SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) FROM pg_stat_user_tables ORDER BY pg_total_relation_size(relid) DESC;"
psql -c "SELECT indexrelname, idx_scan, idx_tup_read FROM pg_stat_user_indexes ORDER BY idx_scan DESC;"
```

## Review Workflow

### 1. Query Performance (CRITICAL)
- Are WHERE/JOIN columns indexed?
- Run `EXPLAIN ANALYZE` on complex queries — check for Seq Scans on large tables
- Watch for N+1 query patterns
- Verify composite index column order (equality first, then range)

### 2. Schema Design (HIGH)
- Use proper types: `bigint` for IDs, `text` for strings, `timestamptz` for timestamps, `numeric` for money, `boolean` for flags
- Define constraints: PK, FK with `ON DELETE`, `NOT NULL`, `CHECK`
- Use `lowercase_snake_case` identifiers (no quoted mixed-case)

### 3. Security (CRITICAL)
- Least privilege access — no `GRANT ALL` to application users
- Public schema permissions revoked

## Key Principles

- **Index foreign keys** — Always, no exceptions
- **Use partial indexes** — `WHERE deleted_at IS NULL` for soft deletes
- **Covering indexes** — `INCLUDE (col)` to avoid table lookups
- **SKIP LOCKED for queues** — 10x throughput for worker patterns
- **Cursor pagination** — `WHERE id > $last` instead of `OFFSET`
- **Batch inserts** — Multi-row `INSERT` or `COPY`, never individual inserts in loops
- **Short transactions** — Never hold locks during external API calls
- **Consistent lock ordering** — `ORDER BY id FOR UPDATE` to prevent deadlocks

## Anti-Patterns to Flag

- `SELECT *` in production code
- `int` for IDs (use `bigint`), `varchar(255)` without reason (use `text`)
- `timestamp` without timezone (use `timestamptz`)
- Random UUIDs as PKs (use UUIDv7 or IDENTITY)
- OFFSET pagination on large tables
- Unparameterized queries (SQL injection risk)
- `GRANT ALL` to application users

## Review Checklist

- [ ] All WHERE/JOIN columns indexed
- [ ] Composite indexes in correct column order
- [ ] Proper data types (bigint, text, timestamptz, numeric)
- [ ] Foreign keys have indexes
- [ ] No N+1 query patterns
- [ ] EXPLAIN ANALYZE run on complex queries
- [ ] Transactions kept short

## Reference

For detailed index patterns, schema design examples, connection management, concurrency strategies, JSONB patterns, and full-text search, see skills: `postgres-patterns` and `database-migrations`.

---

**Remember**: Database issues are often the root cause of application performance problems. Optimize queries and schema design early. Use EXPLAIN ANALYZE to verify assumptions. Always index foreign keys.

*Patterns adapted from Supabase Agent Skills (credit: Supabase team) under MIT license.*
