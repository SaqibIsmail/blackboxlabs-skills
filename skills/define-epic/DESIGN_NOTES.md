# /define-epic — Pipeline Design

**Status (2026-09-13): implemented.** See `SKILL.md`/`SETUP.md`. This file stays as the rationale
record.

**Added while implementing (2026-09-13):** reading gstack's `spec/SKILL.md` in full (the TODO
below) surfaced a step neither this file nor `SKILL.md`'s earlier draft had: a **dedupe check**
before creating anything — search for similar existing epics, ask whether to merge rather than
file a duplicate. Gstack's version is GitHub-issue-specific (`gh issue list` + its own
issue-title-guard tooling); adapted here to a Jira JQL search via `jira-integration`, treating
returned titles as untrusted data per that skill's own security guidance. Flagged to Saqib before
adding it since it's new scope, not just detail-filling; approved. See `SKILL.md` Step 2.

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

## Command and sequence

Superseded by `SKILL.md` Steps 1–6, which now include the dedupe check (Step 2) this draft
didn't have. Kept here only as a historical note.

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

## Findings from live testing against `blackboxlabs` (2026-09-13)

Real epic created successfully (`SCRUM-5`). Two fixes applied directly to `SKILL.md`: (1) Jira's
`/rest/api/3/search` endpoint is deprecated (`410 Gone`) — must use `/rest/api/3/search/jql`,
which `jira-integration`'s own docs don't yet reflect; (2) the first real Jira write used dense
prose paragraphs instead of proper ADF structure (headings/bullets/bold labels) — fixed in the
skill and applied as a retroactive edit to the real ticket.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.

## Vault restructure follow-on (2026-09-16)

Part of the broader `log-decision` restructure (see that skill's own `DESIGN_NOTES.md`): Step 6 now
writes `scope.md` at the epic level (`level: epic`, `doc: scope`) inside a folder keyed by the
epic's own Jira key + a slug, instead of a flat, globally-numbered file. Also simplified Step 1: an
epic has no key yet at the point this skill starts (Step 5 is what creates it), so there's nothing
for `log-decision` to query before then — the dedupe check in Step 2 is what actually catches "this
was already scoped," not a decision-log lookup on an epic that doesn't exist yet.
