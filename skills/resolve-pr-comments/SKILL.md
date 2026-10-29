---
name: resolve-pr-comments
description: >
  Takes an already-open PR and drives its required CI checks (Greptile review and the
  `Standards Review` check) to green. Reads the actual failing findings, then for each one either
  fixes it in code via a fresh implementer subagent, or — when the finding is a false positive or
  already covered by a recorded decision — has a fresh agent write the reasoned reply to
  Greptile's comment. Re-triggers only whichever check failed last round, and repeats until every
  check is green, then messages the user that the PR is ready to merge. Never merges. Called by
  `build` at B10 right after it opens the PR; also invokable directly with a PR number/URL.
argument-hint: '<pr-url-or-number> [--branch <name>] [--worktree <path>] [--tickets KEY-1,KEY-2]'
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
license: MIT
---

# /resolve-pr-comments

Input: an open PR. Output: a PR whose required checks are all green, plus a message to the user
saying so. This skill never merges — merging is the user's call.

The required checks, per the target repo's `docs/hosting.md`:
- **`Standards Review`** (`standards-ci.yml`) — scored sticky comment plus inline comments.
  Re-trigger: comment `/standards` on the PR.
- **`Greptile Review`** — inline findings, 5/5 confidence required. Re-trigger: comment
  `@greptile review this PR again`. Greptile reads replies to its inline comments and remembers
  them, so a reasoned dispute reply counts.

If the target repo's `docs/hosting.md` names different checks or trigger phrases, those win.

## Step 1 — Gather the minimum context

1. Resolve the PR: number, head branch, repo. Use `gh` (or `gh-axi` if present).
2. Locate the working copy: the `--worktree` path if given, else check out the head branch in a
   fresh worktree. All fixes are committed to the PR's head branch and pushed.
3. Read `PROJECT.md` (for the `log-decision` vault path and coding-standards doc routing).
4. Collect the ticket keys: from `--tickets`, else parse them out of the PR description.

Do **not** load the build run's session history. Everything a fresh agent needs gets assembled
per finding in Step 3.

## Step 2 — Read what CI actually says

Never infer status; read it.

- `gh pr checks <pr>` for the state of every check.
- `gh pr view <pr> --comments` plus `gh api repos/{owner}/{repo}/pulls/<pr>/comments` for inline
  review comments (Greptile's and the standards bot's).
- Keep only **unresolved** findings from the **latest** run of each check. A finding already
  replied to and not repeated by the newest review is closed.

If checks are still `pending`/`in_progress`, wait on them without a polling loop: end the turn
with a short status line and resume on the next event (the desktop app's CI monitor, or the
user's message). Do not sit in a `sleep` loop.

If every required check is green → go to Step 6.

## Step 3 — Triage each finding, with a fresh agent

For each unresolved finding, dispatch a **fresh** subagent (new spawn, not a resumed implementer).
Give it only the isolated context it needs — not the PR's thread history or earlier rounds:

- the finding text and the file/line it's attached to
- that file's diff on this PR (base...head)
- the relevant `log-decision` entry for the ticket/Story, if one exists (query the skill; if none
  exists, say so — don't pretend context was available)
- the project's coding-standards docs for that area

Its job is to **decide and act**, returning one of:

- **VALID → fix.** It makes the minimal change following the project's standards, runs the
  relevant verification (tests/lint/typecheck for what it touched) fresh, reads the output, and
  only then reports the fix as done. Independent findings may be dispatched in parallel; findings
  touching the same file go to one agent sequentially.
- **NOT A PROBLEM → dispute.** It writes the reply to the inline comment: specific, citing the
  code or the `log-decision` entry that justifies the current behavior. It does not edit code.
- **UNSURE / needs a product call** → return to the orchestrator; handled in Step 5.

Standards-check findings are code rules, so lean toward fixing; disputes there need a concrete
reason (e.g. the rule's own doc exempts this case). Greptile disputes are the common case for
false positives.

## Step 4 — Apply, push, re-trigger only what failed

1. Commit the fixes (one commit per round, message listing the findings addressed), push.
2. Post the dispute replies on the corresponding inline comments.
3. Re-trigger **only the checks that failed last round**: `/standards` if Standards failed,
   `@greptile review this PR again` if Greptile failed, both if both did. Don't re-trigger a
   check that is already green.
4. Record the round in the ledger: findings, fix vs dispute, commit SHA.
5. Call `log-decision` for each dispute and each non-obvious fix — this is what future rounds and
   future PRs read as prior rationale.

Then return to Step 2 (wait for the new run, read the real result).

## Step 5 — Loop limits and escalation

- **Cap: 5 rounds.** Round counting is per PR, not per finding.
- **A finding that survives a fix twice** (same finding, re-raised after two attempts) gets a fresh
  agent of the next model tier up, given the finding plus a one-paragraph summary of what was
  already tried and why it didn't land.
- **A dispute Greptile re-raises after a reasoned reply** is not argued a second time with the
  same reasoning. Either fix it after all, or escalate.
- **UNSURE findings, or hitting the cap with checks still red**: stop and message the user with
  the PR link, the exact failing finding(s), what was tried, and the question or decision needed.
  Don't merge, don't override branch protection, don't mark checks as passed.

## Step 6 — Done: message the user to merge

Only when `gh pr checks` shows **every required check green**, re-read fresh (not assumed from
the last push):

- Send the user a message with: PR link, one-line check summary (Standards score, Greptile
  confidence), number of rounds taken, and a short list of what changed — fixes made and disputes
  replied. Use `PushNotification` if available so it reaches them away from the terminal.
- Transition nothing and merge nothing. State plainly: "All checks green — ready for you to
  merge."
- Return the same summary to the caller (`build`, if it invoked this skill).

## Guardrails

- Never force-push, never merge, never edit branch protection or required-check config.
- Never claim a finding resolved without having re-run its verification and read the output.
- Never re-trigger a check more than once per pushed state.
- Replies to Greptile are for real false positives only; a dispute is not a way to make a red
  check go away.
