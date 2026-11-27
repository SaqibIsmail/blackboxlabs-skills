# Setup — commands to run in a target repo

Run these once per repo, before using `/define-epic` there.

## 1. Prerequisites

`/define-project` must already have run in this repo — `define-epic` reads `PROJECT.md` for the
Jira project key and connection details. If it hasn't, run that first (see
`../define-project/SETUP.md`).

`define-epic` depends on `jira-integration` (ECC) for both the dedupe search and epic creation —
confirm the `mcp-atlassian` MCP server is configured (see `../define-project/SETUP.md` step 2 for
the exact config block) before your first real run. It will still run without it, but Step 2
(dedupe) and Step 5 (epic creation) will fail with a clear error until it's set up.

## 2. Install define-epic

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "define-epic"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/define-epic your-project/.claude/skills/define-epic
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked
up.

## 4. Run it

```
/define-epic "user notifications system"
```

It checks Jira for similar existing epics first, then interviews for why/scope, then creates the
epic.
