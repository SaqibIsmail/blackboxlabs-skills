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
   screenshot, or none yet).
   - If none exists: propose a design spike ticket first (e.g. "Design: `<page/flow>` layout and
     states") — functional tickets for that surface wait on it, or proceed with an explicit
     placeholder noted as a follow-up risk if the user prefers to unblock now.
6. **Decide the feature split**: does this epic need one `plan-feature` pass or several (e.g. a
   "notifications" epic → "email notifications" + "in-app notifications" as separate spec-kit
   features)? Call `/plan-feature` once per identified feature to produce
   `specs/<feature>/{spec.md,plan.md,tasks.md}` via spec-kit — handing it any baseline spec from
   step 4 as input, for the features it applies to.
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
   `spec.md`/`plan.md`.
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
3. Does this skill call `/plan-feature` as a literal sub-invocation (via the `Skill` tool) per
   feature, or does it need to inline a slimmed-down version of `plan-feature`'s own interview to
   avoid the "duplicate questions" risk already flagged in both `plan-feature` and `define-epic`'s
   open questions? Whichever is chosen here has to match what those two files decide.
4. Confidence/reversibility of ticket right-sizing — if the user disagrees with a specific split
   after tickets are already filed (step 8 confirmed it, but real usage surfaces a bad split
   later), is there a "merge these two tickets" / "split this one further" follow-up mode, or does
   that just become manual Jira editing?
5. Step 4's "no prior baseline spec" check needs a concrete rule — is `openspec/specs/` the fixed
   location regardless of project, or does that path come from `PROJECT.md` (so a project using a
   different spec layout still works)? Also: does a mined baseline ever get re-mined later if the
   existing code changes again before the epic ships, or is it a one-time snapshot per capability?

## TODOs (block turning this into a real `SKILL.md`)

1. Resolve open question 3 first — it determines whether this skill depends on `plan-feature` at
   runtime or duplicates part of it.
2. Read BMAD's `bmad-create-epics-and-stories`, ECC's `jira-integration`, and ECC's `spec-miner`
   skill files in full (already excerpted during source selection) and write the concrete
   ticket-template fields (title format, description sections, type mapping) before finalizing.
3. Write 2-3 worked examples (a real epic → the tickets it should produce) to pressure-test the
   right-sizing judgment before implementation — this is the part with the least mechanical
   backing.
