# `/research-ux` — Pipeline Design

**Status (2026-09-19): implemented.** See `SKILL.md`/`SETUP.md`.

## What this is

`research-reference`'s sibling for a different kind of unknown. `research-reference` answers "how
is this technically built" (DOM/CSS/JS mechanism, mapped onto this project's stack). Prompted by a
real gap Saqib named directly: a ticket can just as easily need "what should a car-listing card
even contain — a picture, a price, what else" — a content/structure/primary-actions question with
no mechanism to reverse-engineer at all. Neither `research-reference`'s Playwright sweeps nor its
output shape (real mechanism + stack-mapping) fit that question; forcing it through
`research-reference` would mean either skipping most of that skill's own steps or quietly stretching
its scope until it covers two unrelated kinds of research.

Cherry-picks nothing new — this is an original technique (no external source in this pipeline's
prior research covers "content-requirements research via comparable examples" specifically), built
by mirroring `research-reference`'s own shape (read context → check prior research → investigate →
synthesize → write, embed-don't-link, cite evidence) since that shape already proved out well for a
different research flavor, not because a source skill dictated it.

## Where it plugs into the pipeline

- **`senior-engineer`'s Step 5 (UI check)** gains a new, orthogonal case: alongside "a concrete
  visual/mechanism reference exists" (→ `research-reference`) and "no UI idea exists yet" (→
  propose a design spike), there's now "the epic/story doesn't know what content/structure/actions
  a page or component needs" (→ propose a `research-ux` spike). These aren't mutually exclusive —
  a single page can need both a mechanism spike and a content spike, as two separate tickets.
- **Labeling**: a spike ticket now gets one of `research-mechanism` or `research-ux` (alongside the
  existing `spike` type label), so `build` knows which research skill to dispatch without
  re-deriving it from the ticket's prose — same reasoning already applied to the `frontend`/
  `backend`/`shared` scope label (the classifying skill has the signal for free at creation time;
  a downstream dispatcher re-deriving it has strictly less context for no benefit).
- **Blocking**: uses the *already-existing* blocker-link mechanism (`senior-engineer` Step 7/9,
  added the same day) unchanged — a `research-ux` spike blocks whichever build ticket(s) actually
  need its findings, exactly like a `research-mechanism` spike already does. No new blocking
  mechanism needed; this is precisely the generic case that mechanism already covers.
- **`build`'s own consult chain** (still a plan, not yet implemented — see
  `blackboxlabs-skills`' companion design doc) needs one addition to actually make the findings
  reach the blocked ticket: when a ticket has a "blocked by" link, `build` reads that blocker
  ticket's own vault file as part of its context gathering, not just `log-decision`'s
  ancestor-chain walk-up (a blocker is a sibling in the hierarchy, not an ancestor, so the walk-up
  alone would never surface it).

## Explicit boundary with `research-reference`

The two skills never overlap by construction: `research-ux` is barred from reporting on CSS,
animation, or DOM mechanics (Step 4); `research-reference` has no concept of "which fields should
this show" at all. A ticket needing both gets two separate spike tickets, each blocking the same
downstream build ticket if both are genuinely needed before it can start.

## Open questions

None blocking — this skill is implemented. `build`'s own Branch A (currently written narrowly
around `research-reference`) still needs generalizing to dispatch by the `research-mechanism`/
`research-ux` label and apply a verification gate adapted to each (structural completeness +
citation fidelity + concreteness + "could someone build directly from this" for `research-ux`,
mirroring `research-reference`'s own four-part gate) — tracked in `build`'s own design doc, not
here, since `build` doesn't exist as a real skill yet.
