---
name: plan-feature
description: >
  The PM skill for one feature underneath an epic. Interviews only about what senior-engineer's
  handed-off context doesn't already answer, then drives github/spec-kit
  (/speckit.specify -> /speckit.clarify -> /speckit.plan) to produce spec.md and plan.md. Called
  once per feature by senior-engineer (an epic may need several passes); not invoked directly
  from a bare ask. Use when senior-engineer calls this skill, or the user invokes /plan-feature
  directly for a single-feature project with no epic layer.
argument-hint: '"<what to build>"'
user-invocable: true
allowed-tools:
  - Read
  - AskUserQuestion
  - Skill
license: MIT
---

# /plan-feature

Interviews about one feature, then drives `spec-kit`'s mechanized commands to produce
`specs/<feature>/{spec.md,plan.md}`. `spec-kit` supplies the artifact format and command sequence
but has no PM persona or question rubric of its own — this skill supplies that, derived from
BMAD-METHOD's real PRD-workflow techniques (not its heavy runtime — no memlog, no multi-agent
reviewer gate, no `addendum.md`; this is a lightweight, purpose-built interview, not a standalone
PRD workflow).

## Step 1 — Read context

Read `PROJECT.md`, and `PRODUCT.md` if `product_context` is set — audience/purpose/voice already
answered there, don't re-ask. If called by `senior-engineer`, read the handed-off context: the
epic's `define-epic` answers (who/what/why/scope/non-goals/MVP-cut), `senior-engineer`'s own code
investigation findings, any baseline spec from `spec-miner`, and **whether its Step 5 UI-check
flagged an unresearched reference** (a live site, an existing component) this feature adapts.
Treat all of this as already-answered — the interview in Step 4 only covers what it leaves
unanswered.

Also read `PROJECT.md.vision_context` → `VISION.md`'s `mode` field (`startup` /
`intrapreneurial` / `builder`), if `founder-vision` has run, to calibrate how much interview rigor
this feature needs — a `builder`-mode project needs a lighter touch than a `startup`-mode one, the
same way `founder-vision` itself branched on this.

## Step 2 — Check prior decisions

Call `log-decision` (query) for this feature. If entries exist, this is a resume: read them aloud,
don't re-derive.

## Step 3 — Choose interaction style

Ask once: "Fast path — I'll batch what's left into one or two questions and draft with
`[ASSUMPTION]` tags for you to correct, or Coaching path — I'll ask what's left one at a time?"
Default to Coaching path if the user doesn't express a preference; switch to Fast path any time
the user signals impatience, same escape-hatch convention this pipeline already uses elsewhere
(`founder-vision`, `define-epic`).

## Step 4 — Interview

Ask only about what Step 1's context leaves genuinely unanswered. **Elicitation, not
direction** (BMAD's real discipline): pull the answer out of the user, don't propose a scope or
MVP cut and ask them to confirm it. Open-ended questions beat multiple-choice. "I'm assuming X
works like Y — is that right?" is fine (infer-and-confirm); walking the user through a tree of
your own pre-baked options is not.

Cover, whatever isn't already answered:

- **Audience for this specific feature** (if `PRODUCT.md`/the epic context didn't already name
  one specific enough for this feature).
- **The primary user journey**: who does what, in what order, and how do they know it worked —
  one concrete scene, not an abstract capability list.
- **Explicit non-goals for this feature** (narrower than the epic's own non-goals — this feature
  specifically, not the whole epic).
- **Concern scan**: ask what quality/domain concerns this feature actually carries — performance,
  security, compliance, an integration, concurrency — don't run through a fixed checklist; name
  what's actually relevant and ask only about those. This is what keeps `/speckit.clarify` from
  having to guess at it later.
- **Unresearched reference check**: if Step 1's handed-off context flagged a UI/animation reference
  that hasn't actually been inspected yet, confirm it explicitly — it must become a real research
  item in Step 5's Phase 0, not an assumption baked silently into the spec.

In Fast path, batch all of the above into one or two consolidated questions and mark inferred
answers `[ASSUMPTION: ...]` in the resulting brief for the user to correct before proceeding.

## Step 5 — Drive spec-kit

Run `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` to produce
`specs/<feature>/{spec.md,plan.md,tasks.md}`, passing the resolved brief from Step 4 as input.
Because Step 4 already interviewed first, `/speckit.clarify` should have little left to surface —
this is one clean round of questions, not two.

**`tasks.md` is not optional.** `senior-engineer`'s own Step 7 groups its `T0xx` tasks directly
into tickets using BMAD's right-sizing criteria — without it, Step 7 has nothing to walk. If Step 4
flagged an unresearched reference, make sure it surfaces as its own research task in
`/speckit.plan`'s Phase 0 (`research.md`) rather than folded into the build task that depends on
it — a separate research task is what lets Step 7 split it into its own spike ticket, instead of
relying on a rule someone has to remember to apply at ticket-creation time.

## Step 6 — Log the decision

Call `log-decision` (write) once `plan.md`/`tasks.md` are finalized — first entry for this feature:
scope, chosen approach, explicit non-goals. Return value (the written path) is used by
`senior-engineer` for the ticket back-link.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- `/speckit.tasks` always runs as part of Step 5 (fixed 2026-09-14 — an earlier version of this
  skill stopped at `plan.md`, which silently broke `senior-engineer`'s Step 7, already written to
  assume `tasks.md` exists. That was a bug, not a considered design choice).
- Coaching path (ask one at a time) is the default interaction style; Fast path only when the
  user chooses it or signals impatience.
- Interview rigor scales with `VISION.md.mode` when available; defaults to full rigor
  (`startup`-equivalent) if `founder-vision` hasn't run on this project.
- No fixed section template — unlike BMAD's own adaptive PRD sections, this skill's output is
  whatever `/speckit.specify` needs as input, not a standalone document with its own structure.
