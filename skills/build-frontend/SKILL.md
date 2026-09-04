---
name: build-frontend
description: >
  Orchestrates a full frontend build or revision for one page/surface, combining the
  "impeccable" skill (process, commands, detectors) and the "taste-skill" / "design-taste-frontend"
  skill (anti-slop direction, VARIANCE/MOTION/DENSITY dials) with a fixed animation/component
  toolset read from the project's own theme file. Use when the user invokes /build-frontend,
  or asks to build, add, or revise a page/feature/section with polished design and motion.
  Requires the "impeccable" and "design-taste-frontend" skills to already be installed in
  this repo or harness.
argument-hint: '"<what to build this run>" [--ref "<description | live-URL + effect | video-URL>"]'
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Skill
  - Task
  - AskUserQuestion
  - WebFetch
license: MIT
---

# /build-frontend

Builds or revises one frontend page/surface per run by orchestrating the `impeccable` skill
(process) and the `design-taste-frontend` skill (style/anti-slop), against whatever component
and animation stack the current repo has already declared in its own theme file. Nothing about
the visual stack is hardcoded here — it is read fresh from the repo every time so this skill
works unmodified across different projects.

## Dependency check (run first, every invocation)

Before anything else, confirm both dependency skills are available in this harness/repo:
`impeccable` and `design-taste-frontend` (taste-skill). If either is missing, stop and tell the
user which one, with its install command:

- impeccable: `npx impeccable install` or `/plugin marketplace add pbakaus/impeccable`
- taste-skill: `npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"`

Do not proceed without both.

## Parse input

- **Required:** free-text description of what to build/change this run.
- **Optional `--ref`:** any of a plain description, a live site URL plus which effect on it, or
  a video link. Presence of `--ref` (however vague) selects Mode 1 below; total absence selects
  Mode 2. Do not require `--ref` to be concrete to count as Mode 1 — a fuzzy description is still
  a user-supplied reference and should not fall back to Mode 2.

## Step 1 — Bootstrap

1. Check for `PRODUCT.md` at the project root. If missing, invoke the Skill tool with
   `skill: "impeccable"`, `args: "init"`. This is one-time, project-level, and durable — it does
   not get re-run just because `/build-frontend` runs again later.
2. Locate the project's theme source (commonly `theme.ts`, `theme.config.ts`, or equivalent —
   search for the file that defines color tokens, typography, and any declared animation
   libraries). Compute a hash of its contents.
3. Compare against the hash stored in `.build-frontend/theme.hash` (create the directory if
   absent). If the hash differs or no `DESIGN.md` exists yet, invoke the Skill tool with
   `skill: "impeccable"`, `args: "document"` to regenerate `DESIGN.md` from the theme file, then
   write the new hash. If the hash matches, skip regeneration and load the existing `DESIGN.md`
   as-is — do not re-run `document` on every invocation.
4. If the theme file does not declare an animation library, ask the user once which to use
   (default suggestion: GSAP + ScrollTrigger), record the answer in `DESIGN.md`, and treat it as
   fixed for this project going forward.

## Step 2 — Direction-sourcing fork

This is not two separate pipelines — it only decides where the visual/motion direction comes
from. Whether animation happens at all, and where, is still decided in Step 3 (`shape`).

- **Mode 1 (`--ref` given):** identify the referenced technique — inspect the description, the
  live site's named effect, or the video — and determine the correct implementation tool for it
  (GSAP for pin/scrub effects, React Bits or Skipper UI for pre-built animated components, plain
  CSS for anything simple).
- **Mode 2 (`--ref` omitted):** pick a direction from the curated inspiration list below, choosing
  whichever fits this page's stated purpose best — not just whichever looks most impressive.

### Curated inspiration list (starter set — placeholder, to be replaced/expanded by the user)

- Awwwards — awwwards.com
- Godly — godly.website
- Land-book — land-book.com
- Lapa Ninja — lapa.ninja
- SiteInspire — siteinspire.com
- Mobbin — mobbin.com (for app/mobile-flow references)

## Step 3 — Shape (the per-run brief)

Invoke the Skill tool with `skill: "impeccable"`, `args: "shape <description>"`, passing the
parsed description and the Step 2 output as context. This produces the surface brief: job,
audience, outcome, selected direction, scope, states, constraints, and — required for Step 5c —
the page's single primary-action moment.

If a surface brief already exists for this route (i.e. this is a revision of a page built
before), `shape` resumes and updates it rather than starting fresh. This is also how revision
requests are handled — there is no separate `--revise` flag; re-running `/build-frontend` against
an existing route is itself the revision path.

## Step 4 — Direction & dials

Invoke the Skill tool with `skill: "design-taste-frontend"`, passing the surface brief as
context. It sets VARIANCE / MOTION / DENSITY and the concrete design language. Its output must
never contradict `DESIGN.md` — the repo's own tokens always win over taste-skill's defaults.

## Step 5 — Build

1. **Base build** — implement structure and components per the surface brief and `DESIGN.md`.
2. **Motion baseline** — invoke the Skill tool with `skill: "impeccable"`, `args: "animate"`.
   Implements feedback, state-change, and transition motion using whichever library the repo's
   theme file declared.
3. **Focus flag** — read the primary-action moment already named in the Step 3 surface brief.
   This is the fixed target for overdrive; do not let overdrive re-infer its own target.
4. **Overdrive** — invoke the Skill tool with `skill: "impeccable"`, `args: "overdrive"`, passing
   the Step 5c target explicitly. This runs on every build. It will draft 2-3 candidate
   directions in text (with trade-offs: performance cost, browser support, complexity) and use
   AskUserQuestion to get a pick before writing any code. Build only the chosen direction. Never
   apply more than one overdrive moment to the same page.

## Step 6 — Self-QA (before showing the user)

Invoke `impeccable` `critique` (design review) and `impeccable` `audit` (a11y/perf/responsive
scan). Both are diagnostic only — collect findings, fix nothing yet.

## Step 7 — Fix pass

For each finding from Step 6, invoke only the matching command: `impeccable harden` (edge cases,
errors, i18n), `impeccable clarify` (unclear copy), `impeccable optimize` (only if a real
performance bottleneck was found), `impeccable adapt` (only if a responsive issue was flagged).
Do not run a fix command with nothing to fix.

## Step 8 — Polish

Invoke `impeccable` `polish` — the final alignment/shipping-readiness pass.

## Step 9 — Ping the user

Default mechanism (until the user configures something else): present the result directly in
the chat response — a summary of what was built plus a screenshot/preview if a browser tool is
available — and ask via AskUserQuestion whether to approve or request changes. Do not assume any
external notification channel exists.

## Step 10 — Feedback loop

Route feedback to the specific command instead of a full rebuild:

| Feedback sounds like | Command |
|---|---|
| "too plain / boring" | `impeccable bolder` |
| "too busy / loud" | `impeccable distill` or `impeccable quieter` |
| "wrong colors" | `impeccable colorize` |
| "spacing/layout is off" | `impeccable layout` |
| "copy is unclear" | `impeccable clarify` |
| structural/bigger change | back to Step 3 (`shape`) |

Loop until the user approves. Each loop should re-run only Step 6 onward (or Step 3 for
structural changes) — never restart from Step 1.

## Step 11 — Maintenance (not every run)

- Once the project has a few pages built via this skill, suggest (don't force) running
  `impeccable extract` to consolidate repeated components/tokens into the shared design system.
- Run `impeccable onboard` only if the page in question has a first-run or empty state to design.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- `--ref` of any specificity, however vague, counts as Mode 1. Only its total absence is Mode 2.
- `impeccable document` only re-runs when the theme file's hash has changed, not on every call.
- One `/build-frontend` invocation is scoped to exactly one page/route. A multi-page flow means
  multiple invocations, one per page.
- Revisions reuse the same command against the same route; there is no separate revise mode.
