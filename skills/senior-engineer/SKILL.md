---
name: senior-engineer
description: >
  Takes one epic (from define-epic) and turns it into small, individually-implementable Jira
  tickets. Investigates the codebase before asking anything technical, mines existing behavior
  into a baseline spec when the epic touches undocumented business logic, mines existing UI
  component interfaces when it touches undocumented visual/component code, checks for a UI
  reference and spikes a research ticket when one exists, decides how many spec-kit features the
  epic needs, drives plan-feature per feature, then right-sizes the resulting tasks into tickets
  small enough for a coding agent to implement one without also holding an unrelated concern in
  its head. Supersedes assign-tasks. Use when the user invokes /senior-engineer with an epic
  reference, or asks to break an epic down into tickets.
argument-hint: '<epic-key-or-ref>'
user-invocable: true
allowed-tools:
  - Read
  - Write
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

Read `PROJECT.md` (the project-root stub, then `<vault_path>/<company-slug>/project.md` for the
rest — see `define-project`) and the epic (key/ref from the command argument, fetched via
`jira-integration`'s `jira_get_issue`).

## Step 2 — Check prior decisions

Call `log-decision` (via `Skill`) in query mode with the epic's key. This walks up from the epic
level (`scoping-calls.md`, `scope.md`) automatically — see `log-decision`'s own query mode. If
entries exist, this is a resume: read them aloud and treat their `context`/`decision` as
already-established, not
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

Step 3's findings split into two different kinds of "existing thing this epic touches," each
needing a different mining approach — `spec-miner`'s Requirement/Invariant model fits business
logic; it doesn't fit UI/visual/component code, which mostly has no WHEN→THEN triggers to extract.

**4a — Existing business-logic capability.** Check for a baseline spec at the fixed path
`<vault_path>/<company-slug>/openspec/specs/<capability>/spec.md` (company-level in the vault, not
per-epic — a capability is reused across epics, same path convention everywhere this pipeline
runs, not project-configurable):

- **No baseline exists:** run `spec-miner` (via `Skill`) against *only this specific capability* —
  do not run its own "present the whole codebase's capability list, ask which to mine" step; this
  skill already knows the exact capability from Step 3, so hand `spec-miner` that name directly
  and skip straight to its per-module mining phase.
- **Baseline exists — staleness check:** `spec-miner`'s own output always includes a `Last
  verified: YYYY-MM-DD (commit <sha>)` line. Run `git log -1 --format=%H -- <capability's source
  files>` and compare against that recorded SHA. If they differ, the code changed since mining —
  re-run `spec-miner` against the same capability to refresh the baseline. If they match, use the
  existing baseline as-is.

**4b — Existing UI/visual/component code the epic builds on or extends.** Check for
`<vault_path>/<company-slug>/openspec/components/<component-name>/interface.md` (same fixed-path
convention, sibling to `openspec/specs/`, also company-level not per-epic):

- **No interface doc exists:** mine it directly (no external skill for this — an original
  technique, since none of BMAD/gstack/superpowers/ECC cover UI-component interface extraction).
  Extract:
  - **Props/API surface** — every prop, its type, required/optional, defaults. Explicitly note
    when a component has **no props** (self-contained, not configurable from outside) or
    **hardcodes content** that looks like it should be configurable (e.g. literal text baked into
    the component) — these are exactly the details that cause a second use case to silently break.
  - **Behavior summary** — what it does, in plain terms (not a full spec, just enough to orient).
  - **Dependencies** — libraries used, and required ancestors/context (e.g. "must be wrapped by
    `X`" — verify this by reading the code, not by assuming from how it's currently used).
  - **Key constants/config** — cite `file:line`, same discipline as `spec-miner`'s `enforced`
    field.
  - **Accessibility notes** — anything already handled (e.g. `prefers-reduced-motion`) that a new
    use case must not silently drop.
  - **Existing integration points** — `Grep` for every current usage site, cite `file:line`. Often
    the most important finding: a component already wired into a specific page is a real
    constraint on how it can be reused, not a blank slate.
  - **Constraints/gotchas** — performance sensitivity, anything that would break if copied naively.

  Write to `<vault_path>/<company-slug>/openspec/components/<component-name>/interface.md`, headed
  with the same freshness line `spec-miner` uses: `> Mined: YYYY-MM-DD (commit <sha>)`, `<sha>`
  from `git log -1 --format=%H`. The staleness check's `git log` always targets the *project's*
  source files, regardless of where the mined doc itself lives — moving the doc into the vault
  doesn't change what it's compared against.
- **Interface doc exists — staleness check, identical mechanism to 4a:** run `git log -1
  --format=%H -- <component's source files>` and compare against the doc's own recorded `Mined:
  (commit <sha>)` line. If they differ, the component changed since it was mined — re-mine it,
  same process as above. If they match, use the existing doc as-is.

**Genuinely greenfield epic, no existing capability or component touched:** skip Step 4 entirely.

The (possibly freshly-mined) baseline(s) become grounding for both the rest of this investigation
and for `plan-feature` later (Step 6 hands them along so new work is written as a delta against
known behavior/interfaces, not from scratch).

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
investigation findings from Step 3, any baseline spec from Step 4, **and Step 5's UI-check outcome
— whether a reference was flagged, and whether it's already been researched.** This last item
matters as much as the others: without it, an unresearched reference only lives in this skill's own
judgment and can silently fail to reach `tasks.md` as a real research task, which is how a build
ticket can end up adapting a reference with no spike behind it (found live, 2026-09-14 —
see `brain/decisions/0003`/`0004` in a tested project for the concrete case). `plan-feature` only
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

A task that researches an unfamiliar reference before a dependent build task exists precisely
because Step 6 handed that flag through to `plan-feature` — group it as its own `spike` ticket
ordered before the build ticket it unblocks, never folded into the same ticket. This is the actual
mechanism behind Step 4/5's spike rule, not a separate check to remember here: if `tasks.md`
contains the research task, this step's own grouping logic (risk-boundary criterion) naturally
splits it out.

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

**Use real Atlassian Document Format structure, not a wall of prose** — headings, bullet lists,
and bold labels (`Given:`, `When:`, `Then:`) for the story/AC content, mirroring the actual API
call shape (`content` blocks), not one dense paragraph per section.

**Ticket type mapping** (per `PROJECT.md.ticketing.issue_types_available`): when the project has
no native `Bug`/`Spike` issue type (common in a default Jira template), use `Task` plus a label
(`spike`, `bug`) instead of forcing a nonexistent type.

**Creating a ticket also seeds its own vault file.** Call `log-decision` (write, `level: ticket`,
passing the epic/story/ticket key+slug chain) with this ticket's initial scoping context — this
both creates `<vault_path>/<company-slug>/<epic-key>-<slug>/<story-key>-<slug>/<ticket-key>-<slug>.md`
(or directly under the epic if there's no Story tier) and returns its path. Embed that path in the
ticket's own description, alongside a link to the epic's `scope.md` and (if this ticket's feature
has one) the relevant `scoping-calls.md`. Because a ticket's own file is created *at* ticket-creation
time now, not just linked to a pre-existing epic/feature entry, there's no ordering problem the way
there was under the old flat-numbered scheme — see Step 10 for what still comes later.

**For a large epic, group tickets under an intermediate tier, not one flat list.** Jira has no
native nested-Epic support (without premium Advanced Roadmaps) — use the project's `Story` issue
type as the grouping/theme tier (one per logical section of the epic), with the concrete tickets
created as `Subtask`s under each `Story` (`parent` field points to the `Story`, not the `Epic`
directly). Post any **cross-group integration notes** (dependencies between the groups themselves,
shared mechanisms one group's work reuses from another) as a comment on the epic — the place a
reader looking at the whole epic will actually see it, not buried in one subtask's description.

After creation, call `log-decision` (update mode) to enrich the epic's (and relevant feature's)
decision entry with `ticket-refs: [<created ticket keys>]`.

## Step 10 — Log non-obvious deviations

If the investigation surfaced a non-obvious scoping call not already captured (e.g. "epic split
into two features because X," "deferred Y as a separate spike because Z," a reference that didn't
match intent and how it was resolved), call `log-decision` (write) at whichever level actually owns
it — one ticket → that ticket's own file; several tickets in one story → that story's
`scoping-calls.md`; spans stories → the epic's `scoping-calls.md`. Skip this step if nothing
non-obvious came up — not every run needs a new entry beyond what Steps 5/9 already wrote.

**No retroactive back-link problem anymore.** Under the old flat-numbered scheme, a decision logged
here didn't exist yet when Step 9 created the tickets, so it had to be added as a follow-up
comment after the fact. Now, appending a dated entry to an already-existing story/epic
`scoping-calls.md` (or a ticket's own file) doesn't require anything new to be *found* — the file
and its path are already known from Step 9. Post a follow-up comment on the epic/ticket only if the
entry is genuinely new information a reader wouldn't otherwise think to go looking for.

## Relationship to `assign-tasks`

Replaces it. `assign-tasks`'s FE/BE bracket-tag convention on spec-kit's raw `tasks.md` grammar was
too coarse for the actual ask (ticket granularity, not just FE/BE labeling) — this skill's
right-sizing step subsumes that classification; a ticket still ends up FE-, BE-, or shared-scoped,
just as one axis of a richer split, not the only one.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- `openspec/specs/*` and `openspec/components/*` live at **company** level in the vault, not
  per-epic — a mined capability/component is meant to be found and reused by a *later, unrelated*
  epic, so it can't live nested inside the epic that happened to mine it first.
- Every ticket this skill creates gets its own vault file at
  `<epic-key>-<slug>/<story-key>-<slug>/<ticket-key>-<slug>.md` (or directly under the epic if no
  Story tier applies), seeded at creation time (Step 9) — see `log-decision`'s own vault structure.
- Step 6's hand-off to `plan-feature` always includes Step 5's UI-check outcome (fixed
  2026-09-14 — previously omitted, which is how an unresearched reference could silently reach
  Step 9 without ever becoming its own spike ticket; found via a live pipeline run where two build
  tickets adapted a reference with no spike behind them).
- `spec-miner` is always targeted at the one capability Step 3 already identified — never asked
  to present a whole-codebase capability list, since this skill isn't onboarding the whole repo.
- Component-interface mining (Step 4b) uses the identical staleness mechanism as `spec-miner`
  (Step 4a) — a recorded commit SHA compared against `git log -1` on the component's source files.
- The feature-split criteria in Step 6 are diagnostic questions to inform judgment, not a formula
  — a "yes" to one doesn't mechanically force a split or merge.
- No built-in "fix a bad ticket split after filing" mode — a wrong split found later is fixed by
  editing Jira directly.
- Story-as-grouping-tier (Step 9) is judgment, not automatic — use it when an epic is genuinely
  large enough that a flat ticket list would be hard to navigate; a small epic can skip straight
  to `Task`/`Subtask` under the `Epic` with no `Story` tier at all.
