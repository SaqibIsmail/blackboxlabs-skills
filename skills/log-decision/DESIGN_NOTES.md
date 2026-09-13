# /log-decision — Pipeline Design

**Status (2026-09-13): implemented.** See `SKILL.md`/`SETUP.md`. Built ahead of its place in the
original front-half order because `senior-engineer` (and `define-epic`/`plan-feature` before it)
depend on it directly — Saqib flagged this exact gap before starting `senior-engineer`. This file
stays as the rationale record.

**Still open, not blocking:** open question 1, below (the `build-frontend`/`build-backend` write
trigger) — specific to those two skills, which this pass doesn't reach yet.

## What this is

A utility skill, not a pipeline stage that fires on its own — other skills call into it (via the
`Skill` tool) to write or query a per-feature decision log, so that later changes have a
recorded rationale for why something was built a certain way. It writes into an Obsidian vault,
but needs no Obsidian-specific tooling: a vault is just a folder of Markdown files, so this skill
uses plain file read/write, nothing more.

This is deliberately a finer-grained, different thing from a project's own high-level decision
record (e.g. `blackboxlabs/AGENTS.md`'s "Decision register" — a small, static, hand-maintained
table of *locked architectural* choices like DB client or hosting platform, rarely revisited).
`log-decision` is per-feature, per-PR, meant to accumulate continuously as work happens — it
generalizes the same "why was this built this way" idea `build-frontend/DESIGN_NOTES.md` already
embodies for itself, just from one static file per skill to one file per decision per feature.

**Format and two of the open questions below now fold in `affaan-m/ECC`'s
`architecture-decision-records` skill** (cherry-picked as prose/technique, not runtime — same
rule as every other skill in this pipeline): a richer ADR body than plain MADR-minimal, an
explicit human-facing trigger list (this skill isn't only called by other skills — Saqib can
invoke it directly mid-conversation), and a concrete answer to open question 2 (vault
initialization).

## Format

Lightweight ADR (Michael Nygard's format, per ECC's adaptation) — richer than plain MADR-minimal,
because "why was this rejected" is exactly the content this pipeline most wants preserved. One
file per decision at `<vault_path>/<decisions_subpath>/NNNN-<slug>.md` (numbering convention
borrowed from `adr-tools`, without adopting its unmaintained CLI):

```yaml
---
status: proposed | accepted | superseded
date: YYYY-MM-DD
feature: <feature/branch name>
deciders: [...]                   # who/what was involved — usually just Saqib + the calling skill
ticket-refs: [FE-123, BE-124]      # filled in later by senior-engineer once tickets exist
tags: [...]
---

## Context
## Decision
## Alternatives Considered
### Alternative 1: [Name]
- Pros / Cons / Why not
### Alternative 2: [Name]
- Pros / Cons / Why not
## Consequences
### Positive / Negative / Risks
```

`Alternatives Considered` is optional-but-encouraged, not mandatory — a trivial decision with no
real alternative weighed can skip straight to Consequences rather than inventing a strawman
alternative just to fill the section.

## Human-facing trigger phrases (direct invocation, not just other-skill call sites)

Borrowed from ECC's activation list — `log-decision` isn't only invoked by other skills; Saqib
can call it directly mid-conversation:

- "let's record this decision" / "ADR this"
- Choosing between significant alternatives (framework, library, pattern, DB, API design)
- "we decided to..." / "the reason we're doing X instead of Y is..."
- "why did we choose X?" (query mode — read existing entries, don't write)

## Modes

- **Write** — append a new decision file, or (if `status: superseded` applies) mark a prior file
  superseded and link forward to the new one, never silently overwrite. **Always returns the
  written file's path** (`<vault_path>/<decisions_subpath>/NNNN-<slug>.md`) to the caller — this
  is the mechanism that lets `senior-engineer` embed a link back to the decision in each ticket it
  creates (see "Ticket ↔ decision back-link," below). Not an afterthought: the return value is
  part of this skill's contract, same as any other skill's declared output.
- **Query** — given a feature name (or ticket ref), return the decision(s) on record for it, most
  recent first — each result includes its file path, for the same reason.

## Ticket ↔ decision back-link

Two-way linking, not just one direction. `ticket-refs` in the frontmatter above already points
*from* a decision file *to* its tickets. The reverse must also hold: every ticket
`senior-engineer` creates embeds a path/link *back* to the relevant decision file(s) (the epic's
entry from `define-epic`, and the feature's entry from `plan-feature`, whichever apply) in its
description — the same traceability convention already planned for linking a ticket to its
`spec.md`/`plan.md`, just extended to cover the Obsidian decision log too. Without this, someone
reading a ticket in Jira has no way back to *why* it was scoped the way it was — only someone
reading the Obsidian side would see the tickets it produced.

## Vault initialization (resolves open question 2, below)

Per ECC's own discipline: if `<vault_path>/<decisions_subpath>/` does not exist yet on first
write, **ask the user for confirmation before creating it** — seed a `README.md` with an index
table header and a blank `template.md` for manual use. Never create vault files without explicit
consent, since `vault_path` is almost certainly outside this repo (a personal Obsidian vault).

## Call sites (which other skills write/read, and when)

| Stage | Writes | Reads |
|---|---|---|
| `define-epic` | Yes — first entry when the epic is scoped (scope, non-goals, why-now) | On resume, if this epic already has entries |
| `plan-feature` | Yes — entry when `plan.md` finalizes (scope, chosen approach, explicit non-goals), one per feature under the epic | On resume, if this feature already has entries |
| `senior-engineer` | Enriches the epic's entry with `ticket-refs` once tickets exist; also writes when investigation surfaces a non-obvious scoping call | Before investigating, checks for prior entries on this epic or the systems it touches |
| `build-frontend` / `build-backend` | Only when a non-obvious deviation from the plan actually occurs during implementation — not every run | Before implementing, checks for a prior decision that constrains this feature |
| `resolve-pr-comments` | Yes, always — records why a review comment was resolved the way it was | **Yes, hard requirement** — before editing anything |
| `pentest-app` / `audit-seo` | Optional — only for a deliberate accepted-risk/exception | — |

## Open questions

1. Exact write trigger for "a non-obvious deviation happened" in `build-frontend`/`build-backend`
   is inherently fuzzy — this repo already tolerates similar fuzziness elsewhere (`build-frontend`'s
   own `--ref` Mode-1-vs-Mode-2 split), but it's worth naming concrete examples before
   implementation so the two build skills don't drift into either "never writes" or "writes on
   every trivial choice." Left open — it's specific to `build-frontend`/`build-backend`, which
   this implementation pass doesn't reach yet.
2. ~~`vault_path` has no default...~~ **Resolved above** — ask before creating, per ECC.
3. ~~Numbering (`NNNN-`) is global per vault or per-feature-subfolder?~~ **Resolved
   (2026-09-13):** global per vault. List existing files in `decisions_subpath`, parse the
   `NNNN-` prefix of each, take the max, write at `max + 1`. Simple; the race-condition risk this
   question named doesn't apply yet since nothing in this pipeline writes concurrently.

## Query result format (resolves the former TODO 2)

Each result is a structured object, not a raw file dump: `{path, status, date, feature,
ticket_refs, tags, context, decision, consequences}` — parsed frontmatter fields plus the `##
Context`/`## Decision`/`## Consequences` body sections extracted as plain text (skip
`Alternatives Considered` unless the caller asks for full detail — most callers only need the
decision and why, not the road not taken). Sorted most-recent-`date`-first. A caller that wants
the raw file (e.g. to read `Alternatives Considered`) can always read `path` directly.

## Findings from live testing against `blackboxlabs` (2026-09-13)

Vault-creation consent gate worked as designed (asked before creating `brain/decisions/`, user
confirmed). Numbering worked (`0001-...`, no existing files). One bug found and fixed: the query
grep pattern matched `README.md` (its own index table coincidentally contained the search term)
alongside the real decision file — `SKILL.md` now explicitly excludes `README.md`/`template.md`
from query matches.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.
