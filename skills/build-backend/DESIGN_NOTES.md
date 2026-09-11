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
`bmad-agent-dev` (persona "Amelia"); **decided (2026-09-10, after reading both sources'
actual skill files, not just descriptions) to use `obra/superpowers` instead** (284.8k★ — over
5x BMAD's 52.9k★). Its `skills/test-driven-development/SKILL.md`, read in full: an "Iron Law" —
**"NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"** — with any code written before its test
required to be deleted and reimplemented ("delete means delete," no keeping it "as reference");
a mandatory watch-the-test-fail step ("MANDATORY. Never skip.") to prove the test actually
detects the missing behavior; and a named anti-rationalization table ("too simple to test,"
"I'll test after," "already manually tested," "keep as reference") that each trigger a restart
rather than an exception. Confirmed this skill is **purely procedural/generic — it has no
concept of an acceptance-criterion ID**, unlike BMAD's version which is naturally spec-kit-aware.
So this pipeline adds that bridge itself: superpowers' mechanical rules (Iron Law, watch-fail,
anti-rationalization, delete-means-delete) are the enforcement; *this* skill is what ties each
test to the `spec.md` acceptance-criterion ID it's meant to satisfy, since neither source does
that on its own.

Also adopting `skills/verification-before-completion/SKILL.md` (read in full) for this skill's
own Step 6: **"NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"** — identify the
verification command, run it fresh, read the full output (exit code, failure count), confirm it
actually supports the claim, only then state it. No hedging language ("should," "probably"), no
trusting a sub-agent's self-report without independently re-running the command.

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
6. **Present to user** — per `verification-before-completion`: run the real verification command
   (the project's test suite, at minimum) fresh, read its actual output, and only then state
   which tasks are done — never "should be passing." Summary of tasks completed, tests written,
   and anything skipped/deviated with why; leave unstaged for review, same as `build-frontend`'s
   and `page-builder`'s existing "don't commit, this is for review first" convention.

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
   these in a consistent, parseable format. (Confirmed separately: superpowers' own TDD skill has
   no ID scheme of its own, so this pipeline's criterion-ID bridge is needed regardless of that
   answer — it just determines whether the bridge is clean or needs a fallback naming scheme.)

**Resolved:** BMAD vs. superpowers for the TDD/completion-gate technique — settled on
superpowers (see above), after reading both sources' actual skill files rather than deciding
from descriptions alone.

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 3 first — if spec-kit's spec format doesn't reliably number acceptance
   criteria, the criterion-ID bridge needs a fallback naming scheme.
2. Decide open question 2 before writing the self-QA step in detail.
3. Write the actual criterion-ID bridge: the concrete rule for "this test's `it()`/`describe()`
   block name (or a comment) references acceptance-criterion `AC-3` from `spec.md`" — the one
   piece neither spec-kit nor superpowers provides, now that the source skills themselves are
   settled.
