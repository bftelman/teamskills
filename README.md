# teams — Microsoft Teams skill for agents

Drive Microsoft Teams (new web client) from your coding agent via the Playwright MCP and a real
Chrome — **no Microsoft Graph / API**. Read & catch up, send, quote-reply, react (emoji), and
@mention. Chats only (no calls/files/channel posts yet).

**Cross-agent** (Claude Code, Cursor, Copilot, Codex, Windsurf, …) and **cross-platform**
(Windows / macOS / Linux). Packaged for the [`skills` CLI](https://skills.sh/) (`npx skills`).

## Install (MCP first, so it's ready before first use)

**Step 1 — add the Playwright MCP** (the skill's engine; must exist before you run the skill):
```bash
# Claude Code:
claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222
```
Other agents: add the same command+args under `mcpServers` in their MCP config
(`.cursor/mcp.json`, `.vscode/mcp.json`, `~/.codeium/windsurf/mcp_config.json`, …). Use
`127.0.0.1`, **never `localhost`** (localhost → IPv6 `::1`, which the debug port refuses).

**Step 2 — install the skill:**
```bash
npx skills add <owner>/<repo>          # or: -s teams   |  or a hosted URL to teams/SKILL.md
```
Claude Code slash command: also copy `commands/setup-teams.md` → `~/.claude/commands/`.
(Manual alternative: copy `teams/` into your agent's skills dir.)

**Step 3 — restart the agent** once, so it loads the new MCP server and skill.

## Setup — one command
Now run **`/setup-teams`** (Claude Code) or say **"set up Teams"** (any agent). It launches Chrome,
walks you through the one-time Teams login, and seeds your most-used chats into `chats.md`. Because
the MCP was added in Step 1, this runs straight through — no mid-setup restart.

> The `skills` CLI needs Node ≥ 22.20.

## What's in the package
| File | Purpose |
|---|---|
| `teams/SKILL.md` | The skill: connect, navigate, read, send, reply, react, @mention, safety gate. |
| `teams/selectors.md` | Verified Teams `data-tid` selectors + flows. |
| `teams/chats.md` | Per-user friendly-name → chat-id map (ships empty; seed via "set up Teams"). |
| `teams/teams-chrome.ps1` | Windows launcher for debug Chrome (dedicated profile, IPv4 :9222). |
| `teams/teams-chrome.sh` | macOS/Linux launcher (Chrome/Chromium/Edge). |
| `teams/INSTALL.md` | Setup: MCP config (per agent), first-run login. |
| `commands/setup-teams.md` | Claude Code `/setup-teams` slash command — copy to `~/.claude/commands/`. |

## Publishing this repo
```bash
git init && git add . && git commit -m "teams skill"
git remote add origin git@github.com:<owner>/<repo>.git
git push -u origin main
# share:  npx skills add <owner>/<repo>
```

## Safety
The skill confirms before every send / react / @mention (they notify real people). Test safely
against **Notes to self** (`48:notes`).
