# Setup — commands to run in a target repo

Run these once per repo, before using `/plan-feature` there.

## 1. Prerequisites

- **`define-project`** must have already run (`PROJECT.md`).
- **`log-decision`** — queried and written to.
- **`github/spec-kit`** — the actual spec/plan mechanics this skill drives. Install per its own
  docs:

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init --here
```

## 2. Install plan-feature

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "plan-feature"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/plan-feature your-project/.claude/skills/plan-feature
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked
up.

## 4. Run it

Normally invoked by `senior-engineer`, once per feature under an epic. For a standalone
single-feature project with no epic layer, it can also be run directly:

```
/plan-feature "user notification preferences"
```
