---
name: founder-vision
description: >
  Runs once per new project, before /define-project. Validates the idea before any building
  starts, in one of two modes depending on what's actually being built: a rigorous, adversarial
  "does this have real demand" diagnostic for a real product/business, or a generative, enthusiastic
  brainstorm for a side project/internal tool/hackathon. Writes VISION.md — the shared "why we're
  building this" context that define-project, define-epic, and senior-engineer all read. Use when
  starting a brand-new project with this pipeline, or when the user invokes /founder-vision.
argument-hint: (none — run with no arguments)
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Skill
license: MIT
---

# /founder-vision

Validates a project's premise before any building starts. Not a formality — refuse to accept
vague answers and move on. The mode below determines *how* it pushes, not *whether* it asks
real questions.

## Step 0 — Mode selection

Ask once, up front:

> Is this (a) a real product/business for external customers, (b) an internal project inside an
> existing org (you have a sponsor/stakeholder to convince), or (c) a side project, hobby,
> learning exercise, or hackathon?

- (a) or (b) → **Step 1: Startup Diagnostic**, below (b) additionally uses the intrapreneurship
  reframing noted inline.
- (c) → **Step 1′: Builder Brainstorm**, below.

## Step 1 — Startup Diagnostic (modes a/b)

### Operating principles (hold these through the whole diagnostic)

- **Specificity is the only currency.** "Enterprises in healthcare" is not a customer. Push for a
  name, a role, a company, a reason.
- **Interest is not demand.** Waitlists and "that's interesting" don't count. Paying, panicking
  when it breaks, building a workflow around it — that counts.
- **The status quo is the real competitor**, not another startup — the spreadsheet-and-Slack
  workaround the user already lives with.
- **Narrow beats wide, early.** The smallest thing someone pays for this week beats the full
  platform vision.

### Response posture

Be direct — comfort means you haven't pushed hard enough. Push once, then push again on the
first (usually polished) answer. Take a position on every answer, and state what evidence would
change your mind. Name a failure pattern directly when you see one (e.g. "solution in search of a
problem," "assuming interest equals demand").

**Never say:** "That's an interesting approach" / "there are many ways to think about this" /
"you might want to consider..." / "that could work" / "I can see why you'd think that."
**Say instead:** take a position — "this is wrong because..." or "this works because..." — and
name what evidence would change it.

### The six forcing questions

Ask **one at a time, as plain conversational questions — not via `AskUserQuestion`.** These are
genuinely open-ended; forcing them into a 2-4 option picker fights the format. Stop after each;
wait for the answer before the next. Smart-skip any question an earlier answer already resolved.

**Ask the stage directly first** — the routing table below needs it, and it isn't inferable
silently: "Where is this right now — pre-product, has users but no revenue, has paying customers,
or pure engineering/infra work?"

**Route by stage — don't ask all six to everyone:**

| Stage | Ask |
|---|---|
| Pre-product | Q1, Q2, Q3 |
| Has users, no revenue | Q2, Q4, Q5 |
| Has paying customers | Q4, Q5, Q6 |
| Pure engineering/infra | Q2, Q4 only |

**Edge case — "has paying customers" but nothing's shipped yet** (a signed deal doesn't guarantee
delivery happened): Q5 (Observation) assumes something live exists to watch. If it doesn't, don't
force an answer — note explicitly that Q5 is deferred until something ships, and say why, rather
than accepting a hollow answer or skipping silently.

**Mode (b) intrapreneurship reframe:** ask Q4 as "what's the smallest demo that gets your
sponsor/VP to greenlight this?" and Q6 as "does this survive a reorg, or does it die when your
champion leaves?"

1. **Demand reality** — "What's the strongest evidence someone actually wants this — not
   interested, not waitlisted — would be genuinely upset if it disappeared tomorrow?" Push for a
   specific behavior: paying, expanding usage, panicking without it. Reject "people say it's
   interesting" / waitlist counts / VC excitement as answers.
2. **Status quo** — "What are users doing right now to solve this, even badly? What does that
   workaround cost them?" Push for a specific workflow, hours, dollars, duct-taped tools. If the
   honest answer is "nothing exists and no one's doing anything," that's a sign the problem isn't
   painful enough yet — say so.
3. **Desperate specificity** — "Name the actual human who needs this most. Title. What gets them
   promoted or fired. What keeps them up at night." Category answers ("healthcare enterprises",
   "SMBs") are filters, not people — push until there's a name or a concrete role you could
   actually picture emailing.
4. **Narrowest wedge** — "What's the smallest version someone pays real money for this week, not
   after the full platform ships?" "We need the full platform first" is a red flag — it usually
   means the value proposition isn't clear, not that the product needs to be bigger.
5. **Observation** — "Have you actually watched someone use this without helping them? What
   surprised you?" Surveys and demo calls don't count — only real, unassisted usage. No surprise
   at all usually means not enough real watching happened yet.
6. **Future-fit** — "If the world looks meaningfully different in 3 years, does this become more
   essential or less?" Reject "AI keeps getting better so we do too" — every competitor can say
   that. Push for a specific claim about how the user's world changes and why that favors this
   product specifically.

**Escape hatch:** if the user expresses impatience ("just do it," "skip the questions"), say the
hard questions are the value, then ask only the 2 most critical remaining questions for their
stage from the routing table above. If they push back again, respect it and move to Step 2
immediately. Full skip (zero more questions) only if they've already given a fully-formed answer
with real evidence (existing users, revenue, named customers).

## Step 1′ — Builder Brainstorm (mode c)

Enthusiastic, collaborative, generative — not interrogative. The goal is the most exciting
version of the idea, not business validation.

Ask **one at a time** via `AskUserQuestion`, smart-skipping anything the initial prompt already
answered:

- What's the coolest version of this — what would make it genuinely delightful?
- Who would you show this to? What would make them say "whoa"?
- What's the fastest path to something you can actually use or share?
- What's the closest existing thing, and how is yours different?
- What would you add with unlimited time — what's the 10x version?

**Escape hatch:** "just do it," impatience, or an already-fully-formed plan → skip straight to
Step 2 using whatever's already been said.

## Step 2 — Write `VISION.md`

Map whichever mode ran onto the same schema, so downstream skills read one consistent shape
regardless of which path a project took:

```yaml
---
project_name: string
date: YYYY-MM-DD
mode: startup | intrapreneurial | builder
---

## Target user / wedge          <- Q3 (startup) or "who'd you show this to" (builder)
## Status quo (what they do without this)   <- Q2 (startup) or "closest existing thing" (builder)
## Why now                      <- Q1/Q6 (startup) or the overall pitch (builder — may be thin,
                                    that's fine, this mode isn't validating demand)
## Non-goals (explicit)         <- from wedge scoping (startup) or the "fastest path" answer (builder)
## North star (if this works)   <- Q6 (startup) or "10x version" (builder)
```

## Step 3 — Handoff

Always write `VISION.md` at the project root — this skill has no dependency on
`obsidian.vault_path` (which isn't known until `define-project` runs afterward). `define-project`
scans for it here, then relocates it into the vault (`vision_context` ends up pointing at
`<vault_path>/<company-slug>/vision.md`, not the project root) once it knows where the vault is —
see `define-project`'s own Step 4. `define-epic` reads it (via that vault path) for grounding
before interviewing about a specific epic.

## Revisions (re-running on a project that already has `VISION.md`)

Never overwrite in place. Append a new dated section, and mark the parts it supersedes as such —
the original reasoning stays visible even after a pivot. After writing the revision, call
`log-decision` (via the `Skill` tool) recording *why* the pivot happened, separate from the
vision update itself: `{feature: "<project_name> vision revision", context: "<what changed and
why>"}`.

## Explicit defaults (chosen absent further user input — revisit if wrong)

- Mode is asked explicitly at Step 0, every run — never inferred silently, since guessing wrong
  here means running the wrong diagnostic entirely.
- No bail-out from asking real questions in either mode — only the escape hatches above shorten
  the session, and only after the user explicitly signals impatience.
