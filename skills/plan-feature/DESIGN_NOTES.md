# /plan-feature — Pipeline Design

**Status (2026-09-13): implemented.** See `SKILL.md`/`SETUP.md` — **this completes the front
half of the pipeline** (`founder-vision`→`define-project`→`define-epic`→`senior-engineer`→
`plan-feature`, plus `log-decision` as cross-cutting infrastructure). This file stays as the
rationale record.

**Design enriched while implementing (2026-09-13):** reading BMAD's actual `bmad-prd/SKILL.md` in
full (the TODO below) confirmed it's far too heavy to vendor as designed (memlog audit trail,
multi-agent reviewer gate, `addendum.md` — a standalone PRD workflow, not a per-feature
interview). Extracted three specific, portable techniques instead of the whole system: **(1)
"elicitation, not direction"** — pull the answer out of the user, don't propose scope and ask
them to confirm it; **(2) a concern-scan question** (ask what quality/domain concerns this
specific feature carries, don't run a fixed checklist); **(3) a Fast-path/Coaching-path choice**
for interview pacing, consistent with the escape-hatch pattern already used in `founder-vision`
and `define-epic`. Also wired interview rigor to scale with `VISION.md.mode` (from
`founder-vision`), since BMAD's own "stakes calibration" step is the same idea one level up.

**Updated 2026-09-11:** this skill now runs *underneath* an epic, called once per feature by
`senior-engineer` (an epic may decompose into several features) rather than being invoked
directly from a bare ask.

**Resolved (2026-09-11): epic → feature duplicate-question risk.** Decided "smart hand-off" —
`senior-engineer` passes this skill everything `define-epic` and its own investigation already
learned (see Sequence step 3, below) as pre-filled context, not just the epic name. This skill's
own interview (Sequence step 3) only asks about what that context leaves genuinely unanswered.
This does **not** resolve this file's own open question 1 below, which is a separate, narrower
overlap (this skill's interview vs. spec-kit's own `/speckit.clarify` step) — resolved separately,
see Open Questions below.

**Resolved (2026-09-12), both of this file's own open questions:**
- **Interview ordering**: this skill's interview (step 4) runs *before* spec-kit's own commands
  (step 5) and hands spec-kit an already-resolved brief — `/speckit.clarify` then has little left
  to surface, one clean round of questions instead of two. See step 4/5, below.
- **Question-set source**: a shorter, purpose-built list for this narrower context — but *derived
  by actually reading BMAD's real question categories* (not invented from scratch), matching this
  repo's own habit of reading source skill files directly rather than working from descriptions.
  Also: this pipeline is *not* always missing the "upstream artifact" BMAD's rigor assumes —
  `PRODUCT.md` (from `impeccable`, already linked via `PROJECT.md.product_context`) plays that
  role when present, so step 1 should lean on it rather than treating its absence as the default
  case.

## What this is

The PM skill: interviews about what to build, then drives `github/spec-kit` to produce the
actual planning artifacts (`spec.md`, `plan.md`). Two external things are combined here, doing
different jobs:

- **spec-kit** (declared dependency, `/speckit.specify → /speckit.clarify → /speckit.plan`)
  supplies the mechanized artifact format and command sequence. Confirmed via its own docs and
  templates: spec-kit has **no PM persona or question rubric of its own** — its commands just
  drive whatever agent runs them through a fixed template.
- **BMAD-METHOD's `bmad-agent-pm` / `bmad-prd`** supply the interview rigor and question
  categories to cherry-pick — rewritten in this repo's own plain-prose `SKILL.md` style, not
  ported as BMAD's actual runtime (its real mechanics are a heavy custom system —
  `_bmad/scripts/resolve_customization.py`, `customize.toml` merges, named persona roleplay —
  structurally incompatible with this repo's convention of small, dependency-declaring,
  plain-Markdown skills). BMAD-METHOD is MIT-licensed; only the "BMad"/"BMad Method" names are
  trademark-restricted, so citing it in prose (as `build-frontend` already does for `impeccable`)
  is fine, cherry-picking the technique is fine, vendoring its runtime files is not planned.

## Command and sequence

Superseded by `SKILL.md` Steps 1–6, which incorporate the BMAD-derived interview techniques
above. Kept here only as a historical note.

## Open questions

None — all resolved (see resolution notes near the top of this file).

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.

## Bug found and fixed via live pipeline test (2026-09-14)

Step 5 originally stopped at `/speckit.plan` (`spec.md`/`plan.md` only) — but `senior-engineer`'s
own Step 7 was already written to "walk each feature's `tasks.md`," assuming it exists. These two
files had silently drifted apart: this skill never generated the artifact the other one depended
on. Not caught until an actual live run tried to reconcile spec-kit's output against real Jira
tickets and found `senior-engineer`'s Step 7 had nothing to walk.

Fixed: Step 5 now always runs `/speckit.tasks` too. Also closed a second, related gap — Step 6's
UI-check (an unresearched reference needing a spike) was never explicitly included in the
hand-off `senior-engineer` passes to this skill, so even when a reference was flagged, nothing
carried that flag into `/speckit.plan`'s Phase 0 research, so it never became a task in `tasks.md`
for `senior-engineer`'s Step 7 to split into a spike ticket. Both this skill's Step 1/4/5 and
`senior-engineer`'s Step 6/7 were updated together — this was one gap spanning both files, not two
separate ones.
