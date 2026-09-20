---
name: build
description: >
  Unified builder for one Jira ticket, or — given a Story key — every ticket under it in one
  batched run. Reads the shared context chain once per run, not once per ticket, then for a Story
  runs every spike ticket first (since implementation tickets may depend on their findings),
  followed by every task/bug ticket through one continuous implementer subagent and one continuous
  reviewer subagent, each resumed task-to-task rather than re-spawned, so the worktree, branch,
  ledger, and already-loaded context carry across the whole Story instead of being re-paid per
  ticket. A single ticket key still runs solo, the same mechanics, just with a batch of one. Forks
  the scope-conditioned implementer/reviewer briefs by the ticket's `scope` label
  (frontend/backend/shared) via file-path pointer, never an inlined copy — not the control flow
  itself. Supersedes build-frontend and build-backend outright — neither stays in use.
argument-hint: '<ticket-key-or-story-key>'
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
  - SendMessage
  - Skill
  - AskUserQuestion
  - EnterWorktree
  - ExitWorktree
license: MIT
---

# /build

Picks up one Jira ticket — or, given a Story key, every ticket under it — and carries it through
to done. Reads the shared context chain once per run (Steps 1–9), the same consult every branch
below shares, then runs every `spike` ticket in the batch first (Branch A: dispatches the matching
research skill, independently verifies its findings, embeds them — nothing gets trusted
unverified), because a `task`/`bug` ticket may be blocked by one. Only once every spike in the
batch is done does Branch B start: `subagent-driven-development`'s engine — one worktree, one
ledger, one continuous implementer subagent and one continuous reviewer subagent (each resumed
per task via `SendMessage`, not re-spawned), a fix loop, a final whole-branch review, then
finish — over the batch's whole task/bug set, forking only the implementer/reviewer's brief
*content* by each ticket's `scope` label (`frontend`/`backend`/`shared`), never the control flow
itself. Supersedes `build-frontend` and `build-backend` outright — neither stays in use, for any
case.

**Why batching exists at all:** Steps 1–9 below — reading the ticket chain, the feature plan, doc
routing, mined baselines, coding-standards docs, this repo's own pattern reference — cost real
tokens to load. Run per ticket, that cost is paid once per ticket even when three sibling tickets
under the same Story share the same doc set, the same page, often the same files. Run once per
Story, it's paid once. The same logic applies one level down: a fresh implementer subagent
re-spawned per task re-pays whatever it needs to read to do the task; the same implementer,
resumed via `SendMessage` for the next task, already has it. Batching is that idea applied at both
levels — the orchestrating session's own context, and each dispatched subagent's.

## Step 1 — Read `PROJECT.md`

Read the project-root stub, then `<vault_path>/<company-slug>/project.md` for the rest — stack,
`doc_map_source`, the `coding_standards` fallback list, and ticketing config. Same first move every
skill in this pipeline already makes.

## Step 2 — Read the target and detect its shape

Fetch the given key via a direct REST call — `GET https://<ticketing.jira_site_url>/rest/api/3/issue/<key>`,
HTTP Basic Auth with `<ticketing.jira_email>:$JIRA_API_TOKEN` (the token read from this project's
own `.env`, matching `ticketing.auth_env`; the email/site are non-secret and come straight from
`PROJECT.md`) — not through an MCP-based `jira-integration` tool. Never hardcode the token, echo
it, or let it appear in a logged command string; read it via shell env expansion only.

Check `issuetype`:

- **Story** → this is a **Story-batch run**. Also query `searchJiraIssuesUsingJql`-equivalent REST
  (`GET .../search?jql=parent=<key> ORDER BY key`, which returns each child's `status` for free) for
  every child Subtask, and pull each one's `type`/`scope` labels, `status`, AC body, embedded `T0xx`
  task-ID list, and blocker links the same way.

  **Filter to `status: To Do` only — the batch touches nothing else.** A child ticket already `In
  Progress`, `In Review`, `Blocked`, or `Done` is excluded from the batch entirely: not re-built,
  not re-researched, not re-reviewed, not transitioned. It likely already has its own worktree,
  branch, or PR in flight from a separate run (solo or otherwise) — this batch has no business
  touching that state. This is a status check, not a merge/reconciliation problem: don't try to
  detect or reconcile another run's branch, just leave any non-`To Do` ticket alone entirely and
  report which ones were skipped and why, once, before Step 4 orders what's left. The one exception:
  a `To Do` ticket in the batch that's blocked by an excluded (already-`Done`) ticket still reads
  that blocker's embedded findings normally per Step 3 — reading a finished sibling's output isn't
  touching its ticket. A `To Do` ticket blocked by a non-`Done` excluded ticket (`In Progress`/`In
  Review`/`Blocked`) has no findings to read yet either — leave it out of this batch too, same as
  its blocker, rather than guessing at unfinished work.

  Everything from here on operates over **the batch** — the filtered `To Do` subset of the Story's
  children, not the Story's full child list.
- **Subtask / Task / Bug** → this is a **solo run**. The batch is that one ticket alone. Every step
  below still applies; a batch of one just means Step 4's ordering and the cross-ticket sharing in
  Steps 5–9 and Branch B are no-ops.

## Step 3 — Query `log-decision`, once for the batch

Call it in query mode with `{epic key, story key or null, ticket key}` — the Story's own key in a
batch run, or the lone ticket's in a solo run. This auto-walks ticket file → story
`scoping-calls.md` → epic `scoping-calls.md`/`scope.md` → company `scoping-calls.md` **once**;
every ticket in the batch shares the same epic/story ancestry, so this never repeats per ticket.
Existing entries are a resume, not something to re-derive.

Also resolve each ticket's own blocker, if it carries a "blocked by" link (per `senior-engineer`'s
blocker-link step) — a blocker is a sibling in the hierarchy, not an ancestor, so the walk-up above
never surfaces it on its own. **Check the blocking ticket's own embedded description first** —
once a spike passes A5 its write-up is embedded directly in the ticket it blocks (this pipeline's
"embed, don't just link" convention), and Step 2 already fetched that description in full. Only
fall back to reading the blocker's separate vault file when the ticket predates that embed. Extract
**two distinct named pieces**, not one — a caller two steps downstream (B4a/B4b, resumed hours
later via `SendMessage`) needs both without re-fetching the blocker's ticket description itself:
- `reference_urls`: the `Reference source(s):` line (see `research-reference/SKILL.md` Step 7).
- `build_fidelity_checklist`: the full, literal text of the spike's `Build-fidelity checklist`
  section — every measured value and every named interactive mechanism, verbatim. This is the
  actual thing Branch B builds and reviews against (see B4a/B4b below) — `reference_urls` alone is
  not enough to carry forward, since the whole point of the checklist is that nobody downstream
  should need to go back to the live site to reconstruct it.

## Step 4 — Order the batch

Build the dependency graph from every ticket's blocker links across the batch. **A spike ticket
runs before anything it blocks** — this is why Step 10 runs every spike in the batch before Branch
B starts on anything. Among tickets with no spike blocking them, task/bug tickets keep
`senior-engineer`'s own planned sequence (already forward-dependency-free per that skill's Step
7). A solo run's batch is one ticket, so this step is a no-op there.

## Step 5 — Load the feature plan

Read `specs/<feature>/{spec.md,plan.md,tasks.md}` **once for the batch**, then isolate each
ticket's own `T0xx` subset from that same read. Prefer each ticket's own embedded task-ID list
from Step 2 when present; when it isn't, isolate the subset by matching that ticket's AC text
against `tasks.md` instead.

## Step 6 — Resolve documentation routing

Read `doc_map_source` (the target repo's own `AGENTS.md#Documentation map`), filtered to what the
**batch's** task subsets actually touch: the union of `scope` labels across every ticket in the
batch as the coarse filter, then file-path/keyword match per task. Never load every backend doc
for a batch that only touches one route.

## Step 7 — Load mined baselines, if any

Load `<vault_path>/<company-slug>/openspec/specs/<capability>/spec.md` and/or
`<vault_path>/<company-slug>/openspec/components/<component-name>/interface.md` for whichever
apply across the batch. Prefer each ticket's own embedded baseline link when present; when it
isn't, derive the path directly from the capability/component name named in `spec.md`.

## Step 8 — Load coding-standards docs

Load more `doc_map_source` rows if set; otherwise load `PROJECT.md`'s `coding_standards` fallback
list in full. Once for the batch.

## Step 9 — Load this repo's own pattern reference

Load, filtered by the union of `scope` labels across the batch:

- `frontend`/`shared` present anywhere in the batch → `skills/build/references/ecc-frontend-patterns.md`
- `backend`/`shared` present anywhere in the batch → `skills/build/references/ecc-backend-patterns.md`

This is a different source from Steps 6/8 — this repo's own pre-filtered generic patterns, not the
target repo's own docs. **These get handed to Branch B's implementer as a file path, not pasted
content** — see B4a.

## Step 10 — Run every spike in the batch first

For each `spike`-type ticket in the batch, in Step 4's order, run Branch A to completion (through
A6, closing the ticket) before Step 11 starts. A solo run over a spike ticket runs Branch A alone
and stops there — Branch A never touches Branch B.

### Branch A — spike ticket (produces no code)

**A0 — Dispatch by research-kind label.** A spike ticket carries `research-mechanism` or
`research-ux` (assigned by `senior-engineer`) — dispatch `research-reference` for the former,
`research-ux` for the latter. A design-spike ticket (no idea existed yet at all) carries neither
and isn't a spike this branch handles the same way — it's a request for a human design decision,
not a research dispatch.

- **A1** — Dispatch the chosen skill (via `Skill`) with the reference/subject, "what it needs to
  become"/"why it matters" from the ticket, plus Steps 1–9's gathered context. `build` is
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

## Step 11 — Run every task/bug ticket in the batch

Everything below runs **once**, over the batch's whole task/bug set in Step 4's order — it is not
re-entered per ticket the way Branch A is per spike. A solo run's batch is one ticket, so this is
just that ticket's own task subset.

### Branch B — task/bug tickets (`subagent-driven-development` wraps the whole batch)

**Autonomy contract** (stated once, applies throughout): rulings-not-stalls. Only four things stop
this branch for the user: an irreversible/destructive op, a security-sensitive action, a side
effect outside the worktree (push/merge/publish to shared state), or a plan so broken every path
forward is a guess. Everything else is a judgment call, logged as `Ruling: <what> — <why> — <cost
if wrong>`.

- **B1 — Worktree setup, once for the whole batch.** Per `using-git-worktrees`: detect existing
  isolation first (`GIT_DIR`/`GIT_COMMON`); if none, use this harness's own native worktree tool
  (`EnterWorktree`/`ExitWorktree`) rather than raw `git worktree add`. Get consent before creating
  one if no preference is already on record. **One worktree, one branch, for the entire batch** —
  every child ticket's tasks commit to the same branch, so a Story-batch run produces exactly one
  PR at B9, not one per ticket (a solo run's branch still covers just its one ticket, same as
  before).
- **B2 — Ledger init, check for a resume first.** The ledger lives at the *committed* path
  `.build/<story-key-or-ticket-key>/progress.md`, not worktree-scratch — check whether it already
  exists before creating it. If it does (a prior session was interrupted, or this batch is being
  picked back up), read it and resume at the first task without a `complete` line — don't
  re-dispatch tasks the ledger already shows done. Only on a genuine first run: seed it fresh with
  the batch's identity (Story key, or the lone ticket key), the Step 5 task subsets **across every
  ticket in the batch, in Step 4's order, each entry tagged with its own ticket key**, the scope
  label(s), and *pointers* to Steps 1–9's gathered context (not copies) — so a resumed session
  reconstructs context from the ledger alone. The epic's `scope.md` entry (from Step 3's walk-up)
  gets its own named pointer — `epic_goal_ref: <path to the epic's scope.md>` — not folded
  anonymously into a generic context bucket, since B3.5 needs to cite it specifically. **Each
  ticket's own `reference_urls: <url1>, <url2>, ...` and `build_fidelity_checklist: <verbatim
  text>`** get the same named-pointer treatment when Step 3 extracted them — omit both fields
  entirely for a ticket with no such provenance, rather than writing them empty; B4a and B4b both
  key off `reference_urls`' presence per-task to decide whether a visual-comparison pass applies to
  that task, and read `build_fidelity_checklist` directly from the ledger rather than re-fetching
  the blocking ticket's description each time they're resumed.

  **A resume is for picking up an interrupted batch, not for patching around a process change
  mid-flight.** If `build`'s own steps changed (a `SKILL.md` edit) after this batch's B4 loop
  already ran once, that is not a resume — re-dispatch a fresh B4a implementer and a fresh B4b
  reviewer through the *updated* steps for the affected task(s), rather than the orchestrating
  session patching the result by hand outside the loop.
- **B3 — Model tiering**, recorded per task in the ledger before dispatching — never inherited
  silently from the session default. Choose per task, not off a fixed table: boilerplate/simple
  CRUD work → the cheapest tier; typical feature logic → the standard tier; security-sensitive,
  tricky, or already-escalated work → the most-capable tier. Treat this as a judgment call to
  refine against real tickets, not a fixed lookup table.
- **B3.5 — Frontend surface framing, once per batch, not per ticket (frontend/shared scope only).**
  A Story in this pipeline already corresponds to one page/surface (its sibling tickets are that
  page's sections), so this now runs once at the start of the batch's Branch B phase, directly —
  no per-ticket cache round-trip needed within a single run.
  - Run `impeccable context` once, at the start of this step. It loads whatever `PRODUCT.md`/
    `DESIGN.md`/surface-brief state already exists and returns directives saying what's missing or
    stale — this is the self-check already built into `impeccable` itself; `build` doesn't
    hand-roll an existence or hash check of its own. Follow those directives as-is: if it says
    `PRODUCT.md` is missing, that directive triggers `init` (one-time interview for
    audience/purpose/voice); if it flags `DESIGN.md` missing/stale against the real theme/code,
    that triggers `document` (regenerates it from what's actually built). Neither runs unless
    `context` itself says to. When either does fire, hand over extra context in the same dispatch
    rather than letting them start from a blank interview/scan: `init` gets `vision.md`'s content
    alongside it; `document` gets `project.md`'s `stack:` list alongside its normal codebase scan.
  - `shape` (the surface brief) + `design-taste-frontend`'s **design read** (page kind, vibe words,
    audience, brand assets, quiet constraints), **both grounded in the epic's own `scope.md`**
    (the real goal), not just the batch's narrow ticket text. Already fetched by Step 3's
    `log-decision` walk-up — cite it explicitly here, tagged: `Epic goal (from epic_goal_ref):
    <content>`, kept distinct from any one ticket's own AC and from `PROJECT.md`'s stack.
    Its three dials (VARIANCE/MOTION/DENSITY) are scoped to the **page/surface** — since that's now
    exactly what a Story-batch run *is*, this runs for real exactly once per batch and every task
    in Branch B below shares the same dial-set, with nothing left to redecide. (A solo run over one
    ticket that turns out to share a page with tickets outside this batch still checks
    `log-decision` at the Story level first, the same way, in case a sibling ticket outside this
    run already logged it.) The result is written to the Story's `scoping-calls.md` via
    `log-decision` (write) either way, so a *later, separate* run touching the same page can reuse
    it.
  - **No cap on `overdrive`** — a per-task judgment call, not a tracked, capped resource.
    `impeccable`'s own docs support a `[target]`-scoped `overdrive` per component/section, so a
    page can have more than one flourish moment if a task calls for it.
  - **Mandatory contrast check, any time `init`/`document`/`extract` above actually creates or
    changes `DESIGN.md` or `.impeccable/design.json`.** For every text/background color pairing
    declared in `DESIGN.md`'s `components` frontmatter (and the sidecar's matching CSS), compute
    the real WCAG 2.1 relative-luminance contrast ratio — normal text needs ≥4.5:1, large text
    (≥24px, or ≥19px bold) and UI components/graphical objects need ≥3:1. A failing pairing is
    never shipped as-is: replace it with a token the project **already declares** — `DESIGN.md`'s
    own `colors:` block or `docs/design-system.md`/the project's real CSS custom properties — never
    an invented value. Log the swap via `log-decision` citing both numbers. This does not replace
    B4b's independent check below — a pairing can pass here and still get broken by how an
    implementer actually wires it up.
  - `polish` (final alignment pass) is deferred to B6, not repeated per task.
  - **`extract`** doesn't belong here at all — it's a periodic, non-blocking suggestion at B9 once
    several tickets have landed.
- **B4 — One continuous implementer, one continuous reviewer, resumed task-to-task, not
  re-spawned:**

  - **B4a) Spawn ONE implementer subagent for the whole batch, once, before the task loop
    starts** (via `Task`). Its opening brief carries what used to be re-pasted into every
    per-task dispatch — now as **file paths, not inlined content**, since the implementer already
    has `Read`/`Grep` and can pull exactly the slice a given task needs instead of absorbing the
    whole file up front:
    - `frontend` → the path to Step 9's `ecc-frontend-patterns.md`, `design-taste-frontend`'s own
      anti-slop rules doc, `impeccable`'s Commands table + `routing.md`, and B3.5's surface brief
      (design read + dial-set — this one is small enough to pass directly, it's the *output* of a
      dedicated step, not a doc to re-read).
    - `backend` → the path to Step 9's `ecc-backend-patterns.md`, `postgres-patterns`/
      `database-migrations`, `api-design`, and the TDD content (Iron Law, mandatory watch-it-fail,
      anti-rationalization table, delete-means-delete, the `AC-<N>` criterion-ID bridge).
    - `shared` → both.
    - The batch's ordered task queue (ticket key + task ID + AC per entry), and instruction to
      work it in order: **finish one task's self-QA (`critique` + `audit`, non-negotiable
      regardless of which build command(s) it used), report done, then stop and wait** — don't
      start the next task until told to. This is what lets the orchestrator interleave an
      independent reviewer pass between tasks while the implementer's own context (everything it
      already read, every decision it already made this batch) stays intact for the next one.

    Then, for each task in Step 4's order: `SendMessage` the same implementer (not a fresh `Task`
    — it's already alive) with just that task's own AC, ticket key, and, when set,
    `build_fidelity_checklist` in full, verbatim — it's small, and it must be checked line-by-line,
    not summarized.

    **Build strictly from the checklist. Do not browse `reference_urls` by default.** This
    reverses an earlier version of this step (live-tested against a real ticket): the checklist
    exists precisely so nobody downstream has to re-derive it, and a real run showed the
    implementer re-browsing the reference site found nothing the checklist didn't already have —
    same interactions, same coordinates, no new measurement, at real token cost (a full live-site
    sweep runs tens of thousands of tokens in screenshots alone). If a checklist line is genuinely
    ambiguous or contradictory once you're actually building against it — not "I'd feel more
    confident checking," a specific, nameable gap — that's a **Research-gap exception** (see B4c):
    stop and let the orchestrator dispatch `research-reference` back for that one narrow question,
    rather than opening a browser yourself. If the implementer judges a checklist line doesn't fit
    this task at all, it logs a Ruling naming exactly which line and why, and flags it in its own
    done-report — never a silent simplification either way.

    For a `backend` task, the reality check that used to run "per task, never cached" (reading the
    real Flyway migrations, grepping real route handlers for reuse-vs-new) still runs **per task**
    even inside this persistent session — schema/routes genuinely can change task to task within
    the same batch, so this specific check is never skipped, only everything *around* it (the
    static pattern docs) stops being re-read.

    For a `shared` task: backend half first, always — frontend's task then imports the real shared
    type backend's TDD work just created, rather than guessing at a shape.

  - **B4b) Spawn ONE reviewer subagent for the whole batch, once**, the same way — via `Task`,
    separate from the implementer, with its own opening brief as file-path pointers (doc-map rows,
    coding-standards docs, `database-reviewer`'s checklist for DB-touching tasks) rather than
    pasted content. Then, per task, `SendMessage` it that task's real diff plus:
    - the ticket's own AC (Given/When/Then) and `T0xx` task text;
    - **fast, task-scoped self-QA**: lint + typecheck + this task's own affected tests, run for
      real with the output read — not the full suite (that's B6's job);
    - for backend: confirm the criterion-ID tag exists and the test genuinely failed then passed —
      re-run it, don't take the implementer's word for it;
    - **contrast check, mandatory for every `frontend`/`shared` task, unconditional** (does not
      require `reference_urls`) — grep the built component's real classNames/inline styles for the
      actual token pairs used, compute WCAG 2.1 contrast against the live-rendered result (start
      or reuse the dev server, screenshot or read computed styles) — a source-only read isn't
      enough, since a correct token declaration can still get overridden or dropped by the real
      build;
    - **visual comparison, when that task's `build_fidelity_checklist` is set**: check the real
      built result against the checklist, line by line — screenshot the built component at true
      resolution first, DOM/computed-style queries only to confirm a number the screenshot already
      flagged, never as a substitute for looking. **Do not browse `reference_urls` by default** —
      same reasoning as B4a: the checklist is the distilled, already-verified record of what the
      reference does, and re-browsing it to check a checklist that was itself built from browsing
      it is circular spend, not independent verification. The reviewer's actual independence comes
      from checking the *built* component fresh, not from re-visiting the source a second (or
      third) time. A checklist line the implementer flagged as a deliberately skipped Ruling is
      recorded here as a known, disclosed gap, not silently passed. If the built result and the
      checklist seem to genuinely disagree in a way that isn't resolvable by re-reading either one
      — not routine due diligence — that's a Research-gap exception (see B4c), same as B4a: flag it
      rather than opening a browser to adjudicate it yourself.

    *Trade-off, on record*: because this reviewer persists across every task in the batch instead
    of being re-spawned fresh each time, it could start anchoring toward consistency with its own
    prior verdicts in this Story rather than judging each task independently — worth watching for
    if review quality on later tasks in a long batch starts looking rubber-stamped relative to the
    first few. If that shows up in practice, the fix is switching the reviewer back to a fresh
    dispatch per task while keeping this same pointer-based brief — the persistence choice and the
    pointer-vs-copy choice are independent and can be reverted separately.

  - **B4c) Fix loop, max 5 rounds:**
    - **Rounds 1–3**: `SendMessage` the *same* implementer with the reviewer's findings — it's
      already alive with the whole task's context loaded, so this is a plain continuation, not a
      reload.
    - **Rounds 4–5**: this harness fixes an agent's model tier at spawn time — there's no way to
      bump an already-running agent to a more-capable tier, only spawn a new one with a `model`
      override. A fresh spawn is unavoidable here, but it doesn't need Steps 1–9's full context
      again: hand the new higher-tier agent a **compact escalation package** — the reviewer's
      specific failure findings from rounds 1–3, the task's own AC/checklist, and the same file
      pointers B4a already used — not the original doc bundle. By round 4 the unknown is narrow
      (what's failing), not the whole ticket from zero. Once that one task resolves (pass, or
      round-5 exhaustion → escalate to the user via `AskUserQuestion`, logged as a Ruling), this
      escalated agent is discarded — the batch's original long-lived implementer resumes for the
      *next* task in the queue, not the escalated one.
    - **Research-gap exception**: if a finding traces back to the ticket's *embedded research
      findings themselves* being wrong or incomplete in practice — not an implementer mistake —
      don't resume the implementer to guess again with no new information. Dispatch whichever
      research skill produced the original findings once more, scoped narrowly to just that gap,
      re-run A3's verification on the correction, update the ticket's embedded findings, then
      resume the implementer with the correction. Counts as one fix round, same cap. Only applies
      to a ticket with real research provenance behind it (Branch A ran for it earlier).
    - **A visual-comparison failure is not a separate mechanism** — same 5-round cap, same
      round-5 escalation. The one exception: if the mismatch traces back to the research write-up
      itself being too thin to build from, treat it as a Research-gap exception instead.
  - **B4d) Ledger update**: task id, ticket key, model used, round count, reviewer verdict.
- **B5** — repeat B4's per-task send/review/fix cycle for every remaining task in the batch's
  ordered queue, driven by the orchestrating session, using the same two persistent subagents
  throughout — no re-spawn between tasks on the happy path.
- **B6 — Final whole-branch review, most-capable model, over the whole batch's diff** — every
  ticket's tasks together, since they share one branch/PR now: cross-task coherence plus the
  self-QA content fork run at repo scope — real commands run fresh, real output read, no "should
  pass" claims.
- **B7 — Fix loop** for B6 findings, same mechanics as B4c.
- **B8 — Log non-obvious deviations** to `log-decision` — whenever a non-obvious scoping or
  implementation call was actually made during the build that isn't already captured elsewhere.
  Naturally Story-scoped for a batch run.
- **B9 — Finish.** Run `finishing-a-development-branch`'s real flow: verify tests are green, then
  present its exact menu — **(1) merge locally, (2) push + open a PR, (3) keep as-is** — and wait
  for the user's choice. **One PR for the whole batch** — its description lists every child
  ticket key it resolves. Update the ledger with the outcome; enrich the ticket's (and, for a
  batch run, the Story's) `log-decision` entry; transition **every** ticket in the batch to "In
  Review" via the same direct REST + `.env` auth as Step 2/A6, looped over the set. **Frontend/shared
  batches only:** after several tickets have landed for this project, suggest (never force) running
  `impeccable extract` to consolidate repeated built patterns into `DESIGN.md`.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- **A Story key triggers a batch run; a Subtask/Task/Bug key triggers a solo run over a batch of
  one** — same steps either way, since every step is written in terms of "the batch." Detected
  from the fetched issue's own `issuetype` at Step 2, never guessed from the key's shape.
- **A Story-batch run only ever touches `status: To Do` children.** `In Progress`, `In Review`,
  `Blocked`, and `Done` tickets are excluded outright, no exceptions — they may have their own
  worktree/branch/PR already in flight from an earlier solo or batch run, and this run has no way
  to know what state that work is actually in. This is a plain status filter, not an attempt to
  detect, merge, or reconcile another run's branch — simpler and safer than trying to be clever
  about partial state. A `Done` blocker's embedded findings are still read normally (reading a
  finished sibling's output isn't touching its ticket); a non-`Done` excluded blocker's dependent
  `To Do` ticket is excluded too, since there's nothing finished yet to read.
- **One worktree, one branch, one PR per batch** — a Story-batch run's every child ticket commits
  to the same branch and ships as one PR; a solo run's branch still covers just its one ticket.
  Chosen because the same context-reuse argument for the implementer/reviewer applies to review
  overhead too — reviewing one coherent Story-sized PR beats reviewing N small ones that all touch
  the same page.
- **One continuous implementer and one continuous reviewer per batch, each resumed via
  `SendMessage` task-to-task, not re-spawned per task.** This is the main lever against context
  bloat: Steps 1–9's context, plus whatever static docs each subagent reads on its own, get paid
  once per batch instead of once per task. The reviewer's persistence is a deliberate trade-off
  against per-task independence (see B4b's note) — revisit that one specifically, independent of
  everything else here, if review quality drifts on longer batches.
- **Pointers over pasted content, everywhere except the Build-fidelity checklist.** Every doc that
  used to get read by the orchestrator and re-pasted into a subagent's dispatch prompt (pattern
  references, standards docs, doc-map rows, `impeccable` routing) is now handed over as a file
  path; the subagent reads or greps only what a given task actually needs. The Build-fidelity
  checklist stays embedded verbatim in every per-task message — it's small, and it must be checked
  line by line, not summarized or re-derived from a path.
- **Live-browsing `reference_urls` happens exactly once per reference, inside Branch A's research
  dispatch — never again in Branch B.** Earlier versions of this skill had `research-reference`
  browse live *and* B4a's implementer browse live *and* B4b's reviewer browse live — three full
  passes over the same site for one ticket. A real batch run (SCRUM-11, 2026-09-20) showed why this
  was pure waste: the implementer's live re-browse hit the same clicks at the same coordinates as
  the original research pass and surfaced nothing the checklist didn't already have, and a later
  task in the same batch never even opened the reference site despite being allowed to — it built
  entirely from the checklist. Each full live-site sweep runs tens of thousands of tokens in
  screenshots alone; three per ticket was the single largest cost driver in that run, well ahead of
  the per-task re-dispatch problem the rest of this batching redesign targets. The fix: `A1`'s
  research dispatch is the only place a browser opens for the reference site. B4a builds and B4b
  reviews strictly against `build_fidelity_checklist`. The **Research-gap exception** (B4c) is the
  one sanctioned way back to the live site — a specific, named gap the checklist can't answer,
  never a routine "let me double-check."
- **Tier escalation (B4c rounds 4–5) requires a fresh agent spawn — this harness has no
  running-agent model-tier upgrade** — but the fresh spawn gets a compact escalation package
  (specific failure + AC/checklist + pointers), not a full Steps 1–9 reload, and is discarded after
  its one task; the batch's original implementer resumes the queue afterward.
- Self-QA runs at two tiers, not one flat check: B4b's fast per-task subset (lint, typecheck, this
  task's own affected tests) against the resolved standards docs, and B6's full suite plus
  standards-score across the whole batch's diff. `resolve-pr-comments`/Greptile remains a third,
  later gate on the opened PR.
- Every backend test file/block is tagged `AC-<N>: <criterion text>` against `spec.md`'s real
  criteria — an implemented AC with no matching tag fails review, and a tag citing a nonexistent
  AC fails it too.
- Visual comparison against a `research-reference`-sourced reference lives inside B4b's existing
  reviewer (one subagent judges code/spec compliance and visual similarity together), not a
  separate dedicated subagent — and a real mismatch is blocking, following B4c's existing 5-round
  fix loop. `research-reference` (Branch A) is the only place that visits the live reference site;
  B4a and B4b both work from `build_fidelity_checklist` alone and do not re-visit it, per the
  live-browsing bullet above.
- The visual comparison is graded against `research-reference`'s own `Build-fidelity checklist`
  item by item, not a holistic "looks about right" impression — and it's a screenshot-first check.
  A checklist item the implementer explicitly chose to skip is recorded as a disclosed gap, not
  silently waved through.
- A `SKILL.md` process change mid-batch is not something an interrupted-session resume (B2) covers
  — re-run the affected task(s) through a fresh B4a/B4b send under the *updated* steps rather than
  the orchestrating session hand-patching the result outside the loop.
- Backend resilience/perf choices are either already required by the AC, or logged as an
  architectural call via `log-decision` — no mandatory ritual gate on every task beyond the one
  conditional retry/cache/backoff prompt carried in the backend brief.
- Backend tasks have no B3.5-equivalent per-page framing step — backend consistency comes from the
  coding-standards and doc-map context already loaded at Steps 6/8, not something inferred fresh
  per ticket.
- The ledger is committed at `.build/<story-key-or-ticket-key>/progress.md`, not worktree-scratch,
  and stays in git history after merge — no cleanup step. Everything else in the workspace
  (briefs, review packages, reports) stays worktree-local scratch. Re-invoking `build` on a batch
  whose ledger already exists is a resume, not a fresh run — pick up at the first task with no
  `complete` line rather than re-dispatching finished tasks.
- Model tier is chosen per task, not read off a fixed table — boilerplate/CRUD work → the cheapest
  tier, typical feature logic → the standard tier, security-sensitive/tricky/escalated work → the
  most-capable tier — refined against real tickets over time, and recorded per task in the ledger.
- No cap on `impeccable`'s `overdrive` command — a per-task judgment call like any other
  Enhance-category command, not a tracked/capped resource.
- The VARIANCE/MOTION/DENSITY dial-set is decided once per page — for a Story-batch run, that's
  once at the top of B3.5, directly, since a Story already *is* one page in this pipeline; a solo
  run over one ticket still checks `log-decision` at the Story level first in case a sibling ticket
  outside this run already logged it.
- Both B4c (per task) and B7 (final review) fix loops cap at 5 rounds, escalating to the user via
  `AskUserQuestion` on exhaustion rather than forcing a merge through.
- Only four things pause Branch B for the user: an irreversible/destructive op, a
  security-sensitive action, a side effect outside the worktree (push/merge/publish to shared
  state), or a plan so broken every path forward is a guess. Everything else is a judgment call,
  logged as a Ruling.
- Ticket task-subset isolation (Step 5) and baseline-link loading (Step 7) prefer each ticket's own
  embedded `T0xx` list / baseline link when present, falling back to AC-text matching against
  `tasks.md`, or deriving the baseline path directly from the capability/component name, when a
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
