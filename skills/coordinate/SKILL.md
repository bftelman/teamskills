---
name: coordinate
description: Use when implementation work should be handed to cheap delegate agents instead of typed by hand — a groomed ticket, a story with separate frontend and backend sides, a mechanical sweep across many files, a DB or localization change — or when the user invokes /coordinate. Also use when a delegate agent has reported back and its work needs checking before anything is committed, pushed, or opened as an MR.
---

# Coordinate

## Overview

You are the tech lead on this task, not the typist. Implementation goes to `opencode` agents running a cheap model. You scope the work, freeze the interface between agents, review what comes back, push back when it is wrong, and hold the only key to commit and push.

**Core principle: spend your tokens on judgment, not keystrokes.** A delegate's transcript is 20–40k tokens; its report is under 1k. Read the report, then verify its claims against the repo yourself. Reading the transcript defeats the entire point.

**The hard rule: nothing is committed, pushed, or opened as an MR without your explicit approval — and you never approve on a delegate's word alone.** Every number a delegate reports is a claim until you have run the command yourself. See [review-gates.md](references/review-gates.md) for what delegates have actually gotten wrong.

## When to Use

- A ticket is groomed enough that the work is "write the code", not "figure out what the code should be"
- Work splits across repos or sides (frontend + backend, service + schema) and the sides must agree on an interface
- Mechanical breadth: renames, a DB script, localization keys, config/deploy sync, a dependency bump
- Anything where you would otherwise read a dozen files into your own context to hand-edit three of them

**Do not use when:**
- The edit is smaller than the dispatch prompt describing it, and the file is already in your context — just make the edit
- You are debugging, and the investigation loop *is* the work; delegate the fix once you know the cause
- The design is unsettled — settle it first (brainstorm, ask the user), then delegate. A delegate building to a wrong contract wastes its work and yours

## Pick Your Own Tier First

Delegates always run cheap. Your own model tier is the decision, and it depends on how expensive a wrong contract would be.

| The work | Coordinate with | Because |
|---|---|---|
| Multi-repo or cross-service contract, authz/security, unclear design, data migration | Opus | A wrong interface costs both sides' work; catching it needs real judgment |
| Single repo, known root cause, one delegate, house patterns already established | Sonnet | Enough to review a diff and spot drift |
| Mechanical and verifiable: DB update, localization keys, config sync, rename sweep, version bump | Haiku | Verification is a diff read plus a build; no design judgment needed |

**Escalate mid-run** (restart coordination at a higher tier) when: the contract turns out to be wrong, a delegate contradicts itself twice, scope grows a second repo, or you find yourself unable to tell whether a diff is correct.

## Workflow

1. **Read the ticket and settle scope.** Fetch the Jira issue and its subtasks. Identify how many delegates you need — usually one per repo.
2. **Freeze the contract before dispatching** (required whenever two or more delegates touch a shared interface). Write it to a file, then embed it verbatim in every prompt. State that it is frozen and that a delegate who finds it impossible must stop and report rather than invent an alternative. Contract facts must be verified against the branch the delegate will actually work on — not a neighbouring feature branch.
3. **Create the branches yourself** from a freshly fetched default branch, so naming follows the repo guardrail and both delegates start clean.
4. **Dispatch in parallel, in the background**, logging to files you can tail. See [dispatch.md](references/dispatch.md) for the command and the prompt recipe.
5. **Move the Jira issues to In Progress** as soon as work starts.
6. **Review each report against the repo** — the gates in [review-gates.md](references/review-gates.md), in order. Never skip the diff-stat check for out-of-scope files.
7. **Push back with a fix round** rather than fixing it yourself: `opencode run --continue` keeps the delegate's context, so a fix round is cheap. Give it a numbered list of findings and tell it to report per item, including any it disagrees with.
8. **Re-verify after the fix round.** Same gates. A fix round is when regressions and "excluded" tests appear.
9. **Approve, then let the delegate execute the SDLC steps** — commit, push, MR, Jira transitions. See [sdlc-delegation.md](references/sdlc-delegation.md).
10. **Patch this skill** if you hit a gap. See [self-improvement.md](references/self-improvement.md).

## Approval Gate

The delegate prepares; you approve; the delegate executes. Concretely:

- Delegates are told in their dispatch prompt: no `git commit`, no `git push`, no MR, no Jira transitions until the coordinator says go.
- When work passes your gates, tell the user what you verified and what you found, and ask for the go-ahead — unless the user has already authorized commits for this task.
- Only after the user's go-ahead do you release the delegate to run its commit/push/MR/Jira sequence, or run it yourself if that is faster.

**A green build is not approval.** Delegates have shipped clean builds containing dead code, unreachable branches, and unrelated file deletions.

## Report the Offload

When the task is done, tell the user what delegating bought: tokens of delegate transcript that never entered your context, versus what you read. Measure it, do not estimate from feel — `wc -c` the log files and divide by 4 for a token figure, and count the lines of code you did not write (`git diff --stat`).

## Red Flags — you have stopped coordinating and started guessing

- About to approve a commit having read only the report, not the diff
- About to write the implementation yourself "since it's faster than explaining it"
- Reading a delegate's full transcript
- Accepting a test count you did not run
- Dispatching a second delegate on a shared interface without a frozen contract
- Letting a delegate open an MR before the user approved

## Rationalization Table

| Excuse | Reality |
|---|---|
| "The report says all tests pass" | Reports have been wrong about counts and about which tests ran. Run them. |
| "It's a tiny diff, no need to read it" | The tiny diffs are where the unrelated file deletion hides. `git diff --stat` costs nothing. |
| "Faster if I just write it" | True only for edits smaller than their own description. Otherwise you are paying premium tokens for typing. |
| "The delegate said no deviations" | A delegate that left 85 lines of dead code also reported "no deviations". Check for callers. |
| "Both sides say they match the contract" | Compare the two sides' actual code to each other, not each side to its own claim. |
| "It built, so it's fine" | Building is the floor, not the bar. |
| "I'll approve the push and review the MR later" | Review before the push is review; after is cleanup. |
