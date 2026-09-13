# /define-project — Pipeline Design

**Status (2026-09-12): implemented.** See `SKILL.md` and `SETUP.md` in this directory — the
design below is kept as the rationale record, not duplicated into the skill file itself. This was
the first skill built for the pipeline's front half, since 7+ other skills read its output.

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
- **`VISION.md`** — `founder-vision`'s output (target user/wedge, non-goals, why-now). If
  present, `PROJECT.md` links to it (`vision_context: ./VISION.md`) instead of re-asking. Added
  2026-09-11: `founder-vision` now runs once per project, before this skill.

## Command

```
/define-project
```

No arguments — it interviews only for what a scan genuinely can't answer.

## Bootstrap sequence

1. Scan the project root for `VISION.md`, `PRODUCT.md`, `AGENTS.md` (or a `CLAUDE.md` that just
   imports one), `README.md`, `package.json`/`pyproject.toml`, and any `docs/standards*` path.
2. Pre-fill every `PROJECT.md` field a scan can answer confidently — stack from manifest deps.
   For coding-standards and SEO/AEO/GEO doc paths: **resolved 2026-09-12 — store a pointer, not a
   copy.** If an existing doc-map table is found (concretely: `blackboxlabs/AGENTS.md`'s
   "Documentation map" table), `PROJECT.md` records where that table lives
   (`doc_map_source: ./AGENTS.md#Documentation map`) and downstream skills resolve actual paths
   from it lazily, at read time — never copied in as a snapshot. Copying risked drift if the
   source doc's own map changed later; every downstream skill already reads `existing_context_doc`
   anyway, so pointing costs nothing extra. Only when no doc-map table exists at all does
   `define-project` ask directly and store literal paths in `coding_standards`/`seo_aeo_geo_docs`.
3. Interview for what's left:
   - `ticketing.*` — **resolved 2026-09-12: this is an active interview step, not a passive
     field.** Saqib is connecting Jira, so treat Jira as the expected default: ask directly for
     the Jira site URL, project key, and which env var holds the API token (never the token
     itself) — the exact fields ECC's `jira-integration` skill needs for its MCP setup. Only fall
     back to asking "what ticketing system, if any" when the project genuinely isn't using Jira.
   - `obsidian.*`, `pentest.*`, and stack fields if nothing was found at all.
4. Write `PROJECT.md` at the project root. Re-running later should update in place, not
   overwrite blind — the same "resume" courtesy `build-frontend`'s `shape` step already gives
   revisions of an existing page.

## Schema (draft)

```yaml
---
project_name: string
vision_context: ./VISION.md | null          # founder-vision's file, if present — link, don't re-ask
product_context: ./PRODUCT.md | null        # impeccable's file, if present — link, don't re-ask
existing_context_doc: ./AGENTS.md | null    # a project's own comprehensive doc, if one exists
doc_map_source: ./AGENTS.md#Documentation map | null   # pointer, not a copy — resolved lazily by
                                                        # downstream skills. Only unset when no
                                                        # doc-map table was found at all.
stack: { frontend: [...], backend: [...] }
ticketing:
  system: jira | github-issues | none       # Jira is the expected default (Saqib is connecting
                                             # it) — define-project actively interviews for the
                                             # fields below when this is jira, not just recording
                                             # a flag. "none" stays supported for other projects.
  project_key: null
  jira_site_url: null
  auth_env: null                            # env var NAME holding the token, never the token itself
coding_standards: [ ... ]                    # only literal paths when no doc_map_source exists
obsidian:
  vault_path: null                          # almost certainly outside the repo; filled in on first real run
  decisions_subpath: decisions/
pentest:
  scope: null                               # a PROPOSAL only — pentest-app must still confirm live, every run
  rules_of_engagement_doc: null
seo_aeo_geo_docs: [ ... ]                    # only literal paths when no doc_map_source exists
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

1. ~~If a scanned context doc already lists SEO/AEO/GEO docs...~~ **Resolved (2026-09-12):**
   pointer, not copy — see `doc_map_source` in the schema above.
2. ~~Should `ticketing.system: none` require nothing further...~~ **Resolved (2026-09-12):** Jira
   is the expected default — this became a real interview step (site URL, project key, auth env
   var), not just a recorded flag. See "Bootstrap sequence" step 3 and the schema above. The
   original `none`-case ID-prefix question is deprioritized, not answered — revisit only if this
   pipeline is ever run on a project that genuinely has no ticketing system.
3. ~~Re-running `/define-project` after someone has hand-edited `PROJECT.md`...~~ **Resolved
   (2026-09-12):** re-run asks which mode — "refresh scanned fields only" (default, leaves Step 3
   answers/hand-edits untouched) or "full re-interview" (rare, explicit reset). See `SKILL.md`
   Step 4. Also resolved along the way: Jira needs three fields (`jira_site_url`, `jira_email`,
   `auth_env`), not one — confirmed by reading ECC's `jira-integration` skill in full.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.
