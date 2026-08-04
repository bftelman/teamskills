# Coordinate skill — install & setup

Delegates implementation work to `opencode` agents running a cheap model, while your main agent stays
the reviewer and holds the commit/push gate.

## 1. Install the skill

- **Via the `skills` CLI (any supported agent):** `npx skills add bftelman/skills -s coordinate`
  (needs Node ≥ 22.20).
- **Manual:** copy the `coordinate/` folder into your agent's skills directory — Claude Code
  `~/.claude/skills/coordinate/` (or `<repo>/.claude/skills/coordinate/`); other agents use their own
  path. Keep `references/` alongside `SKILL.md`; the skill links to it.

For the `/coordinate` slash command in Claude Code, also copy `commands/coordinate.md` into
`~/.claude/commands/`.

## 2. Install opencode

```bash
npm install -g opencode-ai
opencode auth login      # pick a provider and sign in
```

Verify it works and see what models the install actually offers:

```bash
opencode --version
opencode models
```

The skill defaults delegates to `opencode/deepseek-v4-flash-free`. If `opencode models` does not list
it, pick the cheapest capable model it does list and pass that with `--model`.

## 3. Check the CLI flags on your version

The skill relies on `run --auto`, `--dir`, `--continue`, `--model`, and `--title`. Confirm they exist:

```bash
opencode run --help
```

Flag names have changed across opencode releases. If one is missing, the dispatch recipe in
`references/dispatch.md` needs adjusting rather than the skill's logic.

## 4. First run

Invoke `/coordinate <TICKET>` (Claude Code) or say "coordinate this ticket — delegate the
implementation to opencode". The coordinator will read the ticket, freeze a contract if more than one
delegate is involved, dispatch, and come back to you with what it verified.

Nothing is committed or pushed until you say so.

## Notes

- **One delegate per repository.** Two delegates in the same working tree will fight over the index.
- **`--auto` auto-approves tool permissions** for the delegate. It is scoped to the `--dir` repo, and
  the dispatch prompt is what keeps it in scope — review the diff, not just the report.
- **Logs are large** (20–40k tokens each). The skill greps them; it does not read them. Point them at a
  scratchpad directory, not the repo.
- **Delegates run the lifecycle steps** (commit, push, MR, ticket transitions) but only after you
  approve. See `references/sdlc-delegation.md`.
