# Self-improvement

This skill is expected to be incomplete. Every coordination run that surprises you is data, and the surprise is worth more than the run.

## When to patch

Patch immediately, in the same session, when any of these happen:

- A delegate failed in a way [review-gates.md](review-gates.md) does not list
- A gate passed something that later turned out broken — the gate is too weak, or a gate is missing
- A dispatch prompt was missing a slot, and the delegate went out of scope or invented an interface because of it
- You picked the wrong coordinator tier: you burned Opus on a rename, or you were out of your depth on Haiku
- An SDLC step blocked on something not in [sdlc-delegation.md](sdlc-delegation.md)
- The user corrected you on something this skill should have told you

## How to patch

Match the form to the failure:

| Failure | Where it goes |
|---|---|
| Delegate produced something wrong that a check would catch | A row in the observed-failure-modes table, plus the gate that catches it |
| Delegate went out of scope or guessed an interface | A slot in the dispatch prompt recipe |
| You rationalized skipping a check | A row in the rationalization table in SKILL.md |
| Wrong model tier | A row or a sharper predicate in the tier table |
| Lifecycle mechanics bit you | An ordering trap in sdlc-delegation.md |

Rules for the patch itself:

- **One row, not a story.** "Reported N tests, actually M, caught by Gate 4" — not a paragraph about the session it happened in. Narratives do not transfer; predicates do.
- **Write the check, not the complaint.** A failure mode without the command or gate that catches it is not actionable.
- **Do not add a nuance clause to a rule that is working.** If there is a real exception, express it as its own conditional keyed to something observable.
- **Delete what is wrong.** If a gate never catches anything and costs a minute each run, remove it. Growth is not improvement.

## Verify the patch

The honest bar is a delegate run: dispatch a task that would have triggered the old failure and confirm the new wording prevents it. Where that is too slow, at minimum re-read the patched section as if you had never seen the failure and ask whether it would have changed your behaviour. If it would not, the wording is too soft.

## Known gaps in this skill

Kept current deliberately — an empty list here means nobody is looking.

- ~~Not pressure-tested against a single-delegate mechanical task, where the overhead may not pay for itself.~~ **Answered: it does not pay.** A single-file fix with the root cause already established in the ticket came in at ~8.8 KB of contract-plus-prompt for a 113-line diff, ~50k coordinator tokens against ~13k of delegate transcript avoided. This produced the size gate in SKILL.md. Note that the *tests* (95 of those 113 lines) delegated well — the floor applies to the deliverable as a whole, so a thin fix with broad test coverage can still qualify.
- No guidance yet on delegate count above two, or on delegates that must hand off sequentially rather than build in parallel to a contract.
- The coordinator tier table has not been tested at the Haiku end.
- The gates assume a delegate that finishes and reports. Gate 0 now catches the dead-session case, but the skill still has no guidance on a delegate that dies *repeatedly* on a task genuinely too large for one session — whether to split the task or escalate the tier.
- Untested: whether a delegate can reliably drive tracker/MCP work at all. The one attempt was blocked by a server outage, so the "offload the retry loop" advice in sdlc-delegation.md is reasoned, not observed.
