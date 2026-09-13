# /founder-vision — Pipeline Design

**Status (2026-09-12): implemented.** See `SKILL.md`/`SETUP.md`. This file stays as the rationale
record.

**Design changed while implementing (2026-09-12):** reading gstack's actual `office-hours` phase
files in full (the TODO below) surfaced that it isn't one fixed 6-question list — it's two modes
(a rigorous adversarial "startup diagnostic" vs. a generative "builder brainstorm" for side
projects), with built-in stage-based routing. Flagged this to Saqib before finalizing, since it
changed the "always ask the same 6" call made before reading the source; **decided to adopt both
modes** — see `SKILL.md` Step 0.

## What this is

Runs once per new project, before `/define-project` — a validation/ideation pass that produces a
lightweight `VISION.md`: target user, non-goals, and the "why now" for building this at all. This
is the founder-mode ideation the pipeline owner asked to keep from `gstack`, so a new project
starts from a validated premise instead of an assumed one.

Cherry-picks `garrytan/gstack`'s `office-hours` skill's question set — rewritten in this repo's
own plain prose, not its runtime (gstack's actual skill is wired into its own telemetry/session
tracking and `gbrain` context-query infra, none of which this pipeline vendors — same
"technique, not runtime" rule already applied to BMAD and superpowers).

## Command, interview, and output

Superseded by the mode split — see `SKILL.md` Steps 0/1/1′/2 for the actual, current command
sequence, question wording, and `VISION.md` schema. Kept only as a historical note that the
original draft assumed a single fixed 6-question list before the source files were read in full.

## Handoff

`define-project` scans for `VISION.md` the same way it already scans for `PRODUCT.md`/`AGENTS.md`,
and links it (`vision_context: ./VISION.md | null`) rather than duplicating its content.
`define-epic` reads it for grounding before interviewing about a specific epic.

## Open questions

1. ~~Should this skill refuse to run for a clearly-not-a-startup project...~~ **Superseded
   (2026-09-12):** the original resolution ("always ask the same 6, no bail-out") assumed a
   single fixed question list. Reading gstack's real source in full showed it's actually two
   modes (startup diagnostic vs. builder brainstorm) — adopted both, see `SKILL.md` Step 0. A
   non-startup project now gets Builder mode, not a bail-out or the wrong diagnostic.
2. ~~Does `VISION.md` ever get revisited/updated...~~ **Resolved (2026-09-12):** yes, it can be
   re-run/updated when the project's direction genuinely changes — see "Revisions," below (new
   section this resolution requires).
3. Confirmed to run per-project, not once company-wide — but for a multi-project owner (same
   person running blackboxlabs, tgc, etc.), does each project really get a from-scratch
   six-question interview with zero pre-fill from a prior project's answers? Worth confirming
   that's genuinely intended, not just simplest-to-describe.

## Revisions (resolves open question 2, above)

`founder-vision` can be re-invoked on a project that already has `VISION.md`. On re-run: don't
overwrite in place — append a new dated section and mark the superseded parts as such (same
non-destructive spirit as `log-decision`'s own "never silently overwrite" rule), so the original
reasoning stays visible even after a pivot. A re-run should also prompt `log-decision` to record
*why* the pivot happened, separately from the vision update itself.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.
