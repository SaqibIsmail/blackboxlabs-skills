# Setup — commands to run in a target repo

Run these once per repo, before using `/senior-engineer` there.

## 1. Prerequisites

This skill depends on several others already being installed and configured:

- **`define-project`** — must have already run (`PROJECT.md` with Jira connection details).
- **`define-epic`** — the epic this skill takes as input comes from there.
- **`log-decision`** — queried and written to throughout.
- **`plan-feature`** — called once per identified feature.
- **`jira-integration`** (ECC) — ticket creation and linking. Confirm the `mcp-atlassian` MCP
  server is configured (see `../define-project/SETUP.md` step 2).
- **`spec-miner`** (ECC) — brownfield behavior mining. Install alongside this skill (step 2
  below).
- **Playwright** — the research-spike step's live-reference inspection tool.

```bash
npm install -D @playwright/test
npx playwright install
```

## 2. Install senior-engineer and its ECC dependencies

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "senior-engineer"
npx skills add https://github.com/affaan-m/ecc --skill "spec-miner"
npx skills add https://github.com/affaan-m/ecc --skill "jira-integration"
```

If `npx skills` isn't available, copy manually:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/senior-engineer your-project/.claude/skills/senior-engineer
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skills are picked
up.

## 4. Run it

```
/senior-engineer PROJ-123
```

Pass the epic key/reference `define-epic` produced. It investigates, mines existing behavior if
relevant, checks for a UI reference, drives `plan-feature`, and presents a draft ticket list
before creating anything.
