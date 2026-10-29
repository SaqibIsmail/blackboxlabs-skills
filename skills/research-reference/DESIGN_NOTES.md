# /research-reference — Pipeline Design

**Status (2026-09-17): implemented.** See `SKILL.md`/`SETUP.md`. This file stays as the rationale
record.

## What this is

Fills a gap `senior-engineer` already names but never resolves: Step 5's UI check decides a spike
is needed for any reference-backed idea, but nothing in this pipeline actually *did* the research
— the resulting tickets (`SCRUM-29`, `SCRUM-30`, etc.) were instructions ("research how X works
before adapting it"), not findings. Saqib decided the actual research belongs at build time, not
planning time, and should be a dedicated skill both `build-frontend` and `build-backend` call
into — not folded into either builder directly (see both files' own reasoning, and
`log-decision/DESIGN_NOTES.md`'s "Call sites" table for where this fits).

## Cherry-pick source

`JCodesMore/ai-website-cloner-template`'s `clone-website` skill — **34.5k★, MIT, verified before
adopting anything** (not from its description; the actual `SKILL.md` was read in full). It solves
a much bigger problem than we need (clone and rebuild an entire live site, dispatching parallel
builder agents across git worktrees) — that whole runtime is explicitly **not** being vendored,
same rule as every other cherry-pick in this repo. What's taken is the investigation discipline
only, narrowed from "the whole page" to "one named reference":

- **"Don't click first."** Scroll through slowly and observe before touching anything. Getting
  the interaction model wrong (click-based UI built for what was actually scroll-driven) is a
  rewrite, not a fix — so determining it correctly comes before any other investigation.
- **Four ordered sweeps** (scroll, click, hover, responsive) as the concrete mechanism behind
  "look for everything, not just the obvious effect" — a scroll-linked header change is invisible
  in a screenshot and easy to miss without a systematic pass.
- **Extract appearance and behavior separately, always both.** Real `getComputedStyle()` values,
  never an eyeballed guess; for anything dynamic, the exact trigger, the before/after values, and
  the transition (library, duration, easing) — not just "it animates."
- **The write-up is the contract, embedded inline, not just linked** — this pipeline already had
  this principle (`senior-engineer` Step 5's "embed findings directly in the build ticket's
  description, not just a link"); finding the same rule independently arrived at in a 34k-star
  project is a good sign it's the right call, not a coincidence to ignore.

Everything cut from the source: parallel builder-agent dispatch, git worktree isolation, the
whole-page scope, its own asset-download/route-generation mechanics. None of that is this skill's
job — `build-frontend`/`build-backend` already own the actual build.

## Two capability requirements Saqib set explicitly (2026-09-17)

- **Playwright**, directly — not a preference, the same declared dependency `senior-engineer`'s
  own research-spike step already named.
- **Convert the reference into this project's own stack cleanly, with minimal visual
  difference** — the deliverable is a translation, not a passive research report. This is why
  Step 5 (write up the real mechanism) and Step 6 (map onto `PROJECT.md.stack`) are two distinct
  steps, not one — the raw mechanism is recorded before any stack-specific translation happens,
  so the translation is traceable back to what was actually found.

## Relationship to other skills

- **Called by `build-frontend`/`build-backend`**, not a sequential stage of its own — mirrors how
  `plan-feature` is called by `senior-engineer` rather than being inlined into it.
- **`log-decision`**: queries before investigating (avoid re-researching the same reference),
  writes after (the findings, and separately, any new-library decision Step 6 surfaced).
- **A different, complementary find, not used here**: ECC's `motion-patterns` skill (a cookbook
  of correct Framer Motion patterns for React/Next.js) is not for investigating a reference — it's
  for `build-frontend`'s own code quality once it's actually writing an animation, regardless of
  whether that animation came from a reference or not. Worth a separate look when `build-frontend`
  itself gets revisited, not part of this skill.

## Open questions

None blocking — this skill is implemented. One thing worth confirming empirically once
`build-frontend`/`build-backend` actually call into it: whether the four-sweep investigation is
fast enough in practice for a simple, single-effect reference, or whether a lighter path is worth
adding for the common case. Left as a judgment call for then, not decided speculatively now.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.
