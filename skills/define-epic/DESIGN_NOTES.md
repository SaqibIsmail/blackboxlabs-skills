# /define-epic — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-11); not yet implemented as
`SKILL.md`. Open questions below block that.

## What this is

The epic-level interview: turns a whole-feature ask ("we want X") into a scoped, Jira-epic-shaped
brief, before any spec-kit mechanics run. Sits **above** `plan-feature` — an epic may span
multiple spec-kit features underneath it; `senior-engineer` decides that split later, not this
skill.

Cherry-picks Phases 1–2 of `garrytan/gstack`'s `spec` skill ("turn vague intent into a precise,
executable spec") — rewritten in prose, not its runtime. Its actual mechanics run five phases
through issue-filing and worktree-spawning; only the Phase 1–2 questioning discipline is
cherry-picked here. Phases 3–5 are covered elsewhere in this pipeline, in its own way, by
`senior-engineer` and `plan-feature`.

## Command

```
/define-epic "<what to build>"
```

## Sequence

1. Read `PROJECT.md` (including `vision_context`, if `founder-vision` has run) for grounding.
2. Check `log-decision` for any prior entry referencing this epic (resume, not restart).
3. **Phase 1 — Why** (adapted from gstack `spec`): ask until all five are answered without
   hand-waving:
   - Who is affected (end user, internal team, just you)?
   - What's the current behavior/situation (verified, not assumed)?
   - What should it be instead?
   - Why now?
   - How will we know it's done — an observable, measurable outcome?
4. **Phase 2 — Scope and boundaries** (adapted from gstack `spec`):
   - What's explicitly out of scope?
   - What existing systems/features does this touch?
   - Any ordering constraints?
   - What's the smallest version that delivers the value (MVP cut)?
   - What are the failure modes / rollback options?
5. Create the epic in the configured ticketing system (`PROJECT.md.ticketing.system`) — or, if
   `none`, write an epic brief file that `senior-engineer` reads the same way. Hand off the epic
   reference to `senior-engineer`.
6. Call `log-decision` (write) — first entry for this epic: scope, explicit non-goals, why-now.

## Open questions

1. Overlap with `plan-feature`'s own interview (scope, non-goals) — since `plan-feature` now runs
   *underneath* an epic (per-feature), does it re-ask anything this skill already answered? Needs
   the same "don't ask near-duplicate questions twice" resolution `plan-feature`'s own open
   question 1 already flags, just one level higher up.
2. What does an "epic" concretely look like when `ticketing.system: none`? A single markdown file
   at a fixed path (`epics/<slug>.md`) that `senior-engineer` and any spawned `plan-feature` calls
   read for context?
3. Can an epic be filed without committing to how many features/tickets it becomes — i.e. is that
   count decided here, or genuinely left open until `senior-engineer` investigates? (Current
   design: left open — this skill only scopes the *ask*, not the *breakdown*.)

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 first — it affects both this file and `plan-feature/DESIGN_NOTES.md`.
2. Read gstack's `spec/SKILL.md` Phase 1–2 sections in full (already excerpted during source
   selection) and write the exact question wording, reviewed against a real epic-shaped example.
3. Resolve open question 2 — needed before writing the `ticketing.system: none` path.
