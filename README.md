# claude-skills — agent skills by @bftelman

Personal, cross-agent skills for coding agents. Each top-level folder is one skill family, packaged for
the [`skills` CLI](https://skills.sh/) (`npx skills`) and usable as a plain copy into any agent's skills
directory.

| Family | Skills | What it gives your agent |
|---|---|---|
| [`coordinate/`](coordinate/) | `coordinate` | Delegate implementation to cheap `opencode` agents and act as the reviewing tech lead — contract-first parallel work, verification gates, and a hard commit/push approval gate. |
| [`teams/`](teams/) | `teams`, `setup-teams` | Drive Microsoft Teams (new web client) via the Playwright MCP and a real Chrome — read, catch up, send, quote-reply, react, @mention. No Microsoft Graph / API. |

Cross-agent (Claude Code, Cursor, Copilot / VS Code, Codex, Windsurf, OpenCode) and cross-platform
(Windows, macOS, Linux).

## Install

```bash
npx skills add bftelman/claude-skills                    # everything
npx skills add bftelman/claude-skills -s coordinate      # one skill
npx skills add bftelman/claude-skills -s teams
```

Needs Node ≥ 22.20. **Manual alternative:** copy the individual skill folder (the one containing
`SKILL.md`) into your agent's skills directory — Claude Code `~/.claude/skills/<name>/`, other agents
use their own path.

Each family has its own `INSTALL.md` with prerequisites and first-run steps. Read it before first use:

- **[coordinate/INSTALL.md](coordinate/INSTALL.md)** — needs the `opencode` CLI and a model provider.
- **[teams/INSTALL.md](teams/INSTALL.md)** — needs the Playwright MCP pointed at a debug Chrome, plus a
  one-time Teams login.

Claude Code slash-command variants live in [`commands/`](commands/); copy them into `~/.claude/commands/`
if you want `/coordinate` and `/setup-teams` as native commands.

## Layout

```
coordinate/
  SKILL.md            # the skill
  INSTALL.md
  references/         # dispatch recipe, review gates, SDLC delegation, self-improvement
teams/
  teams/SKILL.md      # the capability
  setup-teams/SKILL.md# one-shot installer
  INSTALL.md
commands/             # optional native Claude Code slash commands
```

## Safety

Both families gate actions that reach the outside world. `teams` confirms before every send, react, and
@mention, because they notify real people. `coordinate` never lets a delegate commit, push, or open a
merge request without explicit approval, and never accepts a delegate's self-reported test results as
verification.
