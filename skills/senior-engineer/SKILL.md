---
name: senior-engineer
description: >
  Takes one epic (from define-epic) and turns it into small, individually-implementable Jira
  tickets. Investigates the codebase before asking anything technical, mines existing behavior
  into a baseline spec when the epic touches undocumented existing code, checks for a UI
  reference and spikes a research ticket when one exists, decides how many spec-kit features the
  epic needs, drives plan-feature per feature, then right-sizes the resulting tasks into tickets
  small enough for a coding agent to implement one without also holding an unrelated concern in
  its head. Supersedes assign-tasks. Use when the user invokes /senior-engineer with an epic
  reference, or asks to break an epic down into tickets.
argument-hint: '<epic-key-or-ref>'
user-invocable: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - AskUserQuestion
  - Skill
license: MIT
---

# /senior-engineer

Investigates one epic and turns it into small, ordered, individually-implementable tickets —
never guessing at scope, never letting a ticket grow bigger than one coding agent can hold in its
head at once.

## Step 1 — Read context

Read `PROJECT.md` and the epic (key/ref from the command argument, fetched via
`jira-integration`'s `jira_get_issue`).

## Step 2 — Check prior decisions

Call `log-decision` (via `Skill`) in query mode with the epic's name/key. If entries exist, this
is a resume: read them aloud and treat their `context`/`decision` as already-established, not
something to re-derive from scratch during investigation.

## Step 3 — Investigate (code-grounded)

Before asking anything technical: `Grep`/`Glob`/`Read` the actual codebase for the systems the
epic touches. Ground every question in what was actually found — cite `file:line`. Never ask
"what file should I look at?" — find it yourself first.

Map the epic's request to evidence:

- **Concrete file/symbol named in the epic** (e.g. "the dashboard is slow", "auth.ts fails"):
  `Grep` for the symbol, `Read` the file, cite `path:line` in the first question you ask.
- **Project-level ask** ("rethink our auth strategy", "add rate limiting"): read the project
  structure — manifest deps, the relevant top-level directory, any existing
  `docs/<topic>.md` — and cite what you found before asking anything.

## Step 4 — Mine existing behavior, if any

If Step 3 found the epic touches an existing capability, check for a baseline spec at the fixed
path `openspec/specs/<capability>/spec.md` (same path convention everywhere this pipeline runs,
not project-configurable):

- **No baseline exists:** run `spec-miner` (via `Skill`) against *only this specific capability* —
  do not run its own "present the whole codebase's capability list, ask which to mine" step; this
  skill already knows the exact capability from Step 3, so hand `spec-miner` that name directly
  and skip straight to its per-module mining phase.
- **Baseline exists — staleness check:** `spec-miner`'s own output always includes a `Last
  verified: YYYY-MM-DD (commit <sha>)` line. Run `git log -1 --format=%H -- <capability's source
  files>` and compare against that recorded SHA. If they differ, the code changed since mining —
  re-run `spec-miner` against the same capability to refresh the baseline. If they match, use the
  existing baseline as-is.
- **Genuinely greenfield epic:** skip this step entirely — nothing to mine.

The (possibly freshly-mined) baseline becomes grounding for both the rest of this investigation
and for `plan-feature` later (Step 6 hands it along so the new spec is written as a delta against
known behavior, not from scratch).

## Step 5 — UI check

Ask whether a UI/design idea already exists (mockup, Figma, reference screenshot/site, or none
yet).

- **No idea exists:** propose a design spike ticket first (e.g. "Design: `<page/flow>` layout and
  states"). If the user wants to proceed anyway rather than block on it: the resulting
  functional ticket(s) describe the UI requirement needed to complete the task, plus an explicit
  **"No design decided yet"** line in the description — `build-frontend` makes the actual design
  decision when it picks the ticket up.
- **An idea exists with a concrete reference** (a live site, an existing component, an animation
  seen somewhere): **always** spike a separate research ticket first — never let a build ticket
  "just match the reference" from memory (given only a description, a coding agent reproduces
  something different each time). The research spike, using **Playwright**:
  - Live site: navigate to it, read the real DOM/CSS/JS, computed styles, animation
    timing/easing, and network requests for the libraries it loads.
  - Existing component: read its real source, not just how it renders.
  - Write up the *actual mechanism* — library, exact CSS properties/keyframes, DOM structure,
    state transitions — then map it onto this project's own stack (what's already available,
    what's missing, the concrete component/file it becomes here).
  - Embed these findings **directly in the resulting build ticket's description** (not just a
    link) — the build ticket is created after and depends on this spike. Also call `log-decision`
    (write) with the mapping, so the grounding survives even if the build ticket runs in a
    separate session.
- **An idea exists with no concrete reference** (verbal description only): no research spike
  needed — proceed straight to Step 6.

## Step 6 — Decide the feature split

Does this epic need one `plan-feature` pass or several? Informed judgment, using BMAD's real
epic-design criteria as the concrete questions to ask (not a rigid formula):

1. **Is the solution direction already validated** (architecture/UX/approach settled, unlikely to
   change)? If yes, prefer **fewer, larger** features — splitting adds coordination overhead with
   no real benefit.
2. **Is there a genuine risk/feedback boundary** — could learnings from one part plausibly change
   how the next part should be built? If yes, that's where a split earns its keep.
3. **Does the work repeatedly touch the same core files/component** across what looked like
   separate features? If yes, **consolidate into one feature** with the split happening at the
   ticket level instead (ordered stories within one feature), not at the feature level — this
   avoids each "feature" churning the same files independently.

Each resulting feature must be **standalone**: it delivers complete, usable functionality on its
own and must not require a not-yet-built feature to make sense (a later feature may build on an
earlier one, never the reverse).

For each identified feature, call `/plan-feature` (via `Skill`, a real sub-invocation — not
inlined) **passing along as context**: `define-epic`'s Phase 1–2 answers, this skill's own
investigation findings from Step 3, and any baseline spec from Step 4. `plan-feature` only
interviews about what that context leaves genuinely unanswered — it does not re-ask scope/non-goals
already covered at the epic level.

## Step 7 — Right-size into tickets

Walk each feature's `tasks.md`. Group or split spec-kit's `T0xx` tasks into tickets using two
combined principles:

- **Right-sizing** (superpowers): a ticket is as small as the unit that carries its own test cycle
  and is worth a reviewer's independent gate — split only where a reviewer could approve one
  ticket while rejecting its neighbor. This is *why* an animation gets its own ticket: a judgment
  call this principle gives language to, not a fixed rule ("animations are always separate").
- **No forward dependencies** (BMAD): a ticket must be completable using only what prior tickets
  in the same feature have already delivered — never "this only works once ticket N+2 lands."
  Order tickets so each is independently buildable in sequence.

For each ticket, write a description in BMAD's story format:

```
As a <user_type>,
I want <capability>,
So that <value_benefit>.

**Acceptance Criteria:**

Given <precondition>
When <action>
Then <expected_outcome>
And <additional_criteria, one per line as needed>
```

Classify each ticket as `task`, `spike` (research/design unknowns — includes the Step 4/5 spikes
above), or `bug` (only when the epic is itself a fix).

## Step 8 — Present the draft

Show all proposed tickets — title, type, story/AC text, linked `T0xx` IDs, and which feature/spec
they come from. Ask: "Does this capture it? What's wrong?" Iterate until confirmed before
creating anything.

## Step 9 — Create tickets

Via `jira-integration`'s `jira_create_issue` (or delegate to spec-kit's own
`/speckit.taskstoissues` when `PROJECT.md.ticketing.system: github-issues`), each linked to the
parent epic (`jira_create_issue_link`) and to its originating `spec.md`/`plan.md`.

**Every ticket's description also embeds a path/link back to its relevant `log-decision`
entry(ies)** — the epic's decision file from `define-epic`, and the feature's from `plan-feature`
if it wrote its own — using the `path` `log-decision` returned when it wrote them. Without this, a
ticket in Jira has no way back to why it was scoped the way it was.

After creation, call `log-decision` (update mode) to enrich the epic's (and relevant feature's)
decision entry with `ticket-refs: [<created ticket keys>]`.

## Step 10 — Log non-obvious deviations

If the investigation surfaced a non-obvious scoping call not already captured (e.g. "epic split
into two features because X," "deferred Y as a separate spike because Z"), call `log-decision`
(write) recording it. Skip this step if nothing non-obvious came up — not every run needs a new
entry beyond what Steps 5/9 already wrote.

## Relationship to `assign-tasks`

Replaces it. `assign-tasks`'s FE/BE bracket-tag convention on spec-kit's raw `tasks.md` grammar was
too coarse for the actual ask (ticket granularity, not just FE/BE labeling) — this skill's
right-sizing step subsumes that classification; a ticket still ends up FE-, BE-, or shared-scoped,
just as one axis of a richer split, not the only one.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- `spec-miner` is always targeted at the one capability Step 3 already identified — never asked
  to present a whole-codebase capability list, since this skill isn't onboarding the whole repo.
- The feature-split criteria in Step 6 are diagnostic questions to inform judgment, not a formula
  — a "yes" to one doesn't mechanically force a split or merge.
- No built-in "fix a bad ticket split after filing" mode — a wrong split found later is fixed by
  editing Jira directly.
