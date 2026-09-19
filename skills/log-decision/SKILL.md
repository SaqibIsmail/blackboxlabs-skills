---
name: log-decision
description: >
  Utility skill for a hierarchical decision log in an Obsidian vault — write mode records why
  something was built a certain way, query mode reads prior entries back, walking up the
  company → epic → story → ticket chain. Called by other pipeline skills (define-epic,
  plan-feature, senior-engineer, resolve-pr-comments, build-frontend, build-backend,
  research-reference) via the Skill tool, and directly invocable when the user wants to record or
  ask about a decision
  mid-conversation. Use when the user says "let's record this decision", "ADR this", "we decided
  to...", or "why did we choose X?".
argument-hint: 'write --level <company|epic|story|ticket> --doc <scope|scoping-calls> "<context>" "<decision>" "<consequences>" | query --ticket <key> | query --epic <key>'
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

## Vault structure

One vault, one company folder per project, nested by Jira hierarchy underneath. Every folder and
file below the company level is named `<jira-key>-<slug>` — the key is permanent and exact
(always matches Jira); the slug is generated once from the title at creation time and never
renamed afterward, even if the Jira title changes later (renaming would break every link pointing
at it).

```
<vault_path>/
  <company-slug>/
    vision.md                                  # written directly by founder-vision, not this skill
    project.md                                 # written directly by define-project, not this skill
    openspec/                                  # written directly by senior-engineer's mining steps, not this skill
      specs/<capability>/spec.md
      components/<name>/interface.md
    scoping-calls.md                           # this skill, level: company — rare, only for a decision
                                                # that outlives any single epic (e.g. "we standardize
                                                # on Resend for all future email needs")
    <epic-key>-<epic-slug>/
      scope.md                                 # this skill, level: epic, doc: scope
      scoping-calls.md                         # this skill, level: epic, doc: scoping-calls
      <story-key>-<story-slug>/                # only when senior-engineer used a Story tier
        scoping-calls.md                       # this skill, level: story, doc: scoping-calls
        <ticket-key>-<ticket-slug>.md          # this skill, level: ticket
      <ticket-key>-<ticket-slug>.md             # a ticket with no Story parent sits directly under the epic
```

**This skill only manages `scope.md`, `scoping-calls.md`, and ticket files.** `vision.md`,
`project.md`, and `openspec/*` are written directly by the skills that produce them
(`founder-vision`, `define-project`, `senior-engineer`'s Step 4a/4b) using plain `Write`/`Edit` —
they're relocated into the vault, but this skill doesn't manage their content or lifecycle.

**Where a decision belongs** — the only judgment call this skill's callers make:
- Affects only one ticket → that ticket's own file
- Affects multiple tickets *within one story* → that story's `scoping-calls.md`
- Affects multiple stories within one epic → that epic's `scoping-calls.md`
- Affects multiple epics → company-level `scoping-calls.md` (rare)

## Mode: write

**Args:**
- `level`: `company` | `epic` | `story` | `ticket`
- `epic`: `{key, slug}` — required for `epic`/`story`/`ticket` levels
- `story`: `{key, slug}` — required only if this ticket sits under a Story tier
- `ticket`: `{key, slug}` — required only for `level: ticket`
- `doc`: `scope` | `scoping-calls` — required for `company`/`epic`/`story` levels; ignored for
  `level: ticket` (a ticket always has exactly one file, itself)
- `context`, `decision`, `consequences`, and optionally `alternatives` (a list of
  `{name, pros, cons, why_not}`) and `deciders`

1. **First-time vault check:** if `<vault_path>/<company-slug>/` doesn't exist yet, **ask the user
   for confirmation before creating it** (never silently).
2. **Resolve the target path** from `level`/`epic`/`story`/`ticket` per the structure above,
   creating any missing folders in the chain (no separate consent needed per-folder — only the
   vault's own first creation, above, is gated).
3. **`doc: scope` (epic level only) is a single current-state document, not a log** — write or
   overwrite it directly (this is a deliberate re-scope, not a chronological entry; unlike the
   append-only docs below, an old scope isn't preserved unless the caller explicitly wants a
   revision history, in which case treat it like `VISION.md`'s own "append a dated section,
   mark superseded parts" convention instead of overwriting silently).
4. **`doc: scoping-calls` and every ticket file are append-only, dated logs.** If the file doesn't
   exist yet, create it with a one-line frontmatter (`tags: [...]` only — no per-entry
   `status`/numbering, unlike the old flat-vault format) and the first `## YYYY-MM-DD` entry. If
   it exists, append a new `## YYYY-MM-DD` section — never overwrite or remove a prior entry.
   Each entry:

```markdown
## YYYY-MM-DD
**Context:** <context arg>
**Decision:** <decision arg>
**Alternatives considered:** <alternatives, if given — name/pros/cons/why_not, one per line; omit
entirely if none were weighed>
**Consequences:** <consequences arg>
```

5. **Return the written file's path** to the caller — always, not optionally. This is the
   mechanism `senior-engineer` uses to embed a back-link in every ticket it creates (see "Ticket ↔
   decision back-link," below).

## Mode: query

**Args:** one of `ticket: {epic key, story key or null, ticket key}`, `story: {epic key, story
key}`, or `epic: {epic key}` — the caller names the most specific thing it's asking about, and this
skill walks **up** the chain from there, since a higher level's decisions still apply.

1. Resolve the full chain of files that apply, from most specific to least:
   - `ticket` query → `<ticket file>`, then `<story>/scoping-calls.md` (if a story key was given),
     then `<epic>/scoping-calls.md`, then `<epic>/scope.md`, then company `scoping-calls.md`
   - `story` query → `<story>/scoping-calls.md`, then `<epic>/scoping-calls.md`, then
     `<epic>/scope.md`, then company `scoping-calls.md`
   - `epic` query → `<epic>/scoping-calls.md`, then `<epic>/scope.md`, then company
     `scoping-calls.md`
2. For each file that exists, return its parsed entries (or, for `scope.md`, its whole content) as
   a structured object: `{path, level, entries: [{date, context, decision, consequences}, ...]}` —
   `scope.md` returns as a single entry with `date: null`.
3. Order: most-specific level first, each level's own entries most-recent-first within it.
4. No files found anywhere in the chain → return an empty list. Normal for a brand-new
   epic/ticket, not an error.

## Ticket ↔ decision back-link

Two-way linking. A ticket's own decision file lives *at* the ticket (not just referenced from it)
— so the "back-link" `senior-engineer` embeds in a Jira ticket's description is the vault path
this skill returned when the ticket's file was first written, e.g. `<vault_path>/<company>/
<epic-key>-<slug>/<story-key>-<slug>/<ticket-key>-<slug>.md`. Since the path is fully determined by
the ticket's own key chain, a caller that already knows the key chain can also derive it directly
without querying this skill first — the explicit return value is a convenience/confirmation, not
the only way to find it.

## Human-facing trigger phrases (direct invocation, not just other-skill calls)

- "let's record this decision" / "ADR this" → write mode
- Saqib is choosing between significant alternatives (framework, library, pattern, DB, API design)
  → offer to write mode, don't require it
- "we decided to..." / "the reason we're doing X instead of Y is..." → write mode
- "why did we choose X?" → query mode

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Folder/file naming is `<jira-key>-<slug>` everywhere below the company level — the key for exact,
  permanent identity; the slug (generated once, never renamed) for human readability while browsing.
- No global flat index file (no `README.md` of every decision) — the folder tree itself is the
  index; Jira is the index of *tickets*, this vault is the index of *why*, and nesting keeps the two
  aligned without a third list to maintain.
- `scope.md` is the one non-append-only document this skill manages — a current-state description,
  overwritten on a deliberate re-scope, not a growing log.
- Company-level `scoping-calls.md` is expected to be rare — most decisions belong to one epic or
  narrower; only use it for something that would genuinely misinform a *different* epic if it
  weren't visible there too.
