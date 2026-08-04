# Review gates

Run these in order on every report, before and after each fix round. Stop at the first failure and send a fix round.

## Gate 0 — did the delegate actually finish?

`opencode run` **exits 0 on a session that died mid-run.** The exit code proves nothing.

```bash
ls <scratchpad>/report.md          # the report you asked for in the prompt
git -C <repo> status --porcelain   # must be non-empty
```

No report file, or an empty tree, means the session died — there is nothing to review, and every later gate would be reviewing claims that were never made. Resume with `--continue` (the delegate keeps its research context, so this is cheap) rather than re-dispatching from scratch.

A session can also die *after* writing files but *before* reporting. So when the report is missing but the tree is not, do not assume the work is partial in the way it looks — re-check the tree after the resume completes, because a still-running session will keep writing while you are reading.

## Gate 1 — files outside scope

```bash
git -C <repo> diff --stat
git -C <repo> status --porcelain
```

Compare against the report's file list and against the ticket's scope. Anything unrelated — editor settings, lockfiles, generated swagger, formatting-only churn — gets reverted before anything else is considered:

```bash
git -C <repo> checkout -- <unrelated-path>
```

**Re-run this gate immediately before the commit, not only after the report.** Your own verification builds mutate the tree: checked-in generated artifacts (swagger, lockfiles) get rewritten by every build you run, and crash dumps appear as untracked files. Churn you created yourself is still churn in the MR, and it is the coordinator, not the delegate, who put it there. One observed run regenerated 15 generated files across services nobody touched, purely from a build and a test invocation.

## Gate 2 — read the diff of the entry points

Not the whole diff. The controller action, the exported function, the switch case — whatever the outside world calls. This is where shape errors live: code after a response has been completed, a guard in the wrong order, a status never returned.

## Gate 3 — dead and abandoned code

Delegates iterate and leave earlier attempts behind. For every new non-private member in the diff, confirm something calls it:

```bash
grep -rn "<NewMethodName>" <repo>/src | grep -v "<file-it-is-defined-in>"
```

No caller and not on an interface means it is an abandoned iteration. Delete it.

## Gate 4 — run the build and the full test suite yourself

Unfiltered. Not the delegate's filtered subset, not `--no-build` over a stale binary. Compare the real counts to the reported ones. A discrepancy is not a rounding error; it means the delegate reported a run you cannot reproduce.

## Gate 5 — cross-check the contract between sides

Put the two sides' actual code next to each other — the route the backend registered against the path the frontend builds; the field names in the DTO against the query the client emits. Each side claiming "matches the contract" is not evidence that they match each other.

## Gate 6 — the facts you verified came from the right tree

Before treating a grep result as evidence, confirm which branch or worktree you read it from. A fact read off a neighbouring feature branch will contradict the delegate's correct finding and send you chasing a phantom conflict. When a delegate's evidence contradicts yours, re-check your own source first.

## Observed delegate failure modes

Real cases. Each row is a claim that passed casual reading and failed a gate.

| Reported | Actual | Caught by |
|---|---|---|
| "150 tests passed" | 152, then 153 after the fix round | Gate 4 |
| "151 passed, excluding long-running OOM test" | No such test; full unfiltered suite clean | Gate 4 |
| File list of 6 changed files | Also deleted an unrelated `.vscode/settings.json` | Gate 1 |
| "No deviations from the contract" | An 85-line method left in the service, on no interface, never called | Gate 3 |
| "Streaming endpoint, 404 handled" | `Response.StatusCode` assigned after `Response.CompleteAsync()` — throws, and unreachable anyway | Gate 2 |
| "Existing cases already use base X" (contradicting the coordinator) | Delegate was right; the coordinator's grep came from another branch | Gate 6 |
| Exit code 0, no report, no message | Session died mid-run; working tree completely untouched | Gate 0 |
| Exit code 0, one file changed, no report | Session was *still running* and wrote the rest during the review | Gate 0 |
| Code that "builds" | Two `CS0103` errors: `nameof(X)` where X is a member of the injected interface, not of the class, and `{Param}` capitalised inside an interpolated string (valid in a structured-log template, a compile error in `$"..."`) | Gate 4 — the delegate never reached a build |
| Diff of 2 intended files | Plus 15 regenerated `*.swagger.json` and a `bash.exe.stackdump`, all caused by the **coordinator's** verification builds | Gate 1, re-run before commit |

When you hit a failure mode that is not in this table, add a row. That is the mechanism, not an afterthought — see [self-improvement.md](self-improvement.md).

## Writing the fix round

Number the findings. Per finding: what is wrong, where, why it is wrong, and what the fix is. Then:

- Restate what is already verified and must not change — the route, the contract, the passing tests — so the delegate does not "improve" a settled interface while fixing a bug.
- Mark anything optional as optional, with a bail-out condition ("skip it if it starts to sprawl"). Delegates otherwise treat a nice-to-have as mandatory and balloon the diff.
- Require a per-item report, including items it disagrees with and why. A delegate that silently skips a finding is worse than one that argues.
