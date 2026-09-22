# Setup — commands to run in a target repo

Run these once per repo, before `build` can call into this skill.

## 1. Prerequisites

- **`define-project`** must have already run (`PROJECT.md` with `stack` filled in) — this skill
  reads `PROJECT.md.stack` to know which libraries already exist in the project before mapping a
  reference onto any of them.
- **`log-decision`** — queried and written to.
- **A browser-automation tool for live-site investigation.** This pipeline's declared dependency
  is Playwright (same choice `senior-engineer`'s own research-spike step already made). Two ways
  to satisfy it, either is fine:
  - A Playwright-capable MCP server already configured in this harness (check with your Claude
    Code client's own MCP list) — nothing further to install.
  - The `playwright` npm package driven directly via `Bash`/Node scripts, if no MCP server is
    configured:
    ```bash
    npm install -D playwright
    npx playwright install chromium
    ```
- **Existing-component references** (not a live site) need no browser tool at all — this skill
  reads real source files directly for those.

## 2. Install research-reference

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "research-reference"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/research-reference your-project/.claude/skills/research-reference
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked
up.

## 4. Run it

Normally invoked by `build` when either picks up a spike-labeled ticket.
Can also be run directly for ad hoc research, with no ticket involved:

```
/research-reference "https://example.com" "the scroll-driven hero section"
/research-reference "skiper31" "the character-scatter-to-center quote mechanism"
```
