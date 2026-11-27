---
name: research-ux
description: >
  Investigates what content, structure, and primary actions a page or component actually needs —
  not how something is technically built (see research-reference for that). Looks at real
  comparable examples in the same domain to establish an evidence-based baseline (which fields and
  actions real car-listing sites consistently include, say), then writes a concrete, citable
  content/structure recommendation grounded in this ticket's own audience and purpose. Called by
  `build` when it picks up a `research-ux`-labeled spike ticket — not a sequential pipeline stage
  of its own, and not invoked directly from a bare ask. Use when a ticket asks "what should X
  page/component contain," or the user names a page/component type in a domain and asks what
  information or actions it needs, as opposed to how it should look or animate.
argument-hint: '"<what needs figuring out: page/component + domain>" "<why it matters / what depends on it>"'
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - WebFetch
  - AskUserQuestion
  - Skill
license: MIT
---

# /research-ux

Investigates one content/structure unknown — thoroughly enough that whoever builds from the
findings knows exactly what fields, actions, and priority order to include, with real evidence
behind each — then writes that up as a concrete recommendation. This is `research-reference`'s
sibling, not its replacement: `research-reference` answers "how is this technically built,"
this skill answers "what does this need to contain." A single epic can need both, as two separate
spike tickets, when a page has both an unresolved visual mechanism and an unresolved content
question.

## Step 1 — Read context

Read `PROJECT.md.stack` (so any recommendation stays buildable with what this project already
has) and the epic's own goal (`scope.md`, per the same `epic_goal_ref` pattern `build` uses) — the
audience and purpose named there ground what "the right content" even means; a car-buying page for
a dealership's own inventory needs different priority fields than a peer-to-peer marketplace one.
If invoked by `build`, read the handed-off ticket description and its own vault file for what
specifically needs figuring out. If invoked directly by the user, take the subject and reason from
the command arguments.

## Step 2 — Check prior research

Call `log-decision` (query) for this exact page/component type. If it was already researched for a
prior ticket, read that entry aloud and reuse it rather than re-investigating from scratch — flag
if the domain has plausibly moved on (a fast-changing category) and let the user decide whether to
re-run.

## Step 3 — Identify comparable examples

Name 2-4 real, credible examples in the same domain — named by the ticket/epic if it already
points at specific competitors or references, or identified via search if not. Prefer examples a
reasonable person would actually recognize as doing this well (established players in the
category), not arbitrary results. If the epic's own `scope.md`/`vision.md` already names
competitors or references, start there before searching further.

## Step 4 — Investigate content and structure, not mechanism

For each comparable example, read the actual page (via `WebFetch` or a browser tool) for:

- **Information fields shown** — what data is present, and in what order/priority (what's above
  the fold vs. below, what's in the primary card vs. a detail view).
- **Primary actions offered** — every CTA/interactive affordance (e.g. "View details," "Contact
  seller," "Compare," "Save").
- **What's consistent across all examples** (a real pattern worth treating as near-mandatory) vs.
  **what varies** (a genuine choice point this ticket still has to make, not something research
  can resolve for it).

This is explicitly not a computed-CSS/DOM/animation sweep — that's `research-reference`'s job. Do
not report on visual styling, motion, or layout mechanics here; report on *what information and
actions exist*, independent of how they're rendered.

## Step 5 — Synthesize a concrete recommendation

Produce a recommendation, not just raw notes:

- **Near-mandatory fields/actions** — present in most/all examples; cite which examples, so the
  claim is checkable, not asserted.
- **Differentiators** — present in some examples, worth a deliberate yes/no call for this project,
  named explicitly as a choice rather than silently included or dropped.
- **The recommended set for this ticket specifically** — filtered through the epic's own
  audience/purpose (Step 1), not just "everyone has it so we should too." Say explicitly when a
  common field doesn't fit this project's actual audience and should be left out.

Concrete and specific, same bar `research-reference` holds itself to: name the actual field/action,
not "relevant details" or "key information" — a build ticket must be able to implement directly
from this list without guessing what it means.

## Step 6 — Write the output

Embed the recommendation directly in the ticket's own description (embed, never just link — this
pipeline's standing convention) and call `log-decision` (write) recording it, so a later ticket
touching the same page/component type doesn't re-investigate from scratch (Step 2). Cite the
comparable examples by name in both places, not just in scratch notes — the citations are part of
what makes the recommendation checkable later.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Content/structure research never touches CSS, animation, or DOM mechanics — that boundary is
  what keeps this skill and `research-reference` from overlapping. A ticket needing both gets two
  separate spikes.
- 2-4 comparable examples is the target range — fewer risks one outlier being mistaken for a
  pattern; more adds research cost without much added confidence for most content questions.
- A recommendation always distinguishes near-mandatory (evidenced across examples) from
  differentiator (a real choice) — never presents the whole list as equally settled.
- Prior research is reused, not blindly trusted forever — flagged for a possible re-check if the
  domain could plausibly have moved on, but not automatically re-run every time.
