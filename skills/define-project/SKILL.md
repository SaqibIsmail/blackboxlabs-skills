---
name: define-project
description: >
  Bootstraps a project for the rest of this pipeline (founder-vision, define-epic,
  senior-engineer, plan-feature, and beyond) by scanning the repo for context it can already
  answer from (PRODUCT.md, an existing AGENTS.md/CLAUDE.md, VISION.md, package manifests) and
  interviewing only for what's genuinely missing (Jira connection, Obsidian vault, pentest
  scope). Writes PROJECT.md, the shared context file every other pipeline skill reads before
  doing anything. Use when the user invokes /define-project, or starts a brand-new project
  with this pipeline.
argument-hint: (none — run with no arguments)
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
license: MIT
---

# /define-project

Produces `PROJECT.md` — the small, pipeline-owned context file every other skill in this
repo reads before doing anything (stack, ticketing connection, Obsidian vault, pentest scope,
doc paths). Run once per project. Nothing here is invented from scratch when the project
already has an answer: it scans first, and only interviews for genuine gaps.

## Step 1 — Scan

Look for, at the project root (don't go deeper than one level for these):

1. `VISION.md` — `founder-vision`'s output, if that skill has already run.
2. `PRODUCT.md` — `impeccable`'s output (audience/purpose/voice).
3. An **existing comprehensive context doc**: `AGENTS.md` at the root, or — if `AGENTS.md`
   doesn't exist — read `CLAUDE.md`. If `CLAUDE.md` is short (under ~20 lines) and contains an
   `@`-prefixed import line (e.g. `@/path/to/AGENTS.md`, the same convention this very pipeline's
   own `CLAUDE.md` files use), treat the imported file as the existing context doc instead of
   `CLAUDE.md` itself.
4. `README.md`, and a package manifest — `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`,
   `requirements.txt` (whichever exists; a project can have more than one, e.g. a JS frontend +
   Python backend monorepo).
5. Any `docs/standards*` path (SEO/AEO/GEO or coding-standards docs living outside a doc-map
   table).

## Step 2 — Pre-fill from the scan

- **`vision_context`** / **`product_context`**: set to the file's relative path if found, else
  `null`. Never interview for these — if missing, that's `founder-vision`'s or `impeccable`'s job,
  not this skill's.
- **`existing_context_doc`**: the file identified in Step 1.3, or `null`.
- **`doc_map_source`** (pointer, not copy): if `existing_context_doc` is set, search it for a
  heading that looks like a documentation map (`## Documentation map`, `## Doc map`, or a table
  whose header row contains "doc"/"path"/"location"). If found, record
  `<existing_context_doc>#<heading text>` — do **not** copy the paths themselves out of that
  table. If no such heading/table exists, leave `doc_map_source: null` and instead ask directly
  for `coding_standards` and `seo_aeo_geo_docs` paths in Step 3.
- **`stack`**: classify each manifest's declared dependencies into `frontend` / `backend` by
  common framework name (React, Vue, Svelte, Next.js's client side → frontend; Express, Fastify,
  Django, Rails, Spring, Next.js's API routes/server actions → backend). A framework that's
  genuinely both (Next.js, Nuxt, SvelteKit) goes in both lists — don't force a single bucket.

## Step 3 — Interview for what's left

### Ticketing (Jira)

Jira is the expected system for this pipeline (not a hypothetical "if any"). Ask for, in order:

1. **Jira site URL** (e.g. `https://yourorg.atlassian.net`) → `ticketing.jira_site_url`.
2. **Atlassian account email** → `ticketing.jira_email`. Not a secret — store it directly.
3. **Project key** (e.g. `PROJ`) → `ticketing.project_key`.
4. **Name of the environment variable holding the Jira API token** (never the token value itself)
   → `ticketing.auth_env`. After naming it, check with `Bash` whether that env var is actually set
   in the current shell (`printenv <name>` non-empty). If it isn't, tell the user plainly — "that
   env var isn't set yet; `senior-engineer`/`jira-integration` will fail until it is" — but do
   **not** block finishing this skill on it. Recording the intended name now is still useful.
5. **If the token is confirmed set and reachable**, query the project's available issue types
   (`GET /rest/api/3/project/<project_key>`) and record them at
   `ticketing.issue_types_available`. A default Jira template commonly lacks native `Bug`/`Spike`
   types — `senior-engineer` needs to know this before it tries to create one. Skip silently if
   the token isn't set yet; this isn't worth blocking on either.

If the user says this project genuinely won't use Jira (a one-off exception, not the default),
set `ticketing.system: none` and skip the four fields above — `assign-tasks`/`senior-engineer`'s
ticketing-agnostic fallback then applies, though that path is not the maintained default.

### Obsidian

Ask for `obsidian.vault_path` (an absolute path, almost certainly outside this repo) and
`obsidian.decisions_subpath` (default suggestion: `decisions/` — accept the default unless the
user wants something else).

### Pentest

Ask for a proposed `pentest.scope` and `pentest.rules_of_engagement_doc` path, if one exists.
Make explicit that this is only a **default proposal** — `pentest-app` re-confirms scope live,
every single run, no exceptions.

### Stack (only if Step 1/2 found nothing at all)

Ask directly what the frontend/backend stack is.

## Step 4 — Write `PROJECT.md`

**First run:** write the file fresh at the project root, using the schema below.

**Re-run** (file already exists) — resolves the "does it preserve hand-edits" question: ask via
`AskUserQuestion` which mode to run in —

- **Refresh scanned fields only** (default/recommended): re-run Steps 1–2 and overwrite only
  `vision_context`, `product_context`, `existing_context_doc`, `doc_map_source`, and `stack`.
  Every field from Step 3 (ticketing, obsidian, pentest, and any manually-added
  `coding_standards`/`seo_aeo_geo_docs`) is left untouched, including hand-edits made directly in
  the file.
- **Full re-interview**: start over from Step 1 as if this were the first run. Use this only for
  a genuine reset (e.g. the project's ticketing system changed).

## Schema

```yaml
---
project_name: string
vision_context: ./VISION.md | null
product_context: ./PRODUCT.md | null
existing_context_doc: ./AGENTS.md | null
doc_map_source: ./AGENTS.md#Documentation map | null
stack: { frontend: [...], backend: [...] }
ticketing:
  system: jira | github-issues | none
  project_key: null
  jira_site_url: null
  jira_email: null
  auth_env: null                             # env var NAME holding the API token, never the token itself
  issue_types_available: [ ... ]              # queried once the token is confirmed set; e.g. a default
                                               # template often lacks native Bug/Spike types
coding_standards: [ ... ]                     # only when doc_map_source is null
obsidian:
  vault_path: null
  decisions_subpath: decisions/
pentest:
  scope: null
  rules_of_engagement_doc: null
seo_aeo_geo_docs: [ ... ]                     # only when doc_map_source is null
---
```

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Jira is the assumed ticketing system; `ticketing.system: none` is a supported exception, not
  the default path.
- A re-run defaults to "refresh scanned fields only" — never silently overwrites Step 3 answers.
- A missing Jira token env var is a warning, not a hard stop, at this stage.
