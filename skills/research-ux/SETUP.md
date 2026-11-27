# Setup — commands to run in a target repo

Run these once per repo, before `build` can call into this skill.

## 1. Prerequisites

- **`define-project`** must have already run (`PROJECT.md` with `stack` filled in) — this skill
  reads `PROJECT.md.stack` so any recommendation stays buildable with what the project already has.
- **`log-decision`** — queried and written to.
- **A way to read a real page's content** — `WebFetch` (already available in most harnesses) covers
  most cases; a full browser tool is only needed if a comparable example requires real interaction
  (e.g. content that only appears after a click) to see, which is uncommon for this skill's own
  scope (content/structure, not mechanism — see `research-reference`'s own Playwright dependency
  for that different case).

## 2. Install research-ux

```bash
npx skills add https://github.com/SaqibIsmail/blackboxlabs-skills --skill "research-ux"
```

If `npx skills` isn't available, copy it manually instead:

```bash
git clone https://github.com/SaqibIsmail/blackboxlabs-skills.git /tmp/blackboxlabs-skills
cp -r /tmp/blackboxlabs-skills/skills/research-ux your-project/.claude/skills/research-ux
```

## 3. Reload the harness

Restart or reload your Claude Code session in this repo so the newly installed skill is picked up.

## 4. Run it

Normally invoked by `build` when it picks up a `research-ux`-labeled ticket. Can also be run
directly for ad hoc research, with no ticket involved:

```
/research-ux "car listing card, for a peer-to-peer marketplace" "blocks the card-grid build ticket"
/research-ux "voice-receptionist product page" "blocks the standard-pages build ticket"
```
