# /build-backend — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.

## What this is

The backend build agent — structurally the mirror of the existing `build-frontend`, but for
`[BE]`-tagged tasks. No `impeccable`/`taste-skill` dependency (that tooling is frontend-UX-only);
instead wraps `github/spec-kit`'s `/speckit.tasks` (BE-subset) → `/speckit.implement`, plus one
real addition spec-kit doesn't provide on its own.

**Why TDD discipline is added on top of spec-kit, not redundant with it:** confirmed via
spec-kit's own templates — `/speckit.implement` carries no dev persona at all; it just drives
whatever generic agent runs it through `tasks.md` in order, and its own docs mark tests as
"OPTIONAL... only if explicitly requested." Originally scoped to cherry-pick from BMAD-METHOD's
`bmad-agent-dev` (persona "Amelia") for this — a genuine test-first discipline: red, green,
refactor, tied to acceptance-criterion IDs. **Superseded by a stronger candidate found after
that scoping: `obra/superpowers` (284.8k★ — over 5x BMAD's 52.9k★).** Its
`test-driven-development` skill enforces TDD as a stated "Iron Law" with harder mechanical
guardrails than BMAD's persona-based approach: delete any code written before its test existed,
a mandatory watch-the-test-fail step (never trust a test you haven't seen fail first), and an
explicit anti-rationalization table naming specific excuses ("I'll test after," etc.) and
refusing them. It also has a separate `verification-before-completion` skill — a hard gate
forbidding any success/completion claim without pasting fresh command output in the same turn —
which is a real, independently useful addition to this skill's own Step 6 ("present to user").
Both need weighing against BMAD's version (see open question 4) before this is finalized — not
a settled swap yet, since the plan this repo's owner approved specifically named BMAD and this
is a real change to that, not just an implementation detail.

## Command

```
/build-backend <feature>
```

## Sequence (mirrors `build-frontend`'s shape, backend-flavored)

1. **Bootstrap** — read `PROJECT.md` for backend stack/coding-standards doc paths; confirm
   `specs/<feature>/{spec.md,plan.md,tasks.md}` exists (from `plan-feature`/`assign-tasks`); load
   this feature's `[BE]`-tagged task subset.
2. **Check `log-decision`** — before implementing, read for any prior decision constraining this
   feature (a plan deviation recorded earlier, an accepted-risk note, etc.).
3. **Build, task by task, test-first** — for each `[BE]` task: write the test against its
   acceptance-criterion ID first (red), implement to pass (green), then refactor. This is the one
   genuinely new mechanical rule this skill adds over spec-kit's bare `/speckit.implement`.
4. **Self-QA** — unlike `build-frontend` (which has `impeccable critique`/`audit` for UX/a11y/perf),
   backend has no equivalent dependency skill yet. Placeholder for now: run the project's own
   existing checks (`docs/standards/backend.md`'s rules, the standards-scoring CI categories,
   if run locally) as a substitute self-QA pass — see open question 2.
5. **Call `log-decision`** — only if a non-obvious deviation from `plan.md` actually happened
   during implementation, same fuzzy-by-design trigger as `build-frontend`'s own entry in the
   `log-decision` design notes.
6. **Present to user** — summary of tasks completed, tests written, and anything skipped/deviated
   with why; leave unstaged for review, same as `build-frontend`'s and `page-builder`'s existing
   "don't commit, this is for review first" convention.

## Open questions

1. `build-frontend` has a real "Focus flag" / "overdrive" step (one deliberately sophisticated
   flourish per page, chosen from 2-3 candidates via `AskUserQuestion`). Backend has no obvious
   equivalent — is there a backend-appropriate analogue (e.g. one deliberate resilience/perf
   choice per feature, similarly proposed as 2-3 candidates with trade-offs), or does backend
   simply not need this step at all?
2. Self-QA placeholder (step 4) is genuinely underspecified — should this skill actually invoke
   the project's `standards-score.mjs`-style tooling if present (per `blackboxlabs`'s own
   pattern), fall back to a generic backend checklist if not, or skip self-QA entirely and rely
   entirely on `resolve-pr-comments` + Greptile downstream to catch issues?
3. TDD discipline assumes `spec.md` actually has acceptance criteria with IDs stable enough to
   tie tests to — needs confirming spec-kit's real `/speckit.specify` output always produces
   these in a consistent, parseable format (not verified yet, only assumed from BMAD's side of
   the equation).
4. **BMAD's `bmad-agent-dev` vs `obra/superpowers`' `test-driven-development` +
   `verification-before-completion`** — which is the actual cherry-pick source for this skill's
   Step 3/6? Superpowers is far more adopted and, on the description alone, more mechanically
   strict (an "Iron Law" plus a hard completion-gate) rather than persona-flavored guidance.
   BMAD's version is tied to spec-kit's acceptance-criterion IDs more naturally, since both are
   already part of this pipeline's spec/plan/tasks flow — superpowers' skill isn't spec-kit-aware
   and would need its own adaptation to key off criterion IDs the same way. Worth reading
   superpowers' actual skill file (not just the description) before deciding, same as BMAD's own
   dev-agent file was read directly rather than assumed from its README.

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 3 first — if spec-kit's spec format doesn't reliably number acceptance
   criteria, the whole TDD-tied-to-criterion-ID mechanic needs a fallback.
2. Decide open question 2 before writing the self-QA step in detail.
3. Resolve open question 4 — read `obra/superpowers`' actual `test-driven-development` and
   `verification-before-completion` skill files in full before finalizing which source (or both,
   combined) this skill's TDD/completion-gate steps are built on.
