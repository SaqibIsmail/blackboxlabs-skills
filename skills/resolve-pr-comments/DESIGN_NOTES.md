# /resolve-pr-comments — Pipeline Design (draft, not yet a working skill)

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
3. Decide, informed by an adversarial-review *technique* as a validation lens (not the actual
   reviewer agents) — originally scoped to BMAD-METHOD's `bmad-code-review`: its "Verification
   Gap" discipline (would a test actually catch this if it's real?) and its "claims-check only
   after path-tracing" discipline (verify the code's actual behavior before trusting the
   comment's framing, and before trusting the decision log's framing either). **Two stronger/
   broader candidates found after that scoping, both needing the same read-the-actual-file
   treatment BMAD's got before picking one:** `obra/superpowers` (284.8k★) dispatches code review
   to a subagent given only precisely-crafted context — never the coordinator's own session
   history — which maps well onto this skill's own "read the comment fresh, don't inherit bias
   from however the PR conversation went" need. `garrytan/gstack` (132.5k★) covers 8 specialist
   review categories (api-contract, data-migration, maintainability, performance, red-team,
   security, simplification, testing) versus BMAD's 2 (edge-case-hunter, verification-gap) —
   broader, but reportedly more coupled to its author's own infra (telemetry, a custom
   AskUserQuestion decision-brief format), so may cherry-pick less cleanly. See open question 4.
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
4. BMAD vs superpowers vs gstack for the validation-lens technique in step 3 — not settled; the
   plan this repo's owner approved named BMAD specifically, so swapping needs their sign-off, not
   a unilateral pick, even though superpowers' star count and isolated-context dispatch pattern
   look like a better fit on paper.

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 3 — a real permissions/mechanism check, not a design preference.
2. Write the actual GitHub Actions workflow YAML this skill's `SETUP.md` will tell people to add
   to their target repo.
3. Resolve open question 4 — read `obra/superpowers`' code-review-dispatch skill and
   `garrytan/gstack`'s `review` skill in full (not just their descriptions) before finalizing.
