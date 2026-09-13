# Setup — commands to run in a target repo

Run these once per repo, before using `/define-project` there.

## 1. Install define-project

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "define-project"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/define-project your-project/.claude/skills/define-project
```

## 2. If this project will use Jira (the expected default)

`define-project` only *records* your Jira connection details in `PROJECT.md` — it doesn't call
Jira itself. The actual Jira access is `senior-engineer`'s dependency, via ECC's
`jira-integration` skill, which needs the `mcp-atlassian` MCP server configured:

```bash
# Requires Python 3.10+ and uv (https://docs.astral.sh/uv/)
```

**Use a project-scoped `.mcp.json` in the repo root, not `~/.claude.json`.** `~/.claude.json` is a
large internal app-state file (caches, telemetry, account IDs) — too risky to hand-edit for this.
Create `.mcp.json` at the repo root:

```json
{
  "mcpServers": {
    "jira": {
      "command": "uvx",
      "args": ["mcp-atlassian==0.21.0"],
      "env": {
        "JIRA_URL": "https://YOUR_ORG.atlassian.net",
        "JIRA_EMAIL": "your.email@example.com",
        "JIRA_API_TOKEN": "PASTE_YOUR_TOKEN_HERE"
      }
    }
  }
}
```

**Add `.mcp.json` to `.gitignore` immediately** — it holds a real API token and must never be
committed.

Get an API token at <https://id.atlassian.com/manage-profile/security/api-tokens>. **If an AI
agent is helping you set this up: it must never type the actual token value into this file or
any command — only you should paste your real token in, directly, yourself.** The agent should
fill in every other field and leave the token as an explicit placeholder for you to replace.

This step can happen after `/define-project` runs — it will warn, not block, if the token env var
isn't set yet.

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked up.

## 4. Run it

```
/define-project
```

No arguments. First run interviews for whatever it can't answer by scanning the repo; re-running
later refreshes only the auto-scanned fields by default (see `SKILL.md`).
