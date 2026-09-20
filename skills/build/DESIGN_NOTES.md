# `/build` — Pipeline Design

**Status (2026-09-19): design finalized, all open questions resolved (D1–D10 below, plus the two
`senior-engineer` fixes D7/D8) — `SKILL.md`/`SETUP.md` not yet written.** This file is the
provenance record building starts from, same order `log-decision` was itself resolved before
`senior-engineer`'s own implementation (see that file's opening note). **Supersedes `build-frontend`
and `build-backend`** — see the closing section below, and each file's own updated status note.

## What this is

The unified builder for one Jira ticket, replacing the two skills that used to split this job:
`build-frontend` (a complete, working `SKILL.md`, but built for free-text requests, predating the
ticket flow entirely — no ticket-key input, no PR/branch/review step at all) and `build-backend`
(never got past a draft, blocked on three open questions and a dependency on two `obra/superpowers`
skills that were cherry-picked as prose but never wired into a real control flow). Rather than fix
each in isolation, this design combines one control-loop engine with a scope-conditioned content
fork, so frontend/backend become a fork inside one engine instead of two competing top-level
skills:

- **Control loop — `obra/superpowers`'s `subagent-driven-development` skill** (288.8k★, MIT):
  worktree isolation, a committed ledger, a fresh implementer subagent per task, a separate task
  reviewer, a bounded fix loop, a final whole-branch review, then a real finish menu. Read in full
  before this plan was written — not a paraphrase — matching this repo's existing bar for
  cherry-picking (`senior-engineer`'s own citations of `garrytan/gstack` and BMAD-METHOD held to
  the same rule).
- **Its two sibling skills, same repo — `using-git-worktrees` and `finishing-a-development-branch`**
  (both `obra/superpowers`, 288.8k★, MIT) — cherry-picked into `skills/build/references/` this same
  day, full copies with a provenance note, same shape as the existing ECC pattern files.
- **Frontend content fork — `impeccable` (`pbakaus/impeccable`, 69.2k★, Apache-2.0) +
  `design-taste-frontend` (`Leonxlnx/taste-skill`, 88.5k★, MIT)** — carried straight over from
  `build-frontend`'s own dependency, re-grounded in that skill's own `DESIGN_NOTES.md`/`SETUP.md`
  and each source's actual docs, not `build-frontend`'s old paraphrase of them.
- **Backend content fork — TDD (`test-driven-development`) + `verification-before-completion`**
  (both `obra/superpowers`, 288.8k★, MIT) — carried over from `build-backend`'s draft
  `DESIGN_NOTES.md`, where both were already read in full and adopted (the Iron Law, the mandatory
  watch-it-fail step, the anti-rationalization table, delete-means-delete; and "no completion claims
  without fresh verification evidence").
- **Backend reality-check reference material — `affaan-m/ECC`** (262.9k★, MIT): five new reference
  files (below). `garrytan/gstack` (133.7k★, MIT) and BMAD-METHOD (`bmad-code-org/BMAD-METHOD`,
  53.2k★, no asserted license) were checked live for the same need and had nothing relevant — only
  ECC did.

All of the above are cherry-picked as prose/technique and filtered once against this project's own
locked stack — never wired in as a runtime dependency, same rule every other skill in this pipeline
already follows.

## Why one builder, not two (2026-09-19)

The premise for this whole design. `build-frontend` works but predates the ticket flow — it takes
free text, not a ticket key, and has no PR/branch/review step. `build-backend` stalled at a draft
with three unresolved open questions (self-QA placeholder, no criterion↔spec bridge, no backend
"focus flag" equivalent). Rather than patch each separately, the call was one builder, using
`subagent-driven-development` as the engine, with frontend/backend becoming a content fork *inside*
that one engine rather than two competing top-level skills — plus two things neither predecessor
had: an explicit, reviewable list of everything this builder consults before any code (Steps 1–8),
and a real verification gate on a spike ticket's research output before a later build ticket trusts
it (Branch A). `subagent-driven-development`, `using-git-worktrees`, and
`finishing-a-development-branch` were fetched and read from `obra/superpowers` in full to ground the
plan in what those skills actually say, not a paraphrase.

## Frontend content fork: carried over from `build-frontend`, three concrete corrections (2026-09-19)

The `impeccable`/`design-taste-frontend` dependency itself isn't re-decided — it's re-hosted inside
this skill's B3.5 step, grounded in each source's actual docs after reading `build-frontend`'s own
`DESIGN_NOTES.md`/`SETUP.md` for exact framing:

- `impeccable context`'s own directives are the self-check — `build` doesn't hand-roll an existence
  or hash check of its own on top of it. When `context` does say `PRODUCT.md`/`DESIGN.md` is
  missing or stale, `build` hands the triggered command extra context in the same dispatch
  (`vision.md` alongside `init`, `PROJECT.md`'s `stack:` list alongside `document`) instead of
  letting it start from a blank interview/scan.
- **No cap on `overdrive` anymore** — `build-frontend`'s old "at most once per page" rule doesn't
  carry over; `impeccable`'s own docs support a `[target]`-scoped `overdrive` per component/section,
  so a page can have more than one flourish moment if the task calls for it.
- **Dial-set reuse, once per page, via `log-decision`** — a page can span several sibling tickets
  (e.g. a Story's grouped card-section tickets), and taste-skill's own docs are explicit that one
  consistent VARIANCE/MOTION/DENSITY set gates the whole page. Redeciding per ticket risks two
  siblings picking inconsistent values for the same page, so only the *first* ticket to touch a
  page actually runs `shape` + dials for real, writing the result to the Story's
  `scoping-calls.md`; every sibling ticket reads instead of redecides. The epic goal itself travels
  as its own tagged block (`Epic goal (from epic_goal_ref): <content>`), kept distinct from the
  ticket's AC and from `PROJECT.md`'s stack, so a design-read claim can be traced back to its actual
  source instead of taken on faith.
- `extract` stays a periodic, non-blocking suggestion at B9 — same placement `build-frontend`
  already had it — and its consolidated result feeds the *next* ticket's B3.5 `impeccable context`
  call automatically, closing the loop rather than sitting unused in `DESIGN.md`.

## Backend content fork: TDD carried over, plus a new per-task reality check (2026-09-19)

`test-driven-development` and `verification-before-completion` (`obra/superpowers`) were already
read in full and adopted in `build-backend`'s draft `DESIGN_NOTES.md` — carried over unchanged. New
on top of that carryover: a per-task DB/API "reality check" that nothing upstream did before, run
fresh every task (never cached, since schema/routes can change between tickets unlike frontend's
page brief) —

- **DB**: read the project's real Flyway migration files (not a schema doc) to build the current
  table/column inventory, and decide, citing the migration file(s), what's reused vs. what needs a
  new migration.
- **API + contract**: grep the project's real route handlers for endpoints already covering this
  task's resource, and decide reuse vs. new explicitly. Concrete application of ECC's
  `contract-first` for this single-repo Next.js stack: since frontend and backend share one
  TypeScript build, the canonical artifact is a shared type in `features/shared/`, not OpenAPI — the
  ticket touching a resource first creates it, the sibling ticket imports it, and `pnpm typecheck`
  (already run at B4b/B6) catches drift for free the moment either side diverges — no separate
  contract-check process needed.

## Five new ECC reference files for the reality check (2026-09-19)

`postgres-patterns`, `database-migrations`, `api-design`, `contract-first`, and `database-reviewer`
— full cherry-picked copies from `affaan-m/ECC` (262.9k★, MIT), landing in
`skills/build/references/` alongside the existing `ecc-frontend-patterns.md`/`ecc-backend-patterns.md`.
Needed because the reality check above is a genuinely new per-task step nothing upstream ran before,
and it needs real backend-quality reference material to check schema/route/contract decisions
against — the existing high-level `ecc-backend-patterns.md` doesn't cover schema/index quality,
migration safety, or a DB-focused review checklist. Researched live across three candidate sources
— `affaan-m/ECC`, `garrytan/gstack` (133.7k★, MIT), and BMAD-METHOD (`bmad-code-org/BMAD-METHOD`,
53.2k★, no asserted license) — only ECC had anything relevant to this specific need; gstack's and
BMAD-METHOD's own techniques (already cherry-picked elsewhere in this pipeline, for investigation
rigor and epic/story structure respectively) don't cover backend schema/API quality at all. Filtered
once against `blackboxlabs`' locked stack (raw `pg`, no Supabase — drop the RLS/`auth.uid()`
sections that assume Supabase auth) before landing, same filter-once discipline already applied to
`ecc-backend-patterns.md`, not a per-run re-filter.

## Self-QA split, not one flat check (D1) (2026-09-19)

Resolves `build-backend`'s open Q2. Not one flat gate: B4b (per task) runs the fast subset — lint,
typecheck, this task's own affected tests — against the resolved standards docs; B6 (final) runs
the full suite plus standards-score across the whole ticket diff, using the project's real
CI-equivalent tooling (detected from the repo, e.g. `blackboxlabs`'s own `standards-score.mjs`
pattern), falling back to a generic checklist only if none exists. Applies to both scopes now, not
just backend — `build-frontend`'s old self-QA (`impeccable critique`/`audit`) was UX/a11y/perf-only
and never ran the project's own lint/typecheck/standards gate either. `resolve-pr-comments` +
Greptile stays a third, later gate on the opened PR — it exists for what a same-repo pre-PR check
can't see, not replaced by this. Motivating example: tonight's real two-round post-PR Standards
Review fix (73→98 on PR #16) becomes zero rounds once B4b catches the same class of issue before a
PR ever opens.

## Criterion-ID bridge, concrete rule (D2) (2026-09-19)

Resolves `build-backend`'s open Q3/TODO 3 — the one piece neither spec-kit nor `obra/superpowers`'s
TDD skill provides on its own. Every backend test file/block gets a `describe`/`it` name or comment
matching `AC-<N>: <criterion text>`. The task reviewer (B4b) greps the diff's test files for this
pattern and cross-checks each `AC-N` against `spec.md`'s real criteria — an implemented AC with no
matching tag fails; a tag citing a nonexistent AC also fails.

## No backend equivalent to B3.5 (D3.5) (2026-09-19)

Resolved: no, and not just by default. B3.5 exists because taste-skill's dials are a subjective
style choice that must stay visually consistent across a *page* — a real perceptual unit a sibling
ticket could visibly clash with. Backend has no equivalent unit: its consistency (API shape, error
handling, service-layer conventions) comes from fixed, pre-written project docs already loaded every
ticket (Steps 5/7), not something inferred fresh per ticket. Where a real architectural choice does
get made (D3's retry/cache/backoff pattern), it's naturally a `log-decision` entry already, and Step
3's walk-up already surfaces a sibling ticket's prior choice for the same thing — no separate
mechanism needed.

## Backend "focus flag" equivalent: one conditional prompt, not a mandatory ritual (D3) (2026-09-19)

Resolves `build-backend`'s open Q1. Backend resilience/perf choices are either required by the AC
already, or belong in `log-decision` as an architectural call — so instead of mirroring
`build-frontend`'s "focus flag"/`overdrive` step, there's one conditional prompt in the backend
brief: "if this task involves an external call/cache/queue, name the retry/cache/backoff pattern and
why, citing `ecc-backend-patterns.md`" — not a mandatory `AskUserQuestion` gate on every task.
Confirmed alongside this: `overdrive` is purely an `impeccable` (frontend) command with no backend
equivalent to mirror, so this isn't a compromise on parity.

## `using-git-worktrees` + `finishing-a-development-branch` cherry-picked in full (D5) (2026-09-19)

Read both in full already (see "Why one builder, not two," above). Became
`skills/build/references/using-git-worktrees.md` and `.../finishing-a-development-branch.md` — same
cherry-picked-reference convention as the two ECC pattern files, MIT, with a provenance note. Two
concrete corrections this already caught: use the native `EnterWorktree`/`ExitWorktree` tool (B1)
instead of raw `git worktree add`, since a native tool owns placement/branching/cleanup the fallback
can't see; and B9's Finish step is a real 3-option menu (merge locally / push + open a PR / keep
as-is) that waits for an answer, not an auto-PR default.

## Ledger committed to the repo, not worktree-scratch (D6) (2026-09-19)

Reverses the earlier "worktree-scratch, deleted at Finish" default — the audit trail needs to be
reviewable in the PR. Lives at the committed path `.build/<ticket-key>/progress.md` in the target
repo itself (dot-prefixed like the already-committed `.claude/`, not gitignored), committed as part
of that ticket's own commits, and stays in git history after merge — no cleanup step, since
persisting it is the whole point. This reverses `subagent-driven-development`'s own "delete this
plan's workspace" Finish-step default for this one file specifically; everything else in the
workspace (briefs, review packages, reports) stays worktree-local scratch as that skill designed it.
Follow-on this creates: `build` needs explicit resume logic — if invoked and
`.build/<ticket-key>/progress.md` already exists, read it first and resume at the first task without
a `complete` line, per `subagent-driven-development`'s own resume mechanic, rather than
re-dispatching already-done tasks (B2).

## Model tiering stays a per-task judgment call (D9) (2026-09-19)

Resolved: `build` chooses per task, no pre-named models per tier. Starting point stays as a judgment
call — boilerplate/simple CRUD → cheap; typical feature logic → standard; security-sensitive/
tricky/already-escalated → most-capable — not a fixed table, refined once it's actually run against
a real ticket. Recorded per task in the ledger before dispatching (B3), never inherited silently
from the session default.

## Spike branch dispatches by research-kind label — `research-ux` joins `research-reference` (D10) (2026-09-19)

Content/structure research ("what should this car-listing card even contain") is a different
unknown from mechanism research ("how is this scroll effect actually built"), and neither
`research-reference`'s Playwright/DOM sweeps nor its output shape fit the former — hence
`research-ux`, added as a real, implemented sibling skill (pushed to `blackboxlabs-skills` PR #1,
this same day). A spike ticket now carries `research-mechanism` or `research-ux`
(`senior-engineer` Step 7/9); Branch A's **A0** step reads that label to decide which skill to
dispatch — `research-reference` for a mechanism spike, `research-ux` for a content/structure spike.
A design-spike ticket (no idea existed at all) carries neither and isn't handled by Branch A the
same way — it's a request for a human design decision, not a research dispatch. **A3**'s
verification gate (structural completeness, fidelity/citation check, concreteness/"no guessing," and
zero-memory implementability) is adapted per which skill ran, not just applied to
`research-reference`'s own shape. The existing blocker-link mechanism (`senior-engineer` Step 7/9,
added the same day) already covers a `research-ux` spike blocking its dependent build ticket exactly
like a `research-mechanism` spike does — the one addition `build` itself needs is Step 3 reading a
blocker's vault file directly, since a blocker is a sibling in the hierarchy, not an ancestor, and
`log-decision`'s own ancestor walk-up never reaches it.

## Two `senior-engineer` fixes this skill's consult chain needs (D7, D8) (2026-09-19)

Found while designing this skill's Step 2/6 — see `senior-engineer/DESIGN_NOTES.md`'s own "Two more
Step 9 gaps found while designing the downstream `build` skill" entry, same date, for the full
write-up. Two things `senior-engineer` Step 9 needs to start writing into the real ticket, not just
show in its Step 8 draft view: the ticket's mapped `T0xx` task IDs (D7 — `build` Step 2 needs these
to isolate a ticket's task subset mechanically instead of matching AC text against `tasks.md` by
wording), and the `openspec/specs|components/*` baseline link when Step 4 produced one (D8 — same
embed-don't-just-link convention already used for the vault-file path and `scope.md`/
`scoping-calls.md` links). Also mechanical, not design decisions: `senior-engineer/SKILL.md`,
`log-decision/SKILL.md`, and `research-reference/SKILL.md`+`SETUP.md` get their remaining
`build-frontend`/`build-backend` cross-references renamed to `build`. Every file's own dated history
entries (including this one) stay untouched going forward — append-only rationale record, this
repo's own convention.

## Relationship to `build-frontend`/`build-backend` (2026-09-19)

Both are fully superseded (D4) — Saqib's own explicit call, not a default this skill picked. Neither
stays in use, including `build-frontend` despite being a complete, working skill: it predates the
ticket flow (free text only, no PR/branch/review step), so it doesn't fit alongside a ticket-driven
builder any more than `build-backend`'s stalled draft does. Both get the same "superseded by `build`"
banner `assign-tasks/DESIGN_NOTES.md` already set the precedent for — kept as files (provenance and
history, matching this repo's own never-delete convention), not deleted, and not invoked by anything
going forward, including as a standalone fallback for any use case. Their own `DESIGN_NOTES.md`
files get that banner added directly; their dated history stays untouched underneath it, same as
every other superseded file in this repo.

## Jira access is direct REST + `.env`, not `jira-integration`/MCP (2026-09-19)

Every prior draft of this skill (and `senior-engineer`'s own design) assumed `jira-integration`
(ECC), an MCP-based Jira read/write tool, as the ticket-access mechanism. Live-testing this skill
against `blackboxlabs`'s real Jira surfaced that this assumption was never actually verified end to
end: no `jira-integration` skill was installed anywhere, this session's own Atlassian connector was
scoped to a different org's site entirely, and the project's own `.mcp.json`-declared `mcp-atlassian`
server needs a session restart to load — none of which is a reliable path to depend on for every
`build` invocation.

Saqib's resolution: put `JIRA_API_TOKEN` in the target repo's own gitignored `.env` (matching
`PROJECT.md.ticketing.auth_env`, which already named this exact variable) and call the Jira Cloud
REST API directly (`https://<jira_site_url>/rest/api/3/...`, HTTP Basic Auth with
`jira_email`:`$JIRA_API_TOKEN`) — verified live against the real `blackboxlabss.atlassian.net` for
`SCRUM-19` before adopting this. Every Jira touchpoint in `SKILL.md` (Step 2's read, A6's
transition+comment, B9's transition) now says this explicitly instead of assuming an MCP tool.
`senior-engineer` still assumes `jira-integration` for ticket creation — not fixed here, since that
wasn't part of this change; worth revisiting the same way if it turns out to have the same gap.

## Visual comparison against the real reference, not just its prose write-up (2026-09-19)

Live-testing this skill against a real ticket (`SCRUM-11`, navbar) surfaced a real gap: Branch B's
implementer never actually looked at the reference site `research-reference` investigated for the
blocking spike (`SCRUM-9`) — it only had that skill's prose write-up (mechanism + stack-mapping) to
go on. Nothing checked, after the build, that the finished component actually resembled what was
researched. The result looked plausible on paper (right positioning, right tokens, right motion
timing) but was visually thin next to the actual reference, and Saqib called this out directly.

Three decisions, asked and confirmed directly:

1. **Where the visual check lives**: folded into B4b's existing task reviewer (one dispatch judges
   code/spec compliance and visual similarity together) rather than a new dedicated
   visual-verifier subagent. Simpler, and the reviewer already has to look at the built result
   either way.
2. **How a mismatch is handled**: blocking, following the exact same B4c fix loop and 5-round cap
   as any other reviewer finding — not a softer "flag for later polish" path. A visual gap is a
   real gap, not a nice-to-have.
3. **Whether `research-reference` should save a screenshot during its own investigation**: no —
   it passes the live URL forward via a new, mechanically-extractable `Reference source(s):` line
   (see `research-reference/SKILL.md` Step 7) instead. The builder and reviewer both re-visit the
   real site directly, so they see it as it exists now rather than a capture that could grow stale.

Mechanically: Step 3 now extracts `Reference source(s):` from a `research-reference`-sourced
blocker's write-up; B2's ledger carries it as a named pointer, `reference_urls` (same pattern as
`epic_goal_ref`), omitted entirely when there's no such provenance; B4a's frontend brief includes
the URL(s) and requires the implementer to actually browse them (not just read the summary) before
building; B4b's reviewer does the same and adds a required visual-similarity verdict to its
existing PASS/FAIL gate, not a separate check with its own outcome.

## Retrospective: research findings silently dropped at build time (2026-09-19/20)

Live-testing `SCRUM-11` (navbar) surfaced a deeper failure than the visual-comparison gate above
was built to catch, on the very first ticket it ran against. Saqib looked at the built result
twice and called it bad both times; tracing why revealed four compounding gaps, not one:

1. **A researched interactive mechanism (`v7labs.com`'s hover-expanding mega-menu) was silently
   dropped** when the implementer's brief was written — a unilateral simplification ("no dropdown
   for now"), never surfaced as a decision. The A3 gate checks that `research-reference`'s
   write-up is complete; nothing checked that the *build* actually implemented what it said.
2. **Verification kept checking DOM/functional correctness as a proxy for "looks right."** A link's
   `href` being present and correct says nothing about whether the page reads as designed — these
   are different questions, and treating one as evidence for the other let real gaps (missing edge
   padding, a broken flex layout) pass unnoticed.
3. **Screenshots were taken at a scaled-down size** and elements clipped near the edge were waved
   off as "a display artifact, confirmed not a real bug via `getBoundingClientRect`" — exactly
   backwards: the DOM query should confirm what a full-resolution screenshot already shows, not
   explain away what a compressed one hides.
4. **When the visual-comparison gate above was added mid-ticket, the ticket wasn't re-run through
   it.** The orchestrating session dispatched one more "fix it" subagent and did the before/after
   comparison itself, informally — the same one-shot-implementer-self-reports-success shape the
   whole `subagent-driven-development` engine exists to avoid, just performed by the orchestrator
   instead of a subagent.

Fixes, all in `SKILL.md` now:
- `research-reference`'s Step 7 requires a `Build-fidelity checklist` — every measured value *and*
  every named interactive mechanism as its own checkable line, not folded into prose.
- A3's structural-completeness check fails if that checklist is missing, same as a missing mapping.
- B4a's brief carries the checklist verbatim; the implementer can't drop a line without logging a
  Ruling naming which one and why.
- B4b's visual-comparison check is checklist-driven (line by line, explicit match/mismatch/missing)
  and screenshot-first (true resolution; a DOM query only confirms a number after the image already
  shows something, never substitutes for looking).
- B2 states explicitly that a `SKILL.md` process change mid-ticket forces a fresh B4a/B4b re-run
  under the updated steps, not an orchestrator hand-patch outside the loop.
