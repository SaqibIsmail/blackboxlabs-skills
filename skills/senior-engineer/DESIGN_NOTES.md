# /senior-engineer — Pipeline Design

**Status (2026-09-13): implemented.** See `SKILL.md`/`SETUP.md`. **Supersedes `assign-tasks`** —
see that file's status note. This file stays as the rationale record.

**Changed while implementing (2026-09-13):**
- `log-decision` wasn't fully decided yet (numbering, query format) — resolved and implemented
  first, out of the original order, since this skill depends on it directly. See
  `log-decision/DESIGN_NOTES.md`.
- Reading BMAD's real `step-02-design-epics.md`/`step-03-create-stories.md` in full surfaced
  concrete criteria for the feature-split judgment (Step 6's open question 1, previously "no
  source skill backing it") and a real story/ticket template (As a/I want/So that + Given/When/
  Then) neither this file nor the earlier draft had. Flagged both to Saqib before adopting;
  approved.
- Reading ECC's `spec-miner` in full showed its own output already records a `Last verified:
  (commit <sha>)` line — that's the concrete staleness-check mechanism TODO 4 was still missing,
  not something to invent separately. Also: its "present the whole codebase's capability list"
  step is skipped here, since Step 3's investigation already identifies the one relevant
  capability.

## What this is

Takes one epic (from `define-epic`) and turns it into small, individually-implementable tickets —
investigating scope, asking whatever it takes to get there, and creating the tickets linked to
the epic. This is the "senior engineer investigates the ask" role from the original request,
combining four cherry-picked techniques (none alone covers the whole job):

- **Investigation, code-grounded** — `garrytan/gstack`'s `spec` skill, Phase 3 ("Technical
  Interrogation"): a hard rule to read actual code (Grep/Glob/Read) *before* asking any technical
  question, and cite `path:line` in it — never a generic checklist question. This is what makes
  the FE/BE scoping real instead of guessed.
- **Ticket right-sizing** — `obra/superpowers`'s `writing-plans` skill's "Task Right-Sizing"
  principle: a ticket is as small as the unit that carries its own test cycle and is worth a
  reviewer's independent gate — split only where a reviewer could approve one ticket while
  rejecting its neighbor. This is *why* an animation gets its own ticket: not a fixed rule
  ("animations are always separate"), a judgment call this principle gives language to (matches
  the "judgment via questions, no fixed heuristic list" call already made for this skill).
- **Epic → stories structure** — BMAD-METHOD's `bmad-create-epics-and-stories` skill: organizing
  the breakdown by user value with real acceptance criteria, not by technical layer.
- **Draft-then-file discipline** — gstack `spec` skill's Phase 4: present the full draft (all
  proposed tickets) and ask "does this capture it? what did I get wrong?" — iterate until
  confirmed, before creating anything.
- **Brownfield spec extraction** — `affaan-m/ECC`'s `spec-miner` agent: for an epic that touches
  a part of `blackboxlabs` (or any target project) with existing, un-specced behavior, mine that
  behavior into a baseline spec *before* `plan-feature` writes a new spec-kit spec for it —
  otherwise `plan-feature`'s spec is written as if the feature were greenfield, when really it's
  a change to something that already exists and already has behavior worth preserving.

Ticket creation itself uses `affaan-m/ECC`'s `jira-integration` skill (concrete MCP-based Jira
read/write) — or spec-kit's own `/speckit.taskstoissues` when `ticketing.system: github-issues`
(same delegation `assign-tasks` already planned).

All four cherry-picked as prose/technique, not runtime — same rule as every other skill in this
pipeline.

**Declared dependency (resolved 2026-09-12): Playwright.** The research spike (step 5, "an idea
exists with a concrete reference") named a specific required tool for inspecting a live reference
site's real DOM/CSS/JS/timing, rather than leaving it to "whatever browser tool happens to be
available" — Playwright, since it's already referenced elsewhere in this pipeline (ECC's
`tdd-workflow` uses it for E2E tests), so target projects only need one browser-automation
dependency, not two competing ones.

## Command and sequence

Superseded by `SKILL.md` Steps 1–10, which incorporate the BMAD feature-split criteria and
story template, and the concrete spec-miner staleness check, none of which this draft had. Kept
here only as a historical note.

<details>
<summary>Original draft sequence (superseded)</summary>

```
/senior-engineer <epic-ref>
```

1. Read `PROJECT.md` and the epic (from `define-epic`).
2. Query `log-decision` for anything already on record for this epic or touching the same
   systems.
3. **Investigate** (gstack `spec` Phase 3 discipline): before asking anything technical,
   Grep/Glob/Read the actual codebase for the systems the epic touches. Ground every question in
   what was actually found — cite the file/line, don't ask "what should I look at?"
4. **Mine existing behavior, if any** (`spec-miner`): if step 3 found that the epic touches an
   existing capability with no prior baseline spec, check `openspec/specs/<capability>/spec.md` —
   **resolved 2026-09-12: this is a fixed path everywhere**, not project-configurable, for
   consistency across every project this pipeline runs in. If missing, run `spec-miner` against
   that capability *now*, before interviewing further — it extracts current behavior as flat
   Requirement/Invariant assertions. If a baseline *does* already exist, **resolved 2026-09-12:
   re-mine if the touched code changed since it was mined** — compare the capability's source
   file(s) mtime/hash against the baseline spec's own recorded mining date/commit SHA; re-run
   `spec-miner` if they've diverged, so the baseline never quietly goes stale. This becomes known,
   verified context for both the remaining investigation questions (step 3 continues, now
   grounded in documented behavior, not just raw code) and for `plan-feature` later (its new spec
   is written as a delta against this baseline, not from scratch). Skip entirely for a genuinely
   greenfield epic — nothing to mine.
5. **UI check**: ask whether a UI/design idea already exists (mockup, Figma, reference
   screenshot/site, or none yet).
   - **No idea exists**: propose a design spike ticket first (e.g. "Design: `<page/flow>` layout
     and states") — functional tickets for that surface wait on it. **Resolved 2026-09-12,** if
     the user wants to proceed anyway rather than block on the spike: the resulting ticket(s)
     describe the functional UI requirement needed to complete the task, plus an explicit "no
     design decided yet" note in the description — `build-frontend` makes the actual design
     decision itself when it picks the ticket up, rather than this skill guessing at one now.
   - **An idea exists *with a concrete reference*** (a live site, an existing component, an
     animation seen somewhere) — **always** propose a separate research spike first, never let
     the build ticket "just match the reference" from memory. Motive (Saqib's own): given only a
     description of a reference, a coding agent approximates it differently every time instead of
     reproducing the actual technique. The research spike's job: inspect the reference for real
     using Playwright (live site: read its actual DOM/CSS/JS, computed styles, animation
     timing/easing, and network requests for the libraries it loads; an existing component: read
     its real source, not just how it looks) and write up the *actual mechanism* — library used,
     exact CSS properties/keyframes, DOM structure, state transitions — then map that mechanism
     onto this project's own stack (what's already available, what's missing, the concrete
     component/file it becomes here). **Resolved 2026-09-12:** the spike's findings are embedded
     directly in the resulting build ticket's description (not just linked) — the build ticket
     that implements the effect is created *after* and depends on this spike, and reads its
     findings inline instead of re-guessing from the original reference. Log the mapping via
     `log-decision` once written too, so the grounding also survives if the build ticket runs in
     a separate session.
   - **An idea exists with no concrete reference** (a verbal description only): no research spike
     needed — proceeds straight into the feature/ticket split below.
6. **Decide the feature split**: does this epic need one `plan-feature` pass or several (e.g. a
   "notifications" epic → "email notifications" + "in-app notifications" as separate spec-kit
   features)? Call `/plan-feature` once per identified feature (via the `Skill` tool, a real
   sub-invocation — resolves open question 3, below), **passing along as context**: `define-epic`'s
   Phase 1–2 answers, this skill's own investigation findings from step 3, and any baseline spec
   from step 4. `plan-feature` only interviews about what that context leaves unanswered — it does
   not re-ask scope/non-goals already covered at the epic level.
7. **Right-size into tickets**: walk each feature's `tasks.md`, and using the right-sizing
   judgment above, group or split spec-kit's `T0xx` tasks into tickets small enough that a coding
   agent implementing one doesn't also have to hold an unrelated concern in its head (structure
   vs. styling/animation vs. data-fetching vs. state, as separate tickets *when the questioning
   surfaces them as genuinely separable* — not a fixed checklist). Classify each as `task`,
   `spike` (research/design unknowns), or `bug` (only relevant when the epic is itself a fix).
8. **Present the draft**: show all proposed tickets — title, type, linked `T0xx` IDs, and which
   feature/spec they come from. Ask "does this capture it? what's wrong?" Iterate until confirmed.
9. **Create tickets** via `jira-integration` (or delegate to spec-kit's `/speckit.taskstoissues`
   for the `github-issues` backend), each linked to the parent epic and to its originating
   `spec.md`/`plan.md`. **Each ticket's description also embeds the path/link to its relevant
   `log-decision` entry (entries)** — the epic's decision file from `define-epic` and, if this
   ticket's feature has its own, `plan-feature`'s entry too — using the path `log-decision`
   returned when it wrote them (see `log-decision/DESIGN_NOTES.md`'s "Ticket ↔ decision
   back-link"). Without this, a ticket in Jira has no way back to why it was scoped the way it
   was.
10. Call `log-decision` if the investigation surfaced a non-obvious scoping call (e.g. "epic split
    into two features because X" or "deferred Y as a separate spike because Z").

</details>

## Relationship to `assign-tasks`

Replaces it. `assign-tasks`'s FE/BE bracket-tag convention on spec-kit's raw `tasks.md` line
grammar is too coarse for the actual ask ("ticket granularity, not just FE/BE labeling") — this
skill's right-sizing step subsumes that classification (a ticket still ends up FE-, BE-, or
shared-scoped, just as one axis of a richer split, not the only one).

## Open questions

1. ~~Step 6 (deciding feature count per epic) has no source skill backing it...~~ **Resolved
   (2026-09-12):** judgment via questions, no fixed heuristic — same principle as ticket
   right-sizing, kept consistent rather than inventing a second rule shape for a similar problem.
2. ~~If step 5 finds no UI idea and the user wants to proceed anyway...~~ **Resolved
   (2026-09-12):** see step 5, above — functional requirement + "no design decided yet" note,
   `build-frontend` decides the design at build time.
3. ~~Does this skill call `/plan-feature` as a literal sub-invocation...~~ **Resolved
   (2026-09-11):** yes, a literal sub-invocation via the `Skill` tool — "smart hand-off," not
   inlining. See step 6, above. `plan-feature` and `define-epic`'s own open questions updated to
   match.
4. ~~Confidence/reversibility of ticket right-sizing...~~ **Resolved (2026-09-12):** no built-in
   follow-up mode — a bad split found after filing is fixed by manual Jira editing. Simplest
   option; revisit only if this turns out to happen often in practice.
5. ~~Step 4's "no prior baseline spec" check needs a concrete rule...~~ **Resolved (2026-09-12):**
   fixed path (`openspec/specs/<capability>/spec.md`) everywhere, not project-configurable. And a
   mined baseline **does** get re-mined if the touched code changed since mining — see step 4,
   above, for the staleness check.
6. ~~Step 5's research spike needs a concrete deliverable format...~~ **Resolved (2026-09-12):**
   findings go directly in the build ticket's description (not just a link), and the named
   inspection tool is Playwright — see step 5 and "Declared dependency," above.

All 6 open questions for this skill are now resolved. Remaining work is write-up, not more
decisions — see TODOs below.

## TODOs

None blocking — this skill is implemented. See `SKILL.md`/`SETUP.md`.

## Findings from live testing against `blackboxlabs` (2026-09-13) — the worked example

This is that worked example (real epic `SCRUM-5`, "BlackboxLabs landing page"). Findings, all
already folded into `SKILL.md`:

1. **Step 4 needed a second mining path.** The epic's existing-system touchpoint
   (`features/logo-animation/`, a particle-physics scroll scene) is UI/visual code, not business
   logic — `spec-miner`'s Requirement/Invariant model didn't fit it (no real WHEN→THEN triggers in
   a render loop). Saqib pushed back on dropping mining entirely rather than accepting "greenfield,
   skip it." Added a new **Step 4b: Component-Interface Mining** — an original technique (no
   external source covers this), extracting props/API surface, hardcoded content, dependencies,
   `file:line`-cited constants, accessibility notes, and existing integration points, written to
   `openspec/components/<name>/interface.md` (sibling convention to `spec-miner`'s output path).
   Applied for real against `LogoOpenScene`/`SmoothScroll` — found real, load-bearing facts:
   zero props, hardcoded wordmark text, already wired as the home page's hero in `app/page.tsx`.
2. **Ticket type mapping was untested until now.** `blackboxlabs`'s real Jira project (a default
   Scrum template) has no native `Bug`/`Spike` issue type — only `Epic`/`Task`/`Story`/`Subtask`.
   Added: use `Task` + a label when the project lacks the native type, rather than assuming it
   exists.
3. Same Jira-formatting fix as `define-epic` applies here (Step 9).

## TODOs

None — Component-Interface Mining (Step 4b) now uses the identical commit-SHA staleness
mechanism as `spec-miner` (Step 4a). Both mining paths in Step 4 are symmetric.

## Findings from live testing, continued (2026-09-13) — full ticket creation for real

Ran Steps 6-10 for real against `SCRUM-5`: created 5 `Story`-tier groupings (22 issues total —
1 epic + 5 stories + 16 subtasks) via the Jira `parent` field, confirmed mechanically first with
a throwaway test issue pair before bulk-creating. Two more fixes applied:

1. **Story-as-grouping-tier formalized** (Saqib explicitly wanted "smaller epics" containing
   tickets — Jira has no nested-Epic support without premium Advanced Roadmaps). `Story` fills
   that role; cross-group integration notes go on the epic as a comment, not buried in one
   subtask. Added to Step 9 and "Explicit defaults" as judgment-based (skip for a small epic).
2. **Sequencing bug found and fixed: Step 9's back-link assumed Step 10's decisions already
   existed.** They don't — Step 10 runs after Step 9, so a decision logged there can't have been
   embedded in tickets already created. Fixed: Step 10 now explicitly back-links retroactively
   (a follow-up comment on the epic/relevant tickets) instead of assuming it could happen inline.
   Caught this because I actually forgot the back-link while live-creating tickets and had to
   patch it in after — a real gap, not a hypothetical one.

## Vault restructure follow-on (2026-09-16)

Part of the broader `log-decision` restructure (see that skill's own `DESIGN_NOTES.md`):
`openspec/specs/*` and `openspec/components/*` move from the project repo into the vault, at
**company** level specifically — not nested inside whichever epic happens to mine them first,
since a capability/component is meant to be reused by a later, unrelated epic too. Also: ticket
creation (Step 9) now seeds each ticket's own vault file directly (at
`<epic>/<story>/<ticket-key>-<slug>.md`) instead of only linking to a pre-existing epic/feature
entry — which incidentally simplifies Step 10's old "retroactive back-link" handling, since the
file already exists by the time a later deviation needs to be appended to it.

Also: the `skiper17` reference genuinely didn't match Saqib's intent (a sticky *image*-stack, no
text slots, no pipe connector) — resolved by splitting mechanism (borrow the pin+scrub scroll
technique) from content (build text-cards and the pipe connector custom). The pipe connector
itself reuses `logo-animation`'s `particlePhysics.ts` directly, per Saqib's explicit ask — a good
real example of Component-Interface Mining's output (Step 4b) actually getting used downstream.

## Bug found and fixed: `plan-feature`/`tasks.md` contract gap (2026-09-14)

The tickets created during the 2026-09-13 test (above) never actually went through
`plan-feature`/spec-kit at all — Step 6 concluded one feature was enough and the test used a
hand-built shortcut instead, since `spec-kit` wasn't installed yet. Two build tickets from that
shortcut (quote-animation, sticky-card mechanism) ended up adapting a reference component with no
preceding research spike, unlike three sibling tickets that correctly got one (navbar, socials,
nav-IA) — a plain inconsistency in applying Step 5's own rule that night.

Running the real pipeline the next day (installing `spec-kit`, driving
`speckit-specify→clarify→plan→tasks` for real) surfaced the deeper, structural version of the same
problem: this skill's Step 7 already assumed it could "walk each feature's `tasks.md`," but
`plan-feature`'s Step 5 never actually produced one — it stopped at `plan.md`. And even once fixed,
Step 6's hand-off to `plan-feature` never included Step 5's own UI-check outcome, so an unresearched
reference had no path into `/speckit.plan`'s research phase, so it could never become a task in
`tasks.md` for Step 7 to split into a spike ticket. Fixed both, together (`plan-feature`'s
`SKILL.md` Step 1/4/5, this skill's Step 6/7) — one gap spanning both files, not two independent
ones. `tasks.md` is now always produced and is Step 7's actual, required input, not spec-kit's own
`tasks.md` generation being optional cross-check tooling.

## FE/BE/SHARED labeling made real, not just implicit (2026-09-17)

Checking the real 25 tickets against the "Relationship to `assign-tasks`" note below exposed that
its claim — "a ticket still ends up FE-, BE-, or shared-scoped" — was only ever true as an
unlabeled side-effect of right-sizing, never an actual tag written anywhere. Nothing in Step 7 or
Step 9 wrote FE/BE/SHARED onto a ticket; a build skill (or a human) had no way to tell which was
which without re-reading the ticket's file list by hand.

Motivation: Saqib wants `build-frontend`/`build-backend` to pick up their own tickets directly by
this label, rather than a separate dispatcher agent classifying each ticket right before build —
reasoned through explicitly rather than assumed: `senior-engineer` already has the classification
signal for free from its own Step 3 investigation, so a dispatcher agent would only re-derive the
same thing later with *less* context, for no real benefit, while also losing the human-visible
win of a real Jira label anyone can see without running an agent.

Fixed: Step 7 now classifies scope (frontend/backend/shared) as a second axis alongside type
(task/spike/bug), reusing `assign-tasks`'s own file-path-then-keyword signal rather than
reinventing one. Step 9 now writes it as a real label, the same mechanism already used for
`spike`/`bug` — no new Jira setup, no Components, no custom field.
