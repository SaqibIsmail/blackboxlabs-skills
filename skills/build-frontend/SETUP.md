# Setup — commands to run in a target repo

Run these once per repo, in order, before using `/build-frontend` there.

## 1. Install the dependency skills

```bash
npx impeccable install
```

Interactive — it detects your harness (Claude Code, Cursor, etc.) and asks project vs global
scope. Choose project-scoped for a single repo.

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
```

## 2. Install build-frontend itself

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "build-frontend"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/build-frontend your-project/.claude/skills/build-frontend
```

## 3. Confirm the repo prerequisite

`build-frontend` reads the project's own theme file to learn its colors, typography, and allowed
animation libraries — it does not invent these. Confirm the repo has one (commonly `theme.ts` or
`theme.config.ts`) before your first run. If it's missing, the skill will ask you once and record
the answer, but it's cleaner to have it in place first.

## 4. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skills are
picked up.

## 5. Run it

```
/build-frontend "a rental booking page for a car dealership"
```

or, with a reference for the animation direction:

```
/build-frontend "redo the pricing page hero" --ref "https://example.com pinned card-stack scroll effect"
```

First run in a new repo will also trigger `impeccable init` automatically (one-time product
context interview) if `PRODUCT.md` doesn't exist yet — no separate command needed for that.
