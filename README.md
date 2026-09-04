# blackboxlabs-skills

Personal Claude Code skills.

## build-frontend

Orchestrates a full frontend page build/revision by combining the `impeccable` skill
(process/commands) and the `design-taste-frontend` (taste-skill) skill (anti-slop style),
reading each project's own theme file as the source of visual truth. See
[skills/build-frontend/SKILL.md](skills/build-frontend/SKILL.md) for the full instructions and
[skills/build-frontend/DESIGN_NOTES.md](skills/build-frontend/DESIGN_NOTES.md) for the design
rationale behind it.

### Requires

- [`impeccable`](https://github.com/pbakaus/impeccable)
- [`design-taste-frontend`](https://github.com/Leonxlnx/taste-skill) (taste-skill)

### Install into a repo

Using the `npx skills` CLI (scans this repo's `skills/` folder):

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "build-frontend"
```

Or copy it directly into a project's local skills folder:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/build-frontend your-project/.claude/skills/build-frontend
```

### Use

```
/build-frontend "a rental booking page for a car dealership"
/build-frontend "redo the pricing page hero" --ref "https://example.com pinned card-stack scroll effect"
```
