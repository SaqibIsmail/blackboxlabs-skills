# /founder-vision — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-11); not yet implemented as
`SKILL.md`. Open questions below block that.

## What this is

Runs once per new project, before `/define-project` — a validation/ideation pass that produces a
lightweight `VISION.md`: target user, non-goals, and the "why now" for building this at all. This
is the founder-mode ideation the pipeline owner asked to keep from `gstack`, so a new project
starts from a validated premise instead of an assumed one.

Cherry-picks `garrytan/gstack`'s `office-hours` skill's question set — rewritten in this repo's
own plain prose, not its runtime (gstack's actual skill is wired into its own telemetry/session
tracking and `gbrain` context-query infra, none of which this pipeline vendors — same
"technique, not runtime" rule already applied to BMAD and superpowers).

## Command

```
/founder-vision
```

No arguments — interviews from scratch. Re-running on a project that already has `VISION.md`
asks for confirmation before overwriting (matching `define-project`'s own "don't silently
overwrite" courtesy).

## Interview (adapted from gstack `office-hours`'s six forcing questions)

Ask until each is answered without hand-waving — don't accept a vague answer and move on:

1. **Demand reality** — who actually has this problem today, concretely (not "everyone")?
2. **Status quo** — what do they do about it right now, without this product?
3. **Desperate specificity** — how painful is this, really — would they pay, switch, or change
   behavior for a fix?
4. **Narrowest wedge** — what's the smallest version of this that's still worth building first?
5. **Observation** — what have you actually seen (not assumed) that makes you think this is real?
6. **Future-fit** — where does this go if it works, so early choices don't paint the project into
   a corner?

## Output

Write `VISION.md` at the project root:

```yaml
---
project_name: string
date: YYYY-MM-DD
---

## Target user / wedge
## Status quo (what they do without this)
## Why now
## Non-goals (explicit)
## North star (if this works)
```

## Handoff

`define-project` scans for `VISION.md` the same way it already scans for `PRODUCT.md`/`AGENTS.md`,
and links it (`vision_context: ./VISION.md | null`) rather than duplicating its content.
`define-epic` reads it for grounding before interviewing about a specific epic.

## Open questions

1. Should this skill refuse to run for a clearly-not-a-startup project (an internal tool, a
   client site) where "who has this problem / would they pay" doesn't really apply — or just let
   low-stakes answers ("just me, solo dev" / "N/A, internal tool") pass through undramatically?
2. Does `VISION.md` ever get revisited/updated (pivots, scope changes), or is it a point-in-time
   artifact only referenced, never edited, after the project's first real epic ships?
3. Confirmed to run per-project, not once company-wide — but for a multi-project owner (same
   person running blackboxlabs, tgc, etc.), does each project really get a from-scratch
   six-question interview with zero pre-fill from a prior project's answers? Worth confirming
   that's genuinely intended, not just simplest-to-describe.

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 — determines whether this skill has a bail-out path.
2. Read gstack's actual `office-hours` phase files (`sections/phase-2a-startup-diagnostic.md`,
   `phase-2b-builder-brainstorm.md`) in full before finalizing exact question wording — this draft
   is based on the top-level `SKILL.md` description only, not those phase files yet.
