# /senior-engineer — Pipeline Design (draft, not yet a working skill)

Status: design drafted (architecture agreed via plan review 2026-09-11); not yet implemented as
`SKILL.md`. **Supersedes `assign-tasks`** — see that file's status note. Open questions below
block implementation.

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

## Command

```
/senior-engineer <epic-ref>
```

## Sequence

1. Read `PROJECT.md` and the epic (from `define-epic`).
2. Query `log-decision` for anything already on record for this epic or touching the same
   systems.
3. **Investigate** (gstack `spec` Phase 3 discipline): before asking anything technical,
   Grep/Glob/Read the actual codebase for the systems the epic touches. Ground every question in
   what was actually found — cite the file/line, don't ask "what should I look at?"
4. **Mine existing behavior, if any** (`spec-miner`): if step 3 found that the epic touches an
   existing capability with no prior baseline spec (check for
   `openspec/specs/<capability>/spec.md`), run `spec-miner` against that capability *now*, before
   interviewing further — it extracts current behavior as flat Requirement/Invariant assertions.
   This becomes known, verified context for both the remaining investigation questions (step 3
   continues, now grounded in documented behavior, not just raw code) and for `plan-feature`
   later (its new spec is written as a delta against this baseline, not from scratch). Skip
   entirely for a genuinely greenfield epic — nothing to mine.
5. **UI check**: ask whether a UI/design idea already exists (mockup, Figma, reference
   screenshot/site, or none yet).
   - **No idea exists**: propose a design spike ticket first (e.g. "Design: `<page/flow>` layout
     and states") — functional tickets for that surface wait on it, or proceed with an explicit
     placeholder noted as a follow-up risk if the user prefers to unblock now.
   - **An idea exists *with a concrete reference*** (a live site, an existing component, an
     animation seen somewhere) — **always** propose a separate research spike first, never let
     the build ticket "just match the reference" from memory. Motive (Saqib's own): given only a
     description of a reference, a coding agent approximates it differently every time instead of
     reproducing the actual technique. The research spike's job: inspect the reference for real
     (live site: read its actual DOM/CSS/JS, computed styles, animation timing/easing, and
     network requests for the libraries it loads — this session's own browser tools are the
     model for what that inspection looks like; an existing component: read its real source, not
     just how it looks) and write up the *actual mechanism* — library used, exact CSS
     properties/keyframes, DOM structure, state transitions — then map that mechanism onto this
     project's own stack (what's already available, what's missing, the concrete
     component/file it becomes here). The build ticket that implements the effect is created
     *after* and depends on this spike, and reads its findings instead of re-guessing from the
     original reference. Log the mapping via `log-decision` once written, so the grounding isn't
     lost if the build ticket runs in a separate session.
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

## Relationship to `assign-tasks`

Replaces it. `assign-tasks`'s FE/BE bracket-tag convention on spec-kit's raw `tasks.md` line
grammar is too coarse for the actual ask ("ticket granularity, not just FE/BE labeling") — this
skill's right-sizing step subsumes that classification (a ticket still ends up FE-, BE-, or
shared-scoped, just as one axis of a richer split, not the only one).

## Open questions

1. Step 6 (deciding feature count per epic) has no source skill backing it — every cherry-picked
   technique either operates *within* one feature (right-sizing, code investigation) or *above*
   the epic (gstack's own scoping in Phase 2 of `define-epic`). This specific judgment call may
   need its own worked examples rather than a borrowed technique.
2. If step 5 finds no UI idea and the user wants to proceed anyway (not block on a design spike) —
   does this skill create placeholder/best-guess UI tickets, or explicitly tag them "needs design
   input" and let `build-frontend` surface that gap later?
3. ~~Does this skill call `/plan-feature` as a literal sub-invocation...~~ **Resolved
   (2026-09-11):** yes, a literal sub-invocation via the `Skill` tool — "smart hand-off," not
   inlining. See step 6, above. `plan-feature` and `define-epic`'s own open questions updated to
   match.
4. Confidence/reversibility of ticket right-sizing — if the user disagrees with a specific split
   after tickets are already filed (step 8 confirmed it, but real usage surfaces a bad split
   later), is there a "merge these two tickets" / "split this one further" follow-up mode, or does
   that just become manual Jira editing?
5. Step 4's "no prior baseline spec" check needs a concrete rule — is `openspec/specs/` the fixed
   location regardless of project, or does that path come from `PROJECT.md` (so a project using a
   different spec layout still works)? Also: does a mined baseline ever get re-mined later if the
   existing code changes again before the epic ships, or is it a one-time snapshot per capability?
6. Step 5's research spike needs a concrete deliverable format — is the reference's mechanism
   written directly in the ticket description, a linked file in the repo, or a `log-decision`
   entry the ticket just links to (consistent with the back-link convention above)? Also needs a
   concrete "how to inspect a live reference" mechanism named — this session's own browser tools
   are the model, but the actual coding agent picking up that spike ticket later needs a named,
   available equivalent in its own environment, not just "go look at it."

## TODOs (block turning this into a real `SKILL.md`)

1. ~~Resolve open question 3 first...~~ Resolved above. Remaining: write the exact shape of the
   context object passed to `/plan-feature` in step 6 (a structured handoff, or just prose in the
   invocation prompt?).
2. Read BMAD's `bmad-create-epics-and-stories`, ECC's `jira-integration`, and ECC's `spec-miner`
   skill files in full (already excerpted during source selection) and write the concrete
   ticket-template fields (title format, description sections, type mapping) before finalizing.
3. Write 2-3 worked examples (a real epic → the tickets it should produce) to pressure-test the
   right-sizing judgment before implementation — this is the part with the least mechanical
   backing.
