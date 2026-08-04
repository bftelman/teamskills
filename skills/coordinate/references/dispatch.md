# Dispatching delegate agents

## The command

```bash
opencode run --auto --dir <absolute-repo-path> \
  --model opencode/deepseek-v4-flash-free \
  --title "<TICKET> <side>" \
  "<prompt>"
```

- `--auto` auto-approves permissions. Required for non-interactive runs; the prompt is what keeps the agent in scope.
- `--dir` sets the repo. One delegate per repo — never point two delegates at the same working tree.
- `--model` defaults to deepseek (`opencode/deepseek-v4-flash-free`). Check what this install actually has with `opencode models`; override when the user names a model.
- `--continue` resumes the last session in that directory. Use it for every fix round — the delegate keeps its context, so the round is cheap and it does not re-read the repo.
- Run it in the background and redirect to a log file. Never read the log wholesale; grep it (`## REPORT`, `Passed!`, `error`) when a report looks off.

Ask each delegate to write its report to a scratchpad path as well as ending its final message with it, so you can read a 1k file instead of tailing a log.

## Model choice for delegates

Cheap by default. Deepseek handles house-pattern implementation, tests, and mechanical sweeps. Raise the delegate tier only when a fix round shows it cannot hold the problem — two failed rounds on the same finding is the signal, not a hunch.

### Session death is not incapacity — treat them differently

Two failure shapes look alike in a log and call for opposite responses:

| Shape | What it means | Response |
|---|---|---|
| Delegate produces wrong code, twice, on the same finding | It cannot hold the problem | Raise the tier |
| Session ends mid-run — truncated log, no report, exit 0 | Infrastructure, not comprehension | `--continue`; the research context survives |

Free tiers die silently and often. A resume is cheap precisely because the delegate has already read the repo, so prefer it over re-dispatching. But cap it: **after two deaths on the same task, switch model or take the work back.** Three truncated sessions on one small task cost more than writing it would have.

When you resume after a death, say explicitly in the prompt that nothing landed and tell it to confirm with `git diff --stat` — otherwise it may assume its earlier edits succeeded and build on work that does not exist.

### Tool-call mechanics are a real failure mode on cheap tiers

Delegates lose whole sessions to exact-string edit failures on files with CRLF endings or irregular indentation. If a delegate reports or shows a failed edit, tell it to anchor on a larger unique block, or to rewrite the file wholesale, rather than retrying ever-smaller snippets.

## The prompt recipe

A dispatch prompt is these parts, in this order:

1. **Role and context** — which ticket, which side, and what other delegates are doing in parallel right now.
2. **Repo and branch** — the branch already exists; tell it to verify with `git rev-parse --abbrev-ref HEAD` and not create another.
3. **House rules first** — "read this repo's AGENTS.md / CLAUDE.md and follow its conventions; match surrounding code."
4. **The frozen contract, verbatim** — plus: do not deviate; if a clause is impossible here, stop and report the conflict, because the other side is already building to it.
5. **Why the contract has the shape it does** — one line of reasoning per constraint that would otherwise look arbitrary. A delegate that understands the constraint stops trying to "improve" past it.
6. **Reuse pointers** — the exact files to read first, and what already exists that it must extract or call rather than reimplement.
7. **Out of scope** — the named files, flows, and endpoints it must not touch.
8. **Verification** — build and test commands to run, told as "discover the correct command from the repo" rather than a guessed invocation.
9. **Definition of done and the gate** — implemented, builds, tests pass, then STOP. No commit, no push, no MR, no Jira changes until the coordinator approves.
10. **The report contract** — end the final message with a `## REPORT` section containing named slots (below), and be honest about failures.

## Required report slots

Ask for exactly these, so reports are comparable and cheap to check:

1. The interface it implemented, verbatim (route and query shape, function signature, schema)
2. Response/return behaviour, including error cases
3. Every file created or changed
4. The exact build and test commands run, with real pass/fail counts
5. Any deviation from the contract, and any open question for the other delegate

## Parallel work

Dispatch all delegates in one go; do not serialize a frontend behind a backend when a frozen contract lets both build at once. The contract is what makes parallelism safe — without it, parallel delegates produce two incompatible halves and you pay for both.

Assign each open question to exactly one delegate, and say in the other's prompt that the question is being answered elsewhere and its own shape does not change either way.
