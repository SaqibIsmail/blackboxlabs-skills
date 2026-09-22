# /assign-tasks — Pipeline Design (draft, not yet a working skill)

**Status (2026-09-11): superseded by `senior-engineer`.** The pipeline grew an epic layer above
spec-kit's features, and the ticket-splitting job needed real granularity control (ticket size,
not just FE/BE labeling) — see `senior-engineer/DESIGN_NOTES.md`'s "Relationship to
`assign-tasks`" section. Kept here, not deleted, because its BMAD decomposition-rigor citation
and ticketing-backend research (spec-kit delegation, Jira title conventions) are still the basis
`senior-engineer`'s own notes build on. Do not implement this file as a `SKILL.md` — implement
`senior-engineer` instead.

~~Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.~~

## What this is

The scrum-master skill. Takes spec-kit's `/speckit.tasks` output for a feature, classifies each
task as frontend/backend/shared, creates tickets (or leaves an annotated `tasks.md` as the
ticket, when no ticketing system is configured), and checks that frontend and backend work stays
pointed at the same shared plan.

Cherry-picks decomposition rigor from BMAD-METHOD's `bmad-create-epics-and-stories` and
`bmad-sprint-planning` skills — rewritten in plain prose, not their runtime — for how to think
about splitting a plan into assignable units of work; the actual mechanics (task IDs, ticket
creation) are this pipeline's own, built on spec-kit's real artifact format.

## Command

```
/assign-tasks <feature>
```

## FE/BE task-split convention

spec-kit's real `tasks.md` line grammar is `- [ ] T012 [P] [US1] Description`. This skill appends
a bracket tag using that same grammar — never touching the `T0xx` ID or reordering/rewriting the
description:

```
- [ ] T012 [P] [US1] [FE] Create LoginForm component in features/auth/components/LoginForm.tsx
- [ ] T014 [US1] [BE] Implement AuthService in features/auth/server/auth.service.ts
- [ ] T004 [SHARED] Setup database schema and migrations framework
```

Classification signal, in order:

1. File path in the task description, against `PROJECT.md.stack` conventions — e.g. in
   `blackboxlabs`, concretely `features/*/components/` → FE, `features/*/server/` or
   `app/api/**/route.ts` → BE.
2. Keyword fallback (UI/component/page/style vs. service/route/schema/migration) when the path
   is ambiguous or the task predates any file existing yet.

`[SHARED]` tasks (spec-kit's own foundational/setup phase, which its template already marks as
blocking every user story) are never auto-assigned to either build agent — flagged for the user
to route manually.

## Ticket creation

- `PROJECT.md.ticketing.system: none` — first-class, not a fallback. The annotated `tasks.md`
  itself is the "ticket": FE/BE agents read their own tagged subset directly, no external system
  involved.
- `github-issues` — spec-kit already ships `/speckit.taskstoissues`, which creates GitHub issues
  from `tasks.md` with working dedup (matches `\bT\d{3,}\b` in existing issue titles before
  creating). Delegating to it (rather than reimplementing) reuses that dedup logic for free —
  see open question 1.
- `jira` (or another system) — no existing mechanism to delegate to; this skill makes the actual
  ticket-creation calls itself, using `PROJECT.md.ticketing.project_key`/`auth_env`. Title format:
  `FE-<slug>` / `BE-<slug>` (a convention this pipeline invents, since neither spec-kit nor Jira
  has a native frontend/backend layer concept). Each ticket description embeds a link to
  `specs/<feature>/{spec.md,plan.md}` and the originating `T0xx` ID, for traceability.

## Cross-ticket compatibility check

Before cutting the BE batch, compare a hash/mtime of `plan.md` against the value recorded when
the FE batch was cut (a small sidecar, e.g. `.assign-tasks/<feature>.state.json`). If `plan.md`
changed in between, flag it to the user rather than silently creating BE tickets against a stale
plan — this is the concrete mechanism behind "ensure frontend and backend will work together"
from the original ask.

## Call to `log-decision`

Once tickets (or the annotated `tasks.md`) exist, enrich the entry `plan-feature` already wrote
for this feature with `ticket-refs: [FE-123, BE-124]`.

## Open questions

1. Delegate to spec-kit's own `/speckit.taskstoissues` when `ticketing.system: github-issues`
   (reusing its real dedup), and only reimplement ticket creation for Jira/other systems — or
   always reimplement, for consistent behavior across every ticketing backend? Delegating is
   less code but means this skill's behavior genuinely differs by backend.
2. Per `define-project`'s own open question 2 — does `ticketing.system: none` still need a
   recorded task-ID-prefix convention, or is `[FE]`/`[BE]` directly in `tasks.md` sufficient on
   its own with nothing else to configure?

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 — materially changes how much of this skill needs writing per
   ticketing backend.
2. Write the concrete keyword-fallback list for task classification (open to expansion per
   stack, but needs a real starting list to be implementable).
