# /log-decision — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-10); not yet implemented as
`SKILL.md`. Open questions below block that.

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
  superseded and link forward to the new one, never silently overwrite.
- **Query** — given a feature name (or ticket ref), return the decision(s) on record for it,
  most recent first.

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
   every trivial choice."
2. ~~`vault_path` has no default...~~ **Resolved above** — ask before creating, per ECC.
3. Numbering (`NNNN-`) is global per vault or per-feature-subfolder? Global numbering is simpler
   but means every writer needs to know the current max across the whole vault before writing —
   a real race-condition risk if two skills ever write concurrently (unlikely today, since
   nothing in this pipeline runs FE/BE build agents and `log-decision` writes at truly the same
   instant, but worth deciding explicitly rather than by accident).

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve numbering scheme (open question 3) — it's a small decision with an annoying-to-fix-later blast radius if wrong.
2. Decide the exact query-result format other skills should expect back (a rendered summary? raw frontmatter + body? just the file paths for the caller to read itself?).
