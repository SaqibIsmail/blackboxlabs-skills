# Setup — commands to run in a target repo

Run these once per repo, before using `/founder-vision` there. This should be the very first
pipeline skill run on a brand-new project — before `/define-project`.

## 1. Install founder-vision

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "founder-vision"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/founder-vision your-project/.claude/skills/founder-vision
```

## 2. (Optional but recommended) Install log-decision

`founder-vision` calls `log-decision` on a re-run (recording why a pivot happened), not on the
first run. Not a hard dependency yet — install it before you expect to revise `VISION.md` later.

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked
up.

## 4. Run it

```
/founder-vision
```

No arguments. It asks up front whether this is a real business, an internal/intrapreneurial
project, or a side project — then runs the matching diagnostic or brainstorm — and writes
`VISION.md` at the project root.
