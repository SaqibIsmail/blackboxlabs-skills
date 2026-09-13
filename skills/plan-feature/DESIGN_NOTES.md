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
   `specs/<feature>/{spec.md,plan.md}`. Because step 4 already interviewed first, this brief is
   already resolved going in — `/speckit.clarify` should have little left to surface.
6. Call `log-decision` (write) once `plan.md` is finalized — first entry for this feature: scope,
   chosen approach, explicit non-goals.

## Open questions

Both resolved 2026-09-12 — see the resolution note near the top of this file. No open questions
remain for this skill.

## TODOs (block turning this into a real `SKILL.md`)

1. Read BMAD's `bmad-agent-pm`/`bmad-prd` skill files in full (not just descriptions) and extract
   the actual question categories worth keeping — this is the concrete next step now that "derive
   from BMAD's real files, don't invent" is the confirmed approach.
2. Draft the concrete question list/categories this skill asks, reviewed against a couple of
   real feature examples before finalizing — including how `PRODUCT.md` (when present) changes
   which questions are still needed.
