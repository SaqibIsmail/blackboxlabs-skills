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
5. Create the epic in Jira (the expected ticketing system, per `define-project`'s resolved
   Jira-connection interview — **resolved 2026-09-12: no `ticketing.system: none` fallback
   designed here**, a real epic always gets created). Hand off the epic reference to
   `senior-engineer`.
6. Call `log-decision` (write) — first entry for this epic: scope, explicit non-goals, why-now.

## Open questions

1. ~~Overlap with `plan-feature`'s own interview...~~ **Resolved (2026-09-11):** "smart hand-off"
   — `senior-engineer` passes this skill's Phase 1–2 answers to `plan-feature` as pre-filled
   context; `plan-feature` only asks about what that leaves unanswered. See
   `plan-feature/DESIGN_NOTES.md` and `senior-engineer/DESIGN_NOTES.md` step 3.
2. ~~What does an "epic" concretely look like when `ticketing.system: none`?~~ **Resolved
   (2026-09-12): dropped.** Jira is always on for this pipeline's actual use — no fallback
   designed. Revisit only if this pipeline is ever reused on a project without Jira.
3. Can an epic be filed without committing to how many features/tickets it becomes — i.e. is that
   count decided here, or genuinely left open until `senior-engineer` investigates? **Confirmed
   (2026-09-12), consistent with `senior-engineer`'s own resolution:** left open — judgment via
   questions at `senior-engineer`-time, no fixed rule, same principle as ticket right-sizing. This
   skill only scopes the *ask*, never the *breakdown*.

## TODOs (block turning this into a real `SKILL.md`)

1. Read gstack's `spec/SKILL.md` Phase 1–2 sections in full (already excerpted during source
   selection) and write the exact question wording, reviewed against a real epic-shaped example.
2. Write the Jira epic-creation call concretely (which `jira-integration` fields, what the epic
   description template looks like) — now load-bearing since Jira is the confirmed path.
