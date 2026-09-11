# /audit-seo — Pipeline Design (draft, not yet a working skill)

**Not final (2026-09-11):** flagged for re-discussion now that the epic layer
(`founder-vision`→`define-epic`→`senior-engineer`) exists upstream — revisit as a whole before
treating it as settled.

Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.

## What this is

**A generalization of a pattern that already works**, not a new design from scratch. The
`blackboxlabs` website project already has a fully working version of this: `.claude/commands/build-page.md`
runs `seo-specialist`, `geo-aeo-specialist`, and `content-seo-specialist` subagents in parallel
(each reading its own standards doc — `docs/standards/{seo,geo-aeo,content-seo}.md` — and
producing a tiered Tier-1/2/3(+Avoid) checklist for one specific page), then hands all three
checklists to a `page-builder` subagent that implements the page and reports which items it
satisfied or skipped and why.

The only thing this skill needs to change relative to that proven pattern: **read doc paths and
stack conventions from `PROJECT.md.seo_aeo_geo_docs` instead of the hardcoded
`docs/standards/*.md` paths**, so the same pattern works unmodified across different projects —
matching this repo's existing "nothing about a specific project's stack is ever hardcoded"
convention (`build-frontend` already does this for theme/animation-library choices).

## Command

```
/audit-seo <page-name> <goal>
```

## Sequence (unchanged from the proven `build-page` pattern, just parameterized)

1. Read `PROJECT.md.seo_aeo_geo_docs` for the list of standards docs to check against (falls
   back to asking, if `define-project` never found any).
2. Run one specialist subagent per doc, in parallel, each producing a tiered checklist for this
   specific page/goal — never proposing anything outside its own doc, flagging gaps as separate
   notes instead of inventing new rules (same discipline the existing `seo-specialist` etc.
   already follow).
3. Hand all checklists + page name/goal to an implementer subagent (the `page-builder`
   equivalent) to build or audit the page against them. Tier 1 items: satisfy without exception.
   Tier 2: satisfy unless it genuinely conflicts with the stated goal, and say so explicitly if
   skipped. Tier 3: optional polish.
4. Show the diff plus a summary of what was satisfied/skipped and why. Leave unstaged for
   review — same "don't commit, this is for review first" convention `page-builder` and
   `build-frontend` both already follow.

## Relationship to `build-frontend`

This is a narrower, checklist-driven sibling to `build-frontend`, not a replacement — it's usable
standalone (auditing an existing page against a project's SEO/AEO/GEO docs) or as one input
`build-frontend` could eventually call into for pages where SEO/AEO/GEO compliance genuinely
matters (a marketing/landing page) but isn't relevant for others (an authenticated internal
dashboard, say).

## Open questions

1. Does `npx skills add` (the CLI this repo's `SETUP.md` files already use for install) support
   installing accompanying `.claude/agents/*.md` subagent files alongside a skill's own
   `SKILL.md`, or only the skill file itself? The specialist-subagent structure is the actual
   valuable part of the pattern being generalized here — if the CLI can't carry agent files, this
   skill needs a different distribution mechanism (e.g. instructing the SKILL.md itself to define
   the sub-checks inline rather than as separate subagent files). This is a mechanical question
   to verify against the real tool, not a design preference.
2. The existing `page-builder` bakes in real project-specific rules beyond just the three
   checklists (e.g. "don't animate the page's likely LCP element," "don't invent a Postgres
   client — ask before deviating from `docs/auth.md`"). Some of these are genuinely SEO-adjacent
   (LCP/animation) and worth generalizing; others are purely this project's own architecture
   decisions and shouldn't be copied into a portable skill at all. Needs a line drawn per-rule,
   not wholesale.
3. Doc count/naming isn't fixed at exactly three (seo/geo-aeo/content-seo) — should this skill
   spin up one specialist per doc in `PROJECT.md.seo_aeo_geo_docs` dynamically (however many
   there are), rather than assuming exactly three named categories?

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 first — it decides whether this pattern can even port as designed.
2. Go through `blackboxlabs`'s three existing specialist agents rule-by-rule per open question 2,
   splitting genuinely-generic guidance from this-project-specific decisions.
