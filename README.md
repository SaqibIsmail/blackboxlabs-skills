# /build-frontend — Pipeline Design (draft, not yet a working skill)

Status: design agreed, not implemented. Two TODOs and a few open questions block turning this into an actual SKILL.md — see bottom.

## What this is

A single top-level command that orchestrates two existing skills — **Impeccable**
(process: context, commands, detectors) and **taste-skill** (style: anti-slop
direction, dials) — plus a fixed animation/component toolset, to build or revise
one frontend page/surface per run, using each project repo's own theme file as
the source of visual truth.

## Command

```
/build-frontend "<what to build this run>" [--ref "<description | live-URL + effect | video-URL>"]
```

- **Required:** free-text description of what to build this run (e.g. "a rental
  booking page for a car dealership"). Feeds Impeccable's `shape` step.
- **Optional `--ref`:** a reference for the animation/visual direction — a
  plain description, a live site URL plus which effect on it, or a video link.
  Presence/absence of this decides the mode below.

## Pipeline

### 1. Bootstrap (every invocation)
- Check for `PRODUCT.md` in the repo. Missing → run `/impeccable init`
  (one-time, project-level: users, purpose, positioning, constraints — no
  visual/aesthetic questions asked here).
- Always run `/impeccable document` → reads the repo's own `theme.ts`
  (color theme, typography, allowed animation libraries) and
  generates/refreshes `DESIGN.md` from it. The stack (shadcn / GSAP+
  ScrollTrigger / Lenis / React Bits / Skipper UI, or whatever a given repo
  actually declares) is read from the repo — never hardcoded by this pipeline.

### 2. Direction-sourcing fork
One pipeline, not two — this only decides *where the visual/motion direction
comes from*, not whether animation happens at all.
- **Mode 1 — `--ref` given:** identify the technique from the description,
  live site + named effect, or video. Source the right tool for it (GSAP,
  React Bits, Skipper UI, plain CSS, etc.).
- **Mode 2 — `--ref` omitted:** pull from this skill's **hardcoded curated
  inspiration-site list** (see TODO 1) and pick whichever direction best fits
  this page's purpose and looks flawless.

### 3. Shape (the actual per-run brief)
`/impeccable shape` — discovery interview (purpose, audience, outcome, then
material/states/constraints), resolves the visual direction, and writes a
**surface brief**: job/audience, outcome, selected direction, scope, states,
constraints, and — critically for step 5c — the page's single primary-action
moment. `shape` decides *whether and where* animation belongs; Mode 1/2 only
supplies what it draws from.

### 4. Direction & dials
**taste-skill** sets VARIANCE / MOTION / DENSITY and the concrete design
language, constrained by `DESIGN.md` — it never contradicts what the repo's
own `theme.ts` already fixed.

### 5. Build
- **5a. Base build** — components/tokens per `DESIGN.md`.
- **5b. `/impeccable animate`** — baseline motion (feedback, state changes,
  transitions) using whichever library the repo's `theme.ts` allows.
- **5c. Focus flag** — pull the primary-action moment already named in
  `shape`'s surface brief. This replaces `overdrive`'s own (inconsistent)
  guesswork with a fixed target.
- **5d. `/impeccable overdrive`** — runs on every build now, not situationally.
  Targets the step 5c moment, drafts 2-3 candidate directions in text (with
  trade-offs: perf cost, browser support, complexity), and **asks for a pick
  before writing any code**. Only the chosen direction gets built. One
  overdrive moment per page — never stacked.

### 6. Self-QA (before you see it)
`/impeccable critique` (design review: hierarchy, clarity, emotional
resonance) + `/impeccable audit` (technical scan: a11y, perf, responsive).
Both are diagnostic only — they report findings, they don't fix anything.

### 7. Fix pass (only for what step 6 flagged)
`/impeccable harden` (edge cases, errors, i18n), `/impeccable clarify`
(unclear UX copy), `/impeccable optimize` (only if a real perf bottleneck was
found), `/impeccable adapt` (only if a responsive issue was flagged).

### 8. Final polish
`/impeccable polish` — last alignment/shipping-readiness pass.

### 9. Ping you
**TODO 2** — delivery mechanism not yet designed (chat, notification, `live`
in-browser iteration, etc.).

### 10. Feedback loop
Your feedback is routed to the specific fix command instead of a full redo:
- "too plain" → `bolder`
- "too busy" → `distill` / `quieter`
- "wrong colors" → `colorize`
- "spacing off" → `layout`
- "copy unclear" → `clarify`
- structural change → back to `shape`

Loops until approved.

### 11. Maintenance (not per-run)
- `/impeccable extract` once the repo has a few pages built this way —
  consolidates repeated patterns into the shared design system.
- `/impeccable onboard` only if the page needs a first-run/empty state.

## Fixed toolset (read from each repo, not hardcoded)

- Components: shadcn/ui (or whatever `theme.ts` declares)
- Animation: GSAP + ScrollTrigger (no Framer Motion — avoids taste-skill's
  own "never mix GSAP and Motion in one tree" rule)
- Extras: Lenis, React Bits, Skipper UI where they fit

## Why the commands were included (no conflicts)

- `craft` is skipped — deprecated alias, adds nothing over a plain request.
- `audit`/`critique` diagnose only; `harden`/`clarify`/`optimize`/`adapt` fix
  only what was flagged. This ordering is the one rule that prevents overlap.
- `overdrive` has a hard "propose 2-3 directions, get a pick before code" rule
  and a hard "never stack multiple extraordinary moments" rule — both kept
  as-is, not worked around.
- `init` never touches aesthetics; `shape` never invents new visual worlds
  outside what it plans; `document` never invents tokens — it only reflects
  what the repo's `theme.ts` already says. This is what makes the pipeline
  reusable across different projects without rewriting it per repo.

## TODOs (block turning this into a real SKILL.md)

1. **Curated inspiration-site list** for Mode 2 — will be hardcoded into this
   skill. Not yet supplied by the user.
2. **Ping-you delivery mechanism** (step 9) — how the pipeline actually
   reaches the user to request the overdrive pick and to show the finished
   page. Not yet designed.

## Open questions (unresolved as of this draft)

1. Does `/build-frontend` also handle *revising* an existing page, or is it
   strictly for new pages/runs? Would a revision reuse the same command, or
   need e.g. `/build-frontend --revise`?
2. Should `/impeccable document` re-run in full on every invocation, or only
   when `theme.ts` has changed since the last run?
3. If `--ref` is given but only vaguely (no concrete site/video, just a fuzzy
   description), does it still count as Mode 1, or fall back to Mode 2?
4. Is one `/build-frontend` call always scoped to exactly one page/route, or
   could it target a multi-page flow in a single run?
