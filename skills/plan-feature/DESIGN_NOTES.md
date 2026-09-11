# /plan-feature — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.

**Not final (2026-09-11):** flagged for re-discussion now that the epic layer
(`founder-vision`→`define-epic`→`senior-engineer`) exists upstream of this skill. The
epic-to-feature handoff below is settled, but treat the rest of this file as provisional until
revisited as a whole against the new pipeline shape.

**Updated 2026-09-11:** this skill now runs *underneath* an epic, called once per feature by
`senior-engineer` (an epic may decompose into several features) rather than being invoked
directly from a bare ask.

**Resolved (2026-09-11): epic → feature duplicate-question risk.** Decided "smart hand-off" —
`senior-engineer` passes this skill everything `define-epic` and its own investigation already
learned (see Sequence step 3, below) as pre-filled context, not just the epic name. This skill's
own interview (Sequence step 3) only asks about what that context leaves genuinely unanswered.
This does **not** resolve this file's own open question 1 below, which is a separate, narrower
overlap (this skill's interview vs. spec-kit's own `/speckit.clarify` step) — still open.

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

## Command

```
/plan-feature "<what to build>"
```

## Sequence

1. Read `PROJECT.md` (from `define-project`) and `PRODUCT.md` if present, for context — stack,
   coding standards, existing decisions.
2. Check `log-decision` for any prior entry on this feature (a resume, not a fresh start, if one
   exists — same "resume, don't restart" courtesy `build-frontend`'s `shape` step gives page
   revisions).
3. **Read the handed-off context** — when called by `senior-engineer`, this includes: the epic's
   `define-epic` answers (who/what/why/scope/non-goals/MVP-cut), anything `senior-engineer`'s own
   code investigation already found, and a mined baseline spec if `spec-miner` ran. Treat this as
   already-answered, not a prompt to re-verify by re-asking.
4. Interview: don't assume — ask only about what step 3's context left genuinely unanswered:
   scope/audience/non-goals specific to *this* feature (not already covered at the epic level),
   and anything spec-kit's own `/speckit.clarify` step would otherwise have to guess at later.
   The hard requirement from the original ask ("asks questions on what I want to build") still
   holds — this skill refuses to silently assume major scope — it just doesn't re-ask what's
   already known.
5. Drive `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` to produce
   `specs/<feature>/{spec.md,plan.md}`.
6. Call `log-decision` (write) once `plan.md` is finalized — first entry for this feature: scope,
   chosen approach, explicit non-goals.

## Open questions

1. Where does the *interview* actually happen relative to spec-kit's own `/speckit.clarify`
   step — does `plan-feature` do its own interview first and then hand a mostly-resolved brief
   into spec-kit's commands, or does it let `/speckit.clarify` surface ambiguities and only add
   BMAD-style rigor on top of whatever spec-kit's own clarify step already asks? Doing both risks
   asking the user near-duplicate questions twice.
2. How much of BMAD's actual question set is worth reproducing verbatim vs. genuinely
   reinventing for this narrower single-command context — BMAD's PM persona runs as one part of
   a much longer multi-step engagement (analyst → PM → architect), and some of its rigor may
   assume upstream artifacts (a product brief) this pipeline doesn't always have.

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 — it fixes the actual step order.
2. Draft the concrete question list/categories this skill asks, reviewed against a couple of
   real feature examples before finalizing.
