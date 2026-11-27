---
name: research-reference
description: >
  Investigates one concrete UI/animation/component reference (a live site or an existing
  component) in full technical detail, then maps the real mechanism onto this project's own
  tech stack. Called by build when they pick up a spike-labeled research
  ticket — not a sequential pipeline stage of its own, and not invoked directly from a bare ask.
  Use when a ticket says "research how X works before adapting it," or the user names a live site
  or existing component and asks how it actually works.
argument-hint: '"<reference: URL or existing-component name/path>" "<what it needs to become>"'
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

# /research-reference

Investigates exactly one reference — thoroughly enough that whoever builds from the findings
never has to guess a value or re-derive the mechanism themselves — then translates it onto this
project's own stack. Cherry-picks its investigation discipline from `JCodesMore/
ai-website-cloner-template`'s `clone-website` skill (34.5k★, MIT), narrowed to one reference
instead of a whole-page clone, and dropped its parallel-worktree-builder runtime entirely (not
this skill's job — `build` already own that).

**Not for content/structure questions** — "what fields/actions should this contain" (as opposed to
"how does this render/animate") is `research-ux`'s job, a sibling skill for a different kind of
unknown. A `spike` ticket carries a `research-mechanism` or `research-ux` label telling the caller
which one applies.

## Step 1 — Read context

Read `PROJECT.md.stack` (frontend/backend frameworks and animation libraries already in use —
never introduce a new one silently, see Step 5). If invoked by `build`,
read the handed-off ticket reference and its own `log-decision` vault file for what specifically
needs researching and any prior context. If invoked directly by the user, take the reference and
goal from the command arguments.

## Step 2 — Check prior research

Call `log-decision` (query) for this reference (by URL or component name). If it was already
researched for a prior ticket, read that entry aloud and reuse it rather than re-investigating
from scratch — flag if the reference's live version may have changed since then and let the user
decide whether to re-run.

## Step 3 — Identify the reference kind

- **A live site or hosted component** (a URL) → Step 4 (browser-based investigation).
- **An existing component already in this project, or a copied/vendored reference component**
  (a name or file path) → skip Step 4's browser work; read its real source directly (never guess
  from how it's currently used elsewhere) and go straight to Step 5's write-up.

## Step 4 — Browser-based investigation (live references only)

Requires a browser-automation tool capable of real interaction and computed-style inspection
(Playwright is the declared dependency for this pipeline — see `SETUP.md`; any equivalent
MCP-exposed browser tool the harness already provides is acceptable).

**Don't click first.** Scroll through slowly and observe before touching anything — a section
that changes on scroll and a section that changes on click are fundamentally different builds,
and guessing wrong here means a rewrite, not a tweak. Only after the scroll pass is done, test
click and hover.

Run all four sweeps, in this order, and record every finding — completeness beats speed here;
if a builder would have to guess a color, timing value, or trigger, the investigation isn't done:

1. **Scroll sweep** — scroll top to bottom slowly. Does anything change purely from scrolling
   (header behavior, scroll-linked reveals, scroll-snap, parallax, a smooth-scroll library like
   Lenis/Locomotive)? Record the exact trigger (scroll position or intersection threshold).
2. **Click sweep** — test every interactive element. For anything with multiple states (tabs,
   accordions, pagination), click through **every** state, not just the default — record each
   state's content and the transition between them.
3. **Hover sweep** — test every element that plausibly has a hover state; record what changes
   and the transition timing.
4. **Responsive sweep** — check desktop (1440px), tablet (768px), and mobile (390px); note which
   sections change layout and roughly where the breakpoint falls.

For every discovered behavior, extract both halves, never just one:
- **Appearance**: real computed CSS (`getComputedStyle()`), not an eyeballed approximation.
- **Behavior**: the exact trigger, the before/after computed values, and the transition (library
  used if identifiable — GSAP/Framer Motion/CSS `animation`/`transition`/Webflow IX2/etc.,
  duration, easing).

## Step 5 — Write up the real mechanism

Before mapping anything onto this project's stack, write the mechanism as discovered — DOM
structure, the actual library/technique identified, exact keyframes/timing/easing, the precise
trigger. This is the "how it actually works" record, independent of what we'll build it with.

## Step 6 — Map onto this project's own stack

Using `PROJECT.md.stack`: if the discovered mechanism already matches a library this project
already uses (e.g. reference uses scroll-linked opacity and this project already has Framer
Motion's `useScroll`/`useTransform` in play), map onto that directly. If faithfully reproducing
the reference would require a library this project doesn't already have, **don't add it
silently** — flag it explicitly as a decision (matches the project's own "don't choose a new
library in a decided space unilaterally — ask" principle) rather than introducing a new
dependency on your own judgment.

## Step 7 — Write the output

Produce one write-up covering both Step 5 (the real mechanism) and Step 6 (the stack-mapped
implementation guidance) — never just one. Lead it with a clearly labeled
**`Reference source(s):`** line listing the exact URL(s)/identifier(s) actually investigated (e.g.
`Reference source(s): https://v7labs.com, https://demo-powerai.sitesplaced.com`) — separate from
the prose mechanism description, not buried inside a sentence. This is a mechanical requirement,
not a style preference: `build`'s implementer and reviewer subagents extract this line to browse
the real reference themselves later (see `build/SKILL.md`'s B4a/B4b) — a URL paraphrased into prose
is not reliably extractable the same way. Embed the whole write-up directly wherever the caller
needs it (the ticket's own description, per this pipeline's existing "embed, don't just link"
convention) — don't leave the calling builder to go find it. Call `log-decision` (write) recording
the reference and findings, so a later ticket touching the same reference doesn't re-investigate
from scratch (Step 2). If Step 6 flagged a new-library decision, that's what gets recorded here
specifically, for the user to resolve before the build ticket proceeds.

**End the write-up with a `Build-fidelity checklist`** — a literal itemized list, not a
restatement of the prose above it. This is the single most important output of this skill, because
it's the only thing `build`'s reviewer (see `build/SKILL.md` B4b) checks the finished component
against line-by-line. Live-testing this skill against a real ticket showed why prose alone fails:
a paragraph description of a reference's hover-expanding mega-menu got read, understood, and then
quietly dropped at build time anyway — nothing forced anyone to confirm it against a checklist
before calling the ticket done. Two kinds of line belong on it:
- **Every measured value** from Step 4's sweeps: exact px/rem for spacing, padding, gaps, sizes —
  not just "generous spacing," the actual number, and specifically including edge/container
  padding (the gap from the viewport edge to the first element), which is easy to measure
  everything *except*.
- **Every distinct interactive mechanism** found in the click/hover/scroll sweeps, named as its own
  line — e.g. "Header expands from 96px to 448px on hover, revealing a panel" — not folded into the
  general mechanism prose where it can be skimmed past or silently scoped out.

Each line should be independently checkable by someone who never read the prose above it.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Always run all four sweeps for a live reference, even if the ticket only asks about one
  specific effect — a behavior invisible in a static screenshot (a scroll-linked header change,
  say) is exactly what gets missed otherwise, and re-running this skill a second time to catch it
  is more expensive than one thorough pass.
- Never introduce a new animation/UI library on this skill's own authority — a stack gap is
  always surfaced as a decision, never silently resolved.
- Prior research is reused, not blindly trusted forever — flagged for a possible re-check if the
  live reference could plausibly have changed, but not automatically re-run every time.
