---
name: log-decision
description: >
  Utility skill for a per-feature/per-epic decision log in an Obsidian vault — write mode records
  why something was built a certain way, query mode reads prior entries back. Called by other
  pipeline skills (define-epic, plan-feature, senior-engineer, resolve-pr-comments, build-frontend,
  build-backend) via the Skill tool, and directly invocable when the user wants to record or ask
  about a decision mid-conversation. Use when the user says "let's record this decision", "ADR
  this", "we decided to...", or "why did we choose X?".
argument-hint: 'write "<feature/epic>" "<context>" "<decision>" "<consequences>" | query "<feature/epic or ticket ref>"'
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
license: MIT
---

# /log-decision

A utility skill — not a pipeline stage that runs on its own. Other skills call into it via the
`Skill` tool; Saqib can also invoke it directly mid-conversation. Writes/reads plain Markdown
files in an Obsidian vault (`PROJECT.md.obsidian.vault_path`) — no Obsidian-specific tooling
needed, a vault is just a folder.

## Mode: write

**Args:** `feature` (feature/epic name or ticket ref), `context`, `decision`, `consequences`, and
optionally `alternatives` (a list of `{name, pros, cons, why_not}`) and `deciders`.

1. **First-time vault check:** if `<vault_path>/<decisions_subpath>/` doesn't exist yet, **ask the
   user for confirmation before creating it** (never silently). On confirmation, create the
   directory, seed `README.md` with an index-table header, and a blank `template.md`.
2. **Compute the next number:** `Glob` `<vault_path>/<decisions_subpath>/*.md` (excluding
   `README.md`/`template.md`), parse each filename's `NNNN-` prefix, take the max, use `max + 1`
   (or `0001` if none exist). Numbering is global per vault, not per-feature.
3. **Supersession check:** if this write's `feature` matches an existing entry that should now be
   marked superseded (the caller says so explicitly — never inferred), edit that prior file's
   `status: superseded` and add a `superseded_by: <new file path>` frontmatter field. Never
   silently overwrite an existing file.
4. **Write** the new file at `<vault_path>/<decisions_subpath>/NNNN-<slug>.md`:

```yaml
---
status: proposed | accepted | superseded
date: YYYY-MM-DD
feature: <feature/epic/branch name>
deciders: [...]
ticket-refs: []            # filled in later by senior-engineer once tickets exist
tags: [...]
---

## Context
<context arg>

## Decision
<decision arg>

## Alternatives Considered
### <name>
- Pros: ...
- Cons: ...
- Why not: <why_not>
(repeat per alternative — omit this whole section if none were weighed; don't invent a strawman)

## Consequences
### Positive
### Negative
### Risks
<consequences arg, split across these three however it naturally divides>
```

5. **Return the written file's path** to the caller — always, not optionally. This is the
   mechanism `senior-engineer` uses to embed a back-link in every ticket it creates (see "Ticket ↔
   decision back-link," below).

## Mode: query

**Args:** `feature` (a feature/epic name or ticket ref to search for).

1. `Grep` the `feature`/`ticket-refs` frontmatter fields across
   `<vault_path>/<decisions_subpath>/*.md` for a match, **excluding `README.md` and
   `template.md`** — the index's own table can coincidentally contain a matching string (e.g. a
   ticket key), which isn't a real decision entry.
2. For each match, return a structured object — not the raw file: `{path, status, date, feature,
   ticket_refs, tags, context, decision, consequences}`. `context`/`decision`/`consequences` are
   the plain-text body of those sections; `Alternatives Considered` is omitted from the
   structured result (the caller reads `path` directly if it needs that detail).
3. Sort results most-recent-`date`-first.
4. No matches → return an empty list. This is a normal, common result (most epics/features are
   new), not an error.

## Ticket ↔ decision back-link

Two-way linking. `ticket-refs` in the frontmatter points *from* a decision file *to* its tickets,
filled in by `senior-engineer` once tickets exist (a `write` call with only `ticket-refs` changed
— an update, not a new entry: find the file by `feature`, edit its frontmatter). The reverse holds
too: `senior-engineer` embeds the decision file's `path` (returned by this skill's `write` call)
directly in each ticket's description, so a ticket in Jira always has a way back to why it was
scoped that way.

## Human-facing trigger phrases (direct invocation, not just other-skill calls)

- "let's record this decision" / "ADR this" → write mode
- Saqib is choosing between significant alternatives (framework, library, pattern, DB, API design)
  → offer to write mode, don't require it
- "we decided to..." / "the reason we're doing X instead of Y is..." → write mode
- "why did we choose X?" → query mode

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Numbering is global per vault (simplest; the multi-writer race condition this could cause
  doesn't apply yet — nothing in this pipeline writes concurrently today).
- `Alternatives Considered` is optional per entry — skip it rather than invent a strawman
  alternative just to fill the section.
- Query returns structured fields, not raw files — callers needing the full file (e.g.
  `Alternatives Considered`) read `path` themselves.
