# Setup — commands to run in a target repo

Run these once per repo, before using `/build` there.

## 1. Prerequisites

- **`define-project`** must have already run (`PROJECT.md` with `stack` filled in) — `build` reads
  `PROJECT.md.stack` at Step 1 of every ticket, then again directly (never trusting a cached claim)
  in Branch A's stack-fidelity check, and B3.5's design read draws on `PROJECT.md`'s ticketing
  config too.
- **`log-decision`** — queried at the start of every ticket (ticket → story → epic → company
  walk-up, plus a blocker's own vault file when one is linked) and written to throughout: Branch
  A's verify entry, B3.5's per-page surface-brief cache, B8's deviation log, B9's finish entry.
- **`jira-integration`** (ECC) — ticket read (`jira_get_issue`) and write (comments, label/status
  transitions) for every ticket `build` touches. Confirm the `mcp-atlassian` MCP server is
  configured (see `../define-project/SETUP.md` step 2) before your first run.
- **A worktree tool, for Branch B (task/bug tickets) only** — spike tickets (Branch A) never touch
  git, so this isn't needed to run `build` against a research spike. Two ways to satisfy it,
  in priority order:
  - This harness's own native worktree tool (commonly surfaced as `EnterWorktree`/`ExitWorktree`)
    — check your Claude Code client's own tool list; if present, nothing further to install.
  - No native tool available: the manual `git worktree add` fallback, fully documented in this
    skill's own `skills/build/references/using-git-worktrees.md`. Nothing to install — just read
    it once so the directory-selection and `.gitignore` safety steps are followed correctly.
- **Playwright** — Branch A's `research-reference` dispatch needs it for live-site inspection. This
  is the same pipeline dependency `research-reference/SETUP.md` already documents, not a new one
  `build` introduces: a Playwright-capable MCP server already configured in this harness, or the
  `playwright` npm package it documents installing directly. Confirm `research-reference` itself is
  set up before running `build` against a `research-mechanism`-labeled spike ticket.
- **`impeccable` + `design-taste-frontend`** — for `frontend`/`shared`-scope task/bug tickets only
  (B3.5's surface-brief + design-read step). Same external skills and install commands
  `build-frontend/SETUP.md` already documents, carried over unchanged — `build` forks the
  ticket-scoped content built on top of these, not the skills themselves.

## 2. Install the dependency skills

```bash
npx impeccable install
```

Interactive — it detects your harness (Claude Code, Cursor, etc.) and asks project vs global
scope. Choose project-scoped for a single repo.

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
npx skills add https://github.com/affaan-m/ecc --skill "jira-integration"
```

## 3. Install build itself

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "build"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/build your-project/.claude/skills/build
```

This pulls in `skills/build/references/` too (`using-git-worktrees.md`,
`finishing-a-development-branch.md`, and the ECC pattern files) — nothing separate to fetch for
those.

## 4. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked up.

## 5. Run it

```
/build SCRUM-19
```

Pass a ticket key, not free text — `build` reads that ticket's `type` label to fork: a
`spike`-labeled ticket runs research-only (Branch A — no code, no worktree, dispatches into
`research-reference`/`research-ux`), while a `task`/`bug`-labeled ticket runs the full
worktree → implement → review → fix-loop → finish engine (Branch B), forking only the per-task
implementer's brief by the ticket's `scope` label (`frontend`/`backend`/`shared`).

If `.build/SCRUM-19/progress.md` already exists in the repo from an earlier interrupted run,
`/build SCRUM-19` resumes it — it reads the ledger and picks up at the first task without a
`complete` line, rather than re-dispatching tasks already done.
