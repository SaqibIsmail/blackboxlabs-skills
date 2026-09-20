---
name: build
description: >
  Unified builder for one Jira ticket. Reads the ticket's full context chain, then forks on the
  ticket's type label: a spike ticket calls research-reference and independently verifies its
  findings before embedding them; a task/bug ticket runs subagent-driven-development's engine
  (worktree, ledger, fresh-implementer-per-task, task review, fix loop, final review, finish) over
  that ticket's task subset, forking only the per-task implementer's brief content by the ticket's
  scope label (frontend/backend/shared) — not the control flow. Supersedes build-frontend and
  build-backend outright — neither stays in use.
argument-hint: '<ticket-key>'
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
  - Skill
  - AskUserQuestion
  - EnterWorktree
  - ExitWorktree
license: MIT
---

# /build

Picks up one Jira ticket and carries it through to done. Reads the ticket's full context chain
first, the same eight-step consult every branch below shares, then forks on the ticket's `type`
label: a `spike` ticket (Branch A) dispatches the matching research skill and independently
verifies its findings before anything gets trusted or embedded; a `task`/`bug` ticket (Branch B)
runs `subagent-driven-development`'s engine — worktree-isolated, ledger-tracked, a fresh
implementer subagent per task, task review, a fix loop, a final whole-branch review, then finish —
over that ticket's own task subset, forking only the per-task implementer's brief content by the
ticket's `scope` label (`frontend`/`backend`/`shared`), never the control flow itself. Supersedes
`build-frontend` and `build-backend` outright — neither stays in use, for any case.

## Step 1 — Read `PROJECT.md`

Read the project-root stub, then `<vault_path>/<company-slug>/project.md` for the rest — stack,
`doc_map_source`, the `coding_standards` fallback list, and ticketing config. Same first move every
skill in this pipeline already makes.

## Step 2 — Read the ticket

Fetch it via a direct REST call — `GET https://<ticketing.jira_site_url>/rest/api/3/issue/<key>`,
HTTP Basic Auth with `<ticketing.jira_email>:$JIRA_API_TOKEN` (the token read from this project's
own `.env`, matching `ticketing.auth_env`; the email/site are non-secret and come straight from
`PROJECT.md`) — not through an MCP-based `jira-integration` tool. Never hardcode the token, echo
it, or let it appear in a logged command string; read it via shell env expansion only. Pull
`type`/`scope` labels, parent Epic/Story keys, the story/AC body, and its embedded `T0xx` task-ID
list when the ticket description carries one.

## Step 3 — Query `log-decision`

Call it in **query mode** with `ticket: {epic key, story key or null, ticket key}` (query mode's
own argument shape — not `level`, which belongs only to write mode). This auto-walks ticket file →
story `scoping-calls.md` → epic `scoping-calls.md`/`scope.md` → company `scoping-calls.md`. Existing
entries are a resume, not something to re-derive.

Also read the blocker's own vault file directly, if Step 2's ticket carries a "blocked by" link
(per `senior-engineer`'s blocker-link step) — a blocker is a sibling in the hierarchy, not an
ancestor, so `log-decision`'s own walk-up never surfaces it on its own. This is how a
`research-ux`/`research-reference` spike's findings actually reach the ticket that depends on
them, not just tickets under the same story/epic.

**If the blocker is a `research-reference` spike**, extract its `Reference source(s):` line (see
`research-reference/SKILL.md` Step 7) — the exact live URL(s) it investigated. This isn't just
prose context: Branch B's B4a and B4b (below) need the real URL to browse themselves, not a
paraphrase of what it looks like. Carry it forward as `reference_urls` (B2's ledger).

## Step 4 — Load the feature plan

Read `specs/<feature>/{spec.md,plan.md,tasks.md}`, then isolate this ticket's own `T0xx` subset.
Prefer the ticket's own embedded task-ID list from Step 2 when present; when it isn't, isolate the
subset by matching the ticket's AC text against `tasks.md` instead.

## Step 5 — Resolve documentation routing

Read `doc_map_source` (the target repo's own `AGENTS.md#Documentation map`), filtered to what
*this ticket's* task subset actually touches: the `scope` label as the coarse filter, then
file-path/keyword match per task. Never load every backend doc for a ticket that only touches one
route.

## Step 6 — Load mined baselines, if any

Load `<vault_path>/<company-slug>/openspec/specs/<capability>/spec.md` and/or
`<vault_path>/<company-slug>/openspec/components/<component-name>/interface.md` when either applies
to this ticket. Prefer the ticket's own embedded baseline link when present; when it isn't, derive
the path directly from the capability/component name named in `spec.md`.

## Step 7 — Load coding-standards docs

Load more `doc_map_source` rows if set; otherwise load `PROJECT.md`'s `coding_standards` fallback
list in full.

## Step 8 — Load this repo's own pattern reference

Load, filtered by `scope` label:

- `frontend`/`shared` → `skills/build/references/ecc-frontend-patterns.md`
- `backend`/`shared` → `skills/build/references/ecc-backend-patterns.md`

This is a different source from Steps 5/7 — this repo's own pre-filtered generic patterns, not the
target repo's own docs — and feeds B4a's implementer brief directly, not B4b's reviewer standards
check.

## Step 9 — Branch on `type`

`spike` → Branch A. `task`/`bug` → Branch B. Anything else: stop, ask — don't guess.

### Branch A — spike ticket (produces no code)

**A0 — Dispatch by research-kind label.** A spike ticket carries `research-mechanism` or
`research-ux` (assigned by `senior-engineer`) — dispatch `research-reference` for the former,
`research-ux` for the latter. A design-spike ticket (no idea existed yet at all) carries neither
and isn't a spike this branch handles the same way — it's a request for a human design decision,
not a research dispatch.

- **A1** — Dispatch the chosen skill (via `Skill`) with the reference/subject, "what it needs to
  become"/"why it matters" from the ticket, plus Steps 1–8's gathered context. `build` is
  `research-reference`'s documented caller and `research-ux`'s only caller.
- **A2** — Re-read the actual write-up wherever Step 6/7 of the dispatched skill put it (ticket
  description / its `log-decision` entry) as fresh input — its own "done" claim isn't the gate.
- **A3 — Verify (the real gate), adapted by which skill ran:**
  - **Structural completeness**: for `research-reference`, both halves present and non-placeholder
    — the real-mechanism write-up (library/DOM/keyframes/trigger) *and* the stack-mapping (the
    concrete file/component it becomes here) *and* a `Build-fidelity checklist` (its own required
    third part, per `research-reference/SKILL.md` Step 7) itemizing every measured value and every
    distinct interactive mechanism found — missing the checklist fails this check even if the prose
    above it is thorough, since the checklist (not the prose) is what B4b's reviewer later checks
    the finished build against line-by-line. For `research-ux`, the near-mandatory/differentiator
    split *and* the recommended set for this specific ticket — a flat undifferentiated list fails
    this check regardless of how detailed it looks. Either skill: missing or "TBD" on its required
    half fails.
  - **Fidelity check**: for `research-reference`, re-read `PROJECT.md.stack` directly (don't trust
    its own check) and confirm every named library in the mapping is actually in it, unless
    explicitly flagged as a new-library decision resolved via `log-decision`. For `research-ux`,
    confirm every claimed near-mandatory field/action actually cites which comparable example(s)
    it came from — an uncited claim fails this check the same way an unverified stack claim would.
  - **Concreteness ("no guessing") test**: for `research-reference`, every claimed behavior needs
    an exact target file plus real values — hex/rgb, px/ms, easing curve, prop/selector names. For
    `research-ux`, every recommended field/action needs to be a specific, nameable thing ("year,
    make, model, mileage, price, one photo, a 'View details' CTA"), not a category ("relevant
    vehicle info"). Grep for hedge language ("something like," "TBD," "roughly," "relevant," "as
    needed") on either skill's output — a hit with no attached specific value/name fails that item.
  - **Zero-memory implementability**: read the write-up as a session that never ran the research —
    could it write the first line of code (or the first line of a component's markup) from the
    text alone? Any unanswered "which file / which existing component / which exact prop / which
    exact field" question restates a concreteness failure as the actual acceptance test.
- **A4 — On failure**: don't embed anything. Either send `research-reference` back with the
  specific named gap (cap 2 corrective rounds), or — if the gap is a genuine open stack decision —
  stop and ask via `AskUserQuestion` rather than resolve it silently.
- **A5 — On pass**: embed the verified write-up (both halves, plus the `Build-fidelity checklist`
  for `research-reference`) directly in the ticket description (embed, never just link — this
  pipeline's standing convention). Append a short "verified `<date>` by `build` — passes
  structural/stack-fidelity/concreteness gate" entry to the ticket's own vault file via
  `log-decision` (write mode, `level: ticket`) — not a raw file edit, so the same
  graph-connectivity maintenance (parent link + children table) that every other vault write gets
  still applies here.
- **A6 — Close the ticket**: `GET .../issue/<key>/transitions` to find the right transition ID,
  then `POST .../issue/<key>/transitions` with it, and `POST .../issue/<key>/comment` noting that
  the dependent build ticket can proceed with the mapping now embedded and verified — same direct
  REST + `.env` auth as Step 2, no MCP tool involved.

Branch A never touches git — no worktree, no implementer dispatch, no PR.

### Branch B — task/bug ticket (`subagent-driven-development` wraps this ticket)

**Autonomy contract** (stated once, applies throughout): rulings-not-stalls. Only four things stop
this branch for the user: an irreversible/destructive op, a security-sensitive action, a side
effect outside the worktree (push/merge/publish to shared state), or a plan so broken every path
forward is a guess. Everything else is a judgment call, logged as `Ruling: <what> — <why> — <cost
if wrong>`.

- **B1 — Worktree setup.** Per `using-git-worktrees`: detect existing isolation first
  (`GIT_DIR`/`GIT_COMMON`); if none, use this harness's own native worktree tool
  (`EnterWorktree`/`ExitWorktree`) rather than raw `git worktree add`, since a native tool owns
  placement/branching/cleanup the fallback can't see. Get consent before creating one if no
  preference is already on record.
- **B2 — Ledger init, check for a resume first.** The ledger lives at the *committed* path
  `.build/<ticket-key>/progress.md`, not worktree-scratch — check whether it already exists before
  creating it. If it does (a prior session was interrupted, or this branch is being picked back
  up), read it and resume at the first task without a `complete` line — don't re-dispatch tasks the
  ledger already shows done, the same mechanic `subagent-driven-development` itself uses for its
  own (scratch) ledger. Only on a genuine first run: seed it fresh with the ticket key/title, the
  Step 4 task subset in dependency order, the scope label, and *pointers* to Steps 1–7's gathered
  context (not copies) — so a resumed session reconstructs context from the ledger alone. The
  epic's `scope.md` entry (from Step 3's walk-up) gets its own named pointer —
  `epic_goal_ref: <path to the epic's scope.md>` — not folded anonymously into a generic "Steps
  1–7 context" bucket, since B3.5 needs to cite it specifically and a generic bucket would make
  that a re-derivation instead of a lookup. **`reference_urls: <url1>, <url2>, ...`** gets the same
  named-pointer treatment when Step 3 extracted one from a `research-reference` blocker — omit the
  field entirely when the ticket has no such provenance, rather than writing it empty; B4a and B4b
  both key off its presence to decide whether a visual-comparison pass applies at all.

  **A resume is for picking up an interrupted ticket, not for patching around a process change
  mid-flight.** If `build`'s own steps changed (a `SKILL.md` edit) after this ticket's B4 loop
  already ran once, that is not a resume — re-dispatch a fresh B4a implementer and a fresh B4b
  reviewer through the *updated* steps for the affected task(s), rather than the orchestrating
  session patching the result by hand outside the loop. An orchestrator editing files directly to
  "fix" a finding is exactly the failure mode the implementer → reviewer → fix-loop structure
  exists to avoid — a one-shot change with no independent second pass checking it.
- **B3 — Model tiering**, recorded per task in the ledger before dispatching — never inherited
  silently from the session default. Choose per task, not off a fixed table: boilerplate/simple
  CRUD work → the cheapest tier; typical feature logic → the standard tier; security-sensitive,
  tricky, or already-escalated work → the most-capable tier. Treat this as a judgment call to
  refine against real tickets, not a fixed lookup table.
- **B3.5 — Frontend surface framing, once per PAGE, not per ticket (frontend/shared scope only).**
  - Run `impeccable context` once, at the start of this step. It loads whatever `PRODUCT.md`/
    `DESIGN.md`/surface-brief state already exists and returns directives saying what's missing or
    stale — this is the self-check already built into `impeccable` itself; `build` doesn't
    hand-roll an existence or hash check of its own. Follow those directives as-is: if it says
    `PRODUCT.md` is missing, that directive triggers `init` (one-time interview for
    audience/purpose/voice); if it flags `DESIGN.md` missing/stale against the real theme/code,
    that triggers `document` (regenerates it from what's actually built). Neither runs unless
    `context` itself says to. When either does fire, hand over extra context in the same dispatch
    rather than letting them start from a blank interview/scan: `init` gets `vision.md`'s content
    alongside it (its `Target user/wedge` and `Why now`/`North star` sections already answer
    audience + purpose — `init`'s own interview then only needs to ask about voice, which
    `vision.md` doesn't cover); `document` gets `project.md`'s `stack:` list alongside its normal
    codebase scan, so it cross-checks generated tokens against the confirmed tech list instead of
    only inferring library names from raw code.
  - `shape` (the surface brief) + `design-taste-frontend`'s **design read** (page kind, vibe words,
    audience, brand assets, quiet constraints — Section 0, the actual input the dials are derived
    from, not a step to skip) — **both grounded in the epic's own `scope.md`** (the real goal), not
    just this page's narrow ticket text. The epic's context is already fetched by Step 3's
    `log-decision` walk-up on every ticket — cite it explicitly here as an input, not just through
    the sibling-ticket/story-level cache check below. `shape`'s audience/outcome/job fields and the
    design read's audience/vibe-word inference should draw on that already-fetched epic context
    explicitly.

    **Labeled, not merged**: when dispatched to `design-taste-frontend`'s design read and to
    `shape`, the epic goal travels as its own tagged block — `Epic goal (from epic_goal_ref):
    <content>` — kept distinct from the ticket's own AC and from `PROJECT.md`'s stack, not folded
    into one undifferentiated context paragraph. This is what lets B4b's reviewer (or a human
    later) trace a design-read claim ("audience: technical buyers") back to its actual source
    instead of taking it on faith.

    Its three dials (VARIANCE/MOTION/DENSITY) are scoped to the **page/surface**, not the ticket —
    every layout, motion, and density decision on a page is gated by one consistent dial-set for
    that whole page. Since a page in this pipeline can span several sibling tickets (e.g. a
    Story's grouped card-section tickets), redeciding this per ticket risks two sibling tickets
    picking inconsistent dial values for the same page. **Fix**: query `log-decision` at the Story
    level first — if a prior ticket for this same page already logged a surface brief + dial-set,
    reuse it verbatim; only the *first* ticket to touch a given page actually runs `shape` + dials
    for real, then writes the result to the Story's `scoping-calls.md` via `log-decision` (write)
    so every sibling ticket reads instead of redecides.
  - **No cap on `overdrive`** — treated like any other Enhance-category command, a per-task
    judgment call, not a tracked, capped resource. `impeccable`'s own docs support a
    `[target]`-scoped `overdrive` per component/section, so a page can have more than one flourish
    moment if the task calls for it.
  - **Mandatory contrast check, any time `init`/`document`/`extract` above actually creates or
    changes `DESIGN.md` or `.impeccable/design.json`.** For every text/background color pairing
    declared in `DESIGN.md`'s `components` frontmatter (and the sidecar's matching CSS), compute
    the real WCAG 2.1 relative-luminance contrast ratio — normal text needs ≥4.5:1, large text
    (≥24px, or ≥19px bold) and UI components/graphical objects need ≥3:1. A failing pairing is
    never shipped as-is: replace it with a token the project **already declares** — `DESIGN.md`'s
    own `colors:` block or `docs/design-system.md`/the project's real CSS custom properties — never
    an invented value. Log the swap via `log-decision` (ticket or story level, whichever this
    surface's dial-set was logged at) citing both numbers (failing ratio → passing ratio). This is
    the tool's own generation step; it does not replace B4b's independent check below — a pairing
    can pass here and still get broken by how an implementer actually wires it up.
  - `polish` (final alignment pass) is deferred to B6, not repeated per task.
  - **`extract`** (consolidates repeated patterns *actually built* across tickets into
    `DESIGN.md` — not related to `PROJECT.md.stack`, which stays the separate "what libraries are
    allowed" source of truth used elsewhere) doesn't belong here at all — it's a periodic,
    non-blocking suggestion at B9 once several tickets have landed. `extract` writes its
    consolidated result into `DESIGN.md`; it doesn't affect the ticket that triggered it, but the
    *next* ticket's B3.5 step 1 (`impeccable context`) reads that refreshed `DESIGN.md`, so a later
    page's implementer can reuse the newly consolidated component instead of rebuilding a similar
    pattern from scratch.
- **B4 — Per-task loop**, in dependency order:
  - **B4a) Dispatch a fresh implementer subagent** (via `Task`, never this session) with the task's
    AC, ledger context pointers, and the **scope-conditioned content fork**:
    - `frontend` → B3.5's fixed surface brief (design read + dials), `design-taste-frontend`'s own
      concrete anti-slop rules (anti-center-bias, content-density limits, marquee/decorative-dot
      caps, copy-register consistency, its component-library picks per page kind), Step 8's
      `ecc-frontend-patterns.md`, `impeccable`'s own Commands table + `routing.md`, and this task's
      own text (AC/description — it already says what kind of work this is). **The implementer
      picks the fitting command(s) itself**, the same way `impeccable`'s own routing says to — a
      first-structure task runs the base build; an animation-only task runs `animate` (or
      `overdrive`, per B3.5, with no cap) directly, skipping base build entirely; a
      spacing/hierarchy fix runs `layout`; copy issues run `clarify`; and so on. Nothing here
      hardcodes "always build then motion." **The one non-negotiable:** self-QA (`critique` +
      `audit`) runs before reporting done, regardless of which build command(s) were used — a
      completion discipline, not a stylistic pick, that feeds directly into B4b's review.
      **When the ledger's `reference_urls` is set**, the brief includes those exact URL(s)
      explicitly and instructs the implementer to actually browse each one itself (whatever
      browser-automation tool the harness provides) before/while building — not just work from
      `research-reference`'s prose summary of it. A written mechanism description is not the same
      input as looking at the real thing, and the implementer is the one making the concrete
      visual calls (spacing, proportions, type treatment) that a summary can't fully specify. The
      brief also carries the research's own `Build-fidelity checklist` in full, verbatim — **every
      line on it must land in the build.** If the implementer judges a line doesn't fit (e.g. an
      interactive mechanism it decides is out of scope for this ticket), that's not a silent
      simplification it makes on its own: it logs a Ruling naming exactly which line it's skipping
      and why, and flags it in its own done-report so B4b's reviewer (and the user, if it survives
      to B9) sees the omission called out rather than discovering it missing on their own.
    - `backend` → **first, a reality check, run per task and never cached**, since schema/routes
      can change between tickets unlike frontend's page brief:
      - **DB**: read the project's real Flyway migration files (not a schema doc) to build the
        current table/column inventory, cross-reference against what this task needs to
        persist/query, and decide — explicitly, citing the migration file(s) — what's reused vs.
        what needs a new migration.
      - **API + contract**: grep the project's real route handlers (`app/api/**/route.ts`) for
        endpoints already covering this task's resource, and decide reuse (extend) vs. new (add a
        route) explicitly. For this single-repo Next.js stack, where frontend and backend share
        one TypeScript build rather than separate services, the canonical contract artifact is a
        shared type in `features/shared/`, not OpenAPI — grep for it first; if missing, the ticket
        touching this resource *first* (per `senior-engineer`'s no-forward-dependencies ordering)
        creates it, and the sibling ticket imports it rather than re-declaring its own shape. Drift
        is then caught for free by `pnpm typecheck` (already run in B4b/B6) the moment either side
        diverges from the shared type — no separate contract-check process needed.
      - Then: Step 8's `ecc-backend-patterns.md`, `postgres-patterns`/`database-migrations`
        (schema/index quality, migration safety checklist) for any new table/column, `api-design`
        for any new route, plus TDD content (Iron Law: no code without a failing test first;
        mandatory watch-it-fail; anti-rationalization table; delete-means-delete) and the
        criterion-ID bridge: every backend test file/block gets a `describe`/`it` name or comment
        matching `AC-<N>: <criterion text>`, so the task reviewer in B4b can cross-check each
        `AC-N` directly against `spec.md`'s real criteria — an implemented AC with no matching tag
        fails; a tag citing a nonexistent AC also fails.
      - If this task involves an external call, cache, or queue, name the retry/cache/backoff
        pattern being used and why, citing `ecc-backend-patterns.md` — a conditional prompt carried
        in the brief, not a mandatory gate run on every task.
    - `shared` → both, sectioned by which half of the task is frontend vs. backend. **Backend runs
      first, always** — it's the producer of the data shape (the DB/API reality check above
      decides what actually exists), frontend is the consumer. Backend's TDD work creates the real
      shared type; frontend's task then imports it rather than guessing at a shape that might not
      match what backend actually built.
  - **B4b) Task reviewer subagent** (fresh, via `Task`, separate from the implementer) — spec
    compliance AND code quality, both required, both grounded in this ticket's *real* files, not a
    generic rubric. Dispatch it with, as its explicit "global constraints" block:
    - the ticket's own AC (Given/When/Then, from Step 2/4) and its `T0xx` task text from
      `tasks.md`;
    - Step 5's resolved doc-map rows for this task specifically — `design-system.md`'s tokens for
      a frontend task, `docs/standards/backend.md`/`auth.md`/`database.md` for a backend one;
    - Step 7's coding-standards docs (`docs/standards/frontend.md`/`backend.md`/`testing.md`) —
      the same rules the project's own Standards Review CI bot scores (file/folder naming,
      `export const` vs `export function`, relative-vs-`@/` import rules, spec-file coverage);
    - for backend: confirm the criterion-ID tag exists and that the test genuinely failed then
      passed — re-run it, don't take the implementer's word for it; and, for any task touching
      DB/query code, `database-reviewer`'s checklist (indexed FK/WHERE/JOIN columns, correct types,
      RLS-equivalent access checks, no `SELECT *`, no N+1, short transactions).
    - **Fast, task-scoped self-QA**: lint + typecheck + this task's own affected tests, run for
      real with the output read — not the full suite (that's B6's job).
    - **Contrast check, mandatory for every `frontend`/`shared` task, unconditional** (unlike the
      visual-comparison check below, this does **not** require `reference_urls` to be set — it
      applies to any UI built at all, referenced or not). For every text/background color pairing
      actually rendered in this task's diff — grep the built component's classNames/inline styles
      for the real `text-*`/`bg-*` token pairs it uses, don't just re-read `DESIGN.md`'s intent —
      compute the WCAG 2.1 contrast ratio against the same thresholds as B3.5 (≥4.5:1 normal text,
      ≥3:1 large text/UI components). Read the actual live-rendered result (start/reuse the dev
      server, screenshot or read computed styles), not just the source — B3.5's design-time check
      only validates what `DESIGN.md` *declares*; this validates what actually *renders*, since a
      correct token declaration can still get overridden, mis-cascaded, or dropped by the real
      build (exactly the class of bug that slips through a source-only read). A failing pairing
      fails this review exactly like a missed AC: the fix is to swap in a token the project already
      declares — never invent a new color — re-verify the number for real, and re-screenshot to
      confirm the fix actually rendered, not just that the source now says the right class name.
    - **Visual comparison, when the ledger's `reference_urls` is set** (frontend/shared UI work
      built from a `research-reference` spike): browse each reference URL and the actual built
      result yourself — start or reuse the project's dev server and navigate to wherever the new
      component actually renders (a Storybook story if one exists for it, otherwise a minimal
      temporary render). This check is **checklist-driven, not a vibe check**: pull the research's
      own `Build-fidelity checklist` (embedded in the ticket, see A5) and go down it line by line —
      every measured value and every named interactive mechanism gets its own explicit
      match/mismatch/missing verdict. A line the implementer's brief flagged as a deliberately
      skipped Ruling (per B4a) still gets recorded here as a known, disclosed gap, not silently
      passed. "Looks similar overall" is not an acceptable substitute for going through the actual
      list.

      **Screenshot at true resolution, not a scaled-down thumbnail** — take the screenshot at the
      real viewport size the checklist's numbers were measured at (e.g. 1440px desktop), not
      whatever a tool's default preview size happens to be; a compressed thumbnail hides exactly
      the edge-padding and small-spacing problems this check exists to catch. **Look at the image
      first.** A DOM query (`getBoundingClientRect`, computed styles) is only ever used *after* the
      screenshot already shows something looks right, to confirm the exact number matches the
      checklist line — it is never a substitute for looking, and "the element is technically
      present in the DOM with the right attributes" is not a passing visual verdict on its own.

      This whole check is a required part of the same PASS/FAIL verdict as the code/spec checks
      above, not a separate soft note — a real visual mismatch, or a missing checklist item, fails
      the review exactly like a missed AC does.
  - **B4c) Fix loop, max 5 rounds**: rounds 1–3 resume the same implementer; rounds 4–5 dispatch a
    fresh implementer on a more-capable tier. Round 5 exhausted → escalate to the user via
    `AskUserQuestion`, logged as a Ruling — never silently forced through.
    - **Research-gap exception**: if a finding traces back to the ticket's *embedded research
      findings themselves* being wrong or incomplete in practice (a value that doesn't hold, a
      technique that doesn't actually work as described, a recommended field that turns out not to
      fit) — not an implementer mistake — don't resume the implementer to guess again with no new
      information. Dispatch whichever research skill produced the original findings
      (`research-reference` or `research-ux`, per the ticket's own research-kind label) once more,
      scoped narrowly to just that gap (not a full re-investigation), re-run A3's verification on
      the correction, update the ticket's embedded findings, then resume the implementer with the
      correction. Counts as one fix round, same cap. Only applies to a ticket with real research
      provenance behind it (Branch A ran for it earlier).
    - **A visual-comparison failure is not a separate mechanism** — it resumes the implementer
      (or escalates on round 4-5) exactly like any other B4b finding, same 5-round cap, same
      round-5 escalation to the user. The one exception: if the mismatch actually traces back to
      `research-reference`'s own write-up being too thin to build from (not an implementer
      execution mistake), treat it as a Research-gap exception per the bullet above instead.
  - **B4d) Ledger update**: task id, model used, round count, reviewer verdict.
- **B5** — Repeat B4 for every task in the subset.
- **B6 — Final whole-branch review**, most-capable model, over the whole diff: cross-task
  coherence plus the self-QA content fork run at repo scope — real commands run fresh, real output
  read, no "should pass" claims.
- **B7 — Fix loop** for B6 findings, same mechanics as B4c.
- **B8 — Log non-obvious deviations** to `log-decision` — whenever a non-obvious scoping or
  implementation call was actually made during the build that isn't already captured elsewhere,
  the same judgment-based trigger every other pipeline skill already uses for this.
- **B9 — Finish.** Run `finishing-a-development-branch`'s real flow: verify tests are green, then
  present its exact menu — **(1) merge locally, (2) push + open a PR, (3) keep as-is** — and wait
  for the user's choice; don't assume PR-by-default even though that's the most likely pick. Update
  the ledger with the outcome; enrich the ticket's `log-decision` entry; transition the Jira ticket
  (e.g. to "In Review") via the same direct REST + `.env` auth as Step 2/A6. **Frontend/shared
  tickets only:** after several tickets have landed for
  this project, suggest (never force) running `impeccable extract` to consolidate repeated built
  patterns into `DESIGN.md`.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Self-QA runs at two tiers, not one flat check: B4b's fast per-task subset (lint, typecheck, this
  task's own affected tests) against the resolved standards docs, and B6's full suite plus
  standards-score across the whole ticket diff. Applies to frontend and backend scope tickets
  alike. `resolve-pr-comments`/Greptile remains a third, later gate on the opened PR — it isn't
  replaced by either of these.
- Every backend test file/block is tagged `AC-<N>: <criterion text>` against `spec.md`'s real
  criteria — an implemented AC with no matching tag fails review, and a tag citing a nonexistent
  AC fails it too.
- Visual comparison against a `research-reference`-sourced reference lives inside B4b's existing
  task reviewer (one dispatch judges code/spec compliance and visual similarity together), not a
  separate dedicated subagent — and a real mismatch is blocking, following B4c's existing 5-round
  fix loop rather than a softer non-blocking flag. `research-reference` itself doesn't save a
  screenshot during its own investigation (Branch A) — it passes the live URL forward via its
  `Reference source(s):` line, and B4a/B4b both re-visit the real site directly rather than working
  from a potentially-stale capture.
- The visual comparison is graded against `research-reference`'s own `Build-fidelity checklist`
  item by item, not a holistic "looks about right" impression — and it's a screenshot-first check:
  look at a true-resolution image before running any DOM/computed-style query, since a query only
  confirms a number a screenshot already flagged, never substitutes for looking. A checklist item
  the implementer explicitly chose to skip (logged as a Ruling, per B4a) is recorded as a disclosed
  gap here, not silently waved through.
- A `SKILL.md` process change mid-ticket is not something an interrupted-session resume (B2) covers
  — re-run the affected task(s) through a fresh B4a/B4b under the *updated* steps rather than the
  orchestrating session hand-patching the result outside the loop.
- Backend resilience/perf choices are either already required by the AC, or logged as an
  architectural call via `log-decision` — there's no mandatory ritual gate on every task beyond
  the one conditional retry/cache/backoff prompt carried in the backend brief.
- Backend tasks have no B3.5-equivalent per-page framing step — backend consistency comes from the
  coding-standards and doc-map context already loaded at Steps 5/7, not something inferred fresh
  per ticket.
- The ledger is committed at `.build/<ticket-key>/progress.md`, not worktree-scratch, and stays in
  git history after merge — no cleanup step. Everything else in the workspace (briefs, review
  packages, reports) stays worktree-local scratch. Re-invoking `build` on a ticket whose ledger
  already exists is a resume, not a fresh run — pick up at the first task with no `complete` line
  rather than re-dispatching finished tasks.
- Model tier is chosen per task, not read off a fixed table — boilerplate/CRUD work → the cheapest
  tier, typical feature logic → the standard tier, security-sensitive/tricky/escalated work → the
  most-capable tier — refined against real tickets over time, and recorded per task in the ledger.
- No cap on `impeccable`'s `overdrive` command — it's a per-task judgment call like any other
  Enhance-category command, not a tracked/capped resource.
- The VARIANCE/MOTION/DENSITY dial-set is decided once per page, by the first ticket to touch that
  page, not re-decided per ticket — later sibling tickets for the same page reuse the Story-level
  `log-decision` entry verbatim.
- Both B4c (per task) and B7 (final review) fix loops cap at 5 rounds, escalating to the user via
  `AskUserQuestion` on exhaustion rather than forcing a merge through.
- Only four things pause Branch B for the user: an irreversible/destructive op, a
  security-sensitive action, a side effect outside the worktree (push/merge/publish to shared
  state), or a plan so broken every path forward is a guess. Everything else is a judgment call,
  logged as a Ruling.
- Ticket task-subset isolation (Step 4) and baseline-link loading (Step 6) prefer the ticket's own
  embedded `T0xx` list / baseline link when present, falling back to AC-text matching against
  `tasks.md`, or deriving the baseline path directly from the capability/component name, when the
  ticket predates those embeds.
- `build` supersedes both `build-frontend` and `build-backend` outright, for every case, not just
  ticket-driven work — neither is invoked by this skill or offered as a fallback for anything.
- **WCAG contrast is checked twice, at two different times, and neither substitutes for the
  other**: once at B3.5 whenever `DESIGN.md`/`.impeccable/design.json` is actually created or
  changed (validates what the design system *declares*), and again, unconditionally, at every
  B4b review of a `frontend`/`shared` task regardless of whether `reference_urls` is set (validates
  what the real diff *renders*). A failing pairing is fixed by swapping in a token the project
  already declares — never a new/invented color — logged via `log-decision` with the before/after
  ratio. This is a blocking failure at both points, same as a missed AC or a visual mismatch.
