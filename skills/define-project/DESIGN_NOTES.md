# /define-project — Pipeline Design (draft, not yet a working skill)

Status: design drafted (overall pipeline architecture agreed via plan review 2026-09-10); this
skill itself not yet implemented as `SKILL.md`. Open questions below block that — this is the
single highest-leverage draft in the pipeline, since 7 other skills read its output verbatim.

## What this is

The bootstrap skill for the whole pipeline. Run once per project (the same "check existence,
generate once" shape `build-frontend` already uses for `PRODUCT.md`/`DESIGN.md`), it produces
`PROJECT.md`: the engineering-level shared context that `plan-feature`, `assign-tasks`,
`build-backend`, `resolve-pr-comments`, `log-decision`, `pentest-app`, and `audit-seo` all read
before doing anything.

It deliberately does **not** duplicate two things a project may already have:

- **`PRODUCT.md`** — impeccable's output (audience, purpose, voice — product/UX truth). If
  present, `PROJECT.md` links to it (`product_context: ./PRODUCT.md`) instead of re-asking those
  questions. `build-frontend`'s own bootstrap checks for `PRODUCT.md`'s literal existence to
  decide whether to fire `impeccable init` — a file written under that name here would silently
  break that check.
- **A project's own comprehensive context doc** — e.g. `blackboxlabs`'s `AGENTS.md`, which
  already documents stack, coding standards, a "Decision register," a task-routing table, and a
  doc map (including SEO/AEO/GEO doc paths). `define-project` scans for this first and
  links/absorbs from it, asking only about the fields nothing existing already answers:
  ticketing, Obsidian vault, pentest scope/RoE.

## Command

```
/define-project
```

No arguments — it interviews only for what a scan genuinely can't answer.

## Bootstrap sequence

1. Scan the project root for `PRODUCT.md`, `AGENTS.md` (or a `CLAUDE.md` that just imports one),
   `README.md`, `package.json`/`pyproject.toml`, and any `docs/standards*` path.
2. Pre-fill every `PROJECT.md` field a scan can answer confidently — stack from manifest deps,
   coding-standards doc paths and SEO/AEO/GEO doc paths from an existing doc-map table if one is
   found (concretely: `blackboxlabs/AGENTS.md`'s "Documentation map" table already lists
   `docs/standards/{seo,geo-aeo,content-seo}.md` — that should populate `seo_aeo_geo_docs`
   without asking).
3. Interview only for what's left — primarily `ticketing.*`, `obsidian.*`, `pentest.*`, and stack
   fields if nothing was found at all.
4. Write `PROJECT.md` at the project root. Re-running later should update in place, not
   overwrite blind — the same "resume" courtesy `build-frontend`'s `shape` step already gives
   revisions of an existing page.

## Schema (draft)

```yaml
---
project_name: string
product_context: ./PRODUCT.md | null        # impeccable's file, if present — link, don't re-ask
existing_context_doc: ./AGENTS.md | null    # a project's own comprehensive doc, if one exists
stack: { frontend: [...], backend: [...] }
ticketing:
  system: none | jira | github-issues       # "none" is first-class, not a fallback — most
                                             # projects (e.g. blackboxlabs today) have no ticketing system yet
  project_key: null
  auth_env: null                            # env var NAME holding the token, never the token itself
coding_standards: [ ... ]                    # doc paths, pulled from an existing doc map if found
obsidian:
  vault_path: null                          # almost certainly outside the repo; filled in on first real run
  decisions_subpath: decisions/
pentest:
  scope: null                               # a PROPOSAL only — pentest-app must still confirm live, every run
  rules_of_engagement_doc: null
seo_aeo_geo_docs: [ ... ]                    # doc paths
---
```

## Why a separate file, not an extension of `PRODUCT.md` or a project's own `AGENTS.md`

Neither of those schemas belongs to this pipeline. `PRODUCT.md`'s shape is owned by `impeccable`,
a declared external dependency — every upstream change to its `init` template would otherwise
become a breaking change here. A project's own `AGENTS.md` evolves on that project's own
timeline and isn't this repo's to rewrite. `PROJECT.md` stays a small, separate, pipeline-owned
file that only *links* to the others — matching this repo's existing "dependencies are declared,
not vendored" convention.

## Open questions

1. If a scanned context doc already lists SEO/AEO/GEO docs and coding standards in its own
   doc-map table, should `define-project` copy those paths into `PROJECT.md` verbatim, or store
   a pointer to the doc-map table itself and resolve paths lazily? Copying risks drift if the
   source doc's map changes later; a pointer requires every downstream skill to parse an
   arbitrary table format, which won't be consistent project to project.
2. Should `ticketing.system: none` require nothing further, or should it still record a
   task-ID-prefix convention (e.g. `FE-<slug>`/`BE-<slug>`) so `assign-tasks` has something
   concrete to put in commit messages/PR titles even with no external ticket system?
3. Re-running `/define-project` after someone has hand-edited `PROJECT.md` (e.g. filled in
   `obsidian.vault_path` themselves) — does the skill need to preserve edits it didn't generate,
   and how does it tell the difference from a stale scan result?

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 1 — it changes the schema.
2. Write the actual scan heuristics (which manifest fields, which doc filenames/paths count as
   "an existing comprehensive context doc") concretely enough to implement, not just describe.
