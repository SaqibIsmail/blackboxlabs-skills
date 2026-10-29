# Setup — commands to run in a target repo

Run these once per repo, before other pipeline skills that call `log-decision` (`define-epic`,
`plan-feature`, `senior-engineer`, `build`, and later `resolve-pr-comments`).

## 1. Prerequisites

`/define-project` must already have run — `log-decision` reads `obsidian.vault_path` from the
project-root `PROJECT.md` stub (`decisions_subpath` no longer exists as a separate field; decisions
live in a fixed, company-keyed folder directly under `vault_path`). If `vault_path` is still
`null`, re-run `/define-project` in "full re-interview" mode, or hand-edit the `PROJECT.md` stub
directly with the path to your Obsidian vault (or any folder of Markdown files — Obsidian itself
isn't required, just the file format).

## 2. Install log-decision

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "log-decision"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/log-decision your-project/.claude/skills/log-decision
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked
up.

## 4. Try it directly (optional — most calls come from other skills)

```
/log-decision write --level epic --doc scope --epic "TEST-1:test-feature" "trying this out" "recorded a test entry" "none, just testing"
```

First real write against a fresh vault path will ask for confirmation before creating the
company folder.
