# SDLC delegation

The delegates run the lifecycle mechanics. You decide when they are allowed to.

## Split of duties

| Step | Who |
|---|---|
| Fetch ticket, read subtasks | Coordinator (you need the scope to write the contract) |
| Create branches from the default branch | Coordinator (naming guardrail, clean start) |
| Move issues to In Progress | Coordinator at dispatch — but out of your own context if the tracker is slow; see below |
| Implement, test, self-report | Delegate |
| Review, fix rounds, verification | Coordinator |
| **Approval** | **User** |
| Commit, push, MR create, MR description | Delegate, after approval |
| Jira transition to code review, required fields | Delegate, after approval |
| Relay MR links to the user | Coordinator |

## What the delegate needs to run the SDLC steps

Delegates do not know your house conventions unless the repo does. In the release prompt, point them at the repo's own MR and commit-message rules and require:

- One commit per MR, in the repo's message format, with the ticket footer
- No AI co-authorship trailers
- MR title and description from a template — summary, ticket links, test plan with the verified results checked off
- A "note for reviewers" section calling out anything a reviewer would otherwise trip on: a deliberate exception, a pre-existing breakage the diff had to fix, a widened diff and why
- Target branch stated explicitly, source branch deleted on merge

## Never serialise the run behind a flaky tracker

Ticket bookkeeping is the least important thing in this workflow and the most likely to hang. Do not retry it in your own context.

- **One timeout, then defer.** Note that the hop did not happen, keep going, and retry at release time when it is actually needed. Three consecutive 300s timeouts on one status hop is 15 minutes of dead wall-clock for a cosmetic field.
- **Diagnose scope before theorising.** Before concluding one endpoint is broken, re-run a *known-good* call against the same server. A read that succeeded earlier in the session is the cheap probe. Observed: a status-transition endpoint was blamed as endpoint-specific when in fact the whole MCP server had degraded — the wrong diagnosis, stated to the user, then corrected.
- **Offload the retry loop, not just the call.** A hanging service costs little in tokens but a lot in wall-clock and turn churn, since each timeout re-invokes you and demands an explanation. Hand the retry loop to a subagent that reports once.
- **Check the delegate can even reach the tracker.** `opencode` carries its own MCP configuration and may have no tracker server at all; a Claude Code subagent inherits yours. Verify before assigning tracker work to a delegate, or it will silently do nothing.
- **Never guess a transition id to route around an outage.** Ids are workflow-scoped, and the write call would hang for the same reason the read did. Hand the status hops to the user instead — they take seconds in a browser.

## Ordering traps

- **Required fields gate transitions.** A workflow validator can reject a status hop for a missing field (AI-usage, fix version) even though the transition is listed as available. Set the field, then retry the hop.
- **Transitions are stepped and IDs are workflow-scoped.** Query the available transitions from the issue's current status every time; never reuse an ID seen on another ticket, and match on the target status name rather than the transition's own name.
- **Parent issues need the same treatment.** When subtask MRs open, the parent's status is stale too.
- **Generated artifacts drift.** Checked-in generated files (swagger, lockfiles) can be rewritten by a build during verification. Check for them at Gate 1 and revert before the commit, or the MR carries unrelated churn.

## Releasing the delegate

Give it the go in one message: the approval, the exact steps in order, and the instruction to report the resulting URLs. Then verify the outcome — branch pushed, MR open against the right target, ticket moved — before you relay anything to the user. Send full clickable URLs, never bare MR numbers.
