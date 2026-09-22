# /resolve-pr-comments — Pipeline Design (draft, not yet a working skill)

**Not final (2026-09-11):** flagged for re-discussion now that the epic layer
(`founder-vision`→`define-epic`→`senior-engineer`) exists upstream. The PRs this skill reacts to
now trace back to epic-linked tickets from `senior-engineer` — revisit as a whole before treating
it as settled.

Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.

## What this is

Automates a workflow that already exists today as a **manual habit**, documented almost verbatim
in `blackboxlabs/docs/hosting.md`: "the dispute loop when Greptile flags something: fix it if
it's a real issue; if it isn't, reply to the inline comment explaining why (Greptile reads
replies and remembers), then re-trigger a review by commenting on the PR." This skill is that
loop, automated, with one addition current practice doesn't have: **consulting the feature's
`log-decision` entry for prior rationale before touching anything** — research across existing
open-source PR-automation tools (Anthropic's own `claude-code-action`, `pullfrog`, CodeRabbit's
`autofix`) confirmed none of them do this; it's a genuine, unfilled gap, not a reinvention.

Runs on `anthropics/claude-code-action` (official, MIT, reacts to PR review comments/`@mentions`
in a GitHub Actions runner that loads this repo's own `.claude/skills/`) as the reactive
substrate — no local Claude Code session can see an external PR comment land on its own, which is
exactly the "reactive edge" this pipeline's orchestration design already carved out for exactly
one external tool.

## Trigger

A GitHub Actions workflow in the *target* repo (not vendored here — declared in `SETUP.md`),
firing on PR review comment events, invoking this skill via `claude-code-action`'s prompt.

## Sequence

1. Read the comment (from Greptile or a human reviewer) and the file/line it's attached to.
2. **Consult `log-decision`** for this feature/PR — read before editing anything. If a prior
   decision entry explains *why* the flagged code is the way it is, that's the deciding context
   for step 3.
3. Decide, informed by `obra/superpowers`' review/verification technique — **settled
   (2026-09-10) over BMAD-METHOD's `bmad-code-review`**, after reading superpowers'
   `skills/requesting-code-review/SKILL.md` in full: it dispatches review to a subagent given
   only precisely-crafted, isolated context — a description of what was built, the requirements,
   the BASE/HEAD SHAs, and the diff itself — explicitly **withholding** the coordinator's own
   session history, reasoning, or prior failed approaches. Its stated reason: "reviewing the diff
   inline burns the context window you need to keep driving the work." This maps directly onto
   this skill's own need: evaluate a PR comment against the actual diff and prior decision-log
   entries, not against however the surrounding PR conversation happened to frame it. Combined
   with `verification-before-completion` (same source, see `build-backend/DESIGN_NOTES.md`):
   before this skill claims a comment is "resolved" — whether by fixing it or by disputing it —
   it must have actually re-run the relevant verification command fresh and read its output, not
   asserted the fix works. (`garrytan/gstack`'s broader 8-category `review` skill was also
   surfaced but not chosen — reportedly more coupled to its author's own infra, e.g. telemetry
   and a custom `AskUserQuestion` decision-brief format, so it cherry-picks less cleanly than
   superpowers' narrower, more portable pattern.)
   - **The comment is valid** → fix it, following the project's own coding standards.
   - **The comment is already addressed by a recorded decision** → reply explaining why, citing
     the specific decision entry, and re-trigger review (mirrors the exact manual step already
     documented in `hosting.md`).
4. **Call `log-decision`** — always, recording why the comment was resolved the way it was
   (a fix, or a reasoned dispute) — this becomes the next entry future runs of this same loop
   will read.

## Open questions

1. `claude-code-action` auto-loads root `CLAUDE.md` on every run — is that sufficient context
   pass-through on its own, or does the workflow prompt need to explicitly point it at
   `PROJECT.md`/the vault path too, since `CLAUDE.md` alone won't mention either?
2. What happens when `log-decision` has no entry for this feature at all (an older PR predating
   this pipeline's adoption, or a feature that skipped `plan-feature`)? Falling back to "resolve
   with no prior context" is the honest default, but should the skill say so explicitly in its
   reply/log entry rather than silently proceeding as if context existed?
3. Re-triggering a Greptile review from an Actions-run agent — needs confirming the actual
   mechanism (commenting on the PR, per `hosting.md`'s existing manual instructions) is something
   `claude-code-action`'s permissions allow it to do on its own, not just read comments.

**Resolved:** BMAD vs. superpowers vs. gstack for the validation-lens technique — settled on
superpowers (see above).

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 3 — a real permissions/mechanism check, not a design preference.
2. Write the actual GitHub Actions workflow YAML this skill's `SETUP.md` will tell people to add
   to their target repo.
3. Decide what "precisely-crafted, isolated context" concretely means for *this* skill's version
   of the pattern — likely: the comment text, the file/line diff, and the relevant `log-decision`
   entry, explicitly not the PR's full comment thread history or Greptile's own prior rounds on
   this same PR. Write that context-assembly step out concretely before finalizing `SKILL.md`.
