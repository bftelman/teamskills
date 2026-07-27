# Teams skill — install & setup

Drives Microsoft Teams (new web client, `teams.cloud.microsoft`) from Claude Code via the Playwright
MCP and a real Chrome. No Microsoft Graph / API. Capabilities: read/catch-up, send, quote-reply,
react (emoji), @mention. Chats only (no calls/files/channel posts).

Works across agents (Claude Code, Cursor, Copilot, Codex, Windsurf, …) and OSes (Windows, macOS,
Linux). The skill uses the Playwright MCP's standard `browser_*` tools; only the Chrome launcher is
OS-specific.

## 1. Install the skill
- **Via the `skills` CLI (any supported agent):** `npx skills add <owner>/<repo>` (needs Node ≥ 22.20).
- **Manual:** copy the `teams/` folder into your agent's skills directory — Claude Code
  `~/.claude/skills/teams/` (or `<repo>/.claude/skills/teams/`); other agents use their own path.

Files: `SKILL.md`, `selectors.md`, `chats.md` (empty template — fill per user), `teams-chrome.ps1`
(Windows), `teams-chrome.sh` (macOS/Linux).

## 2. Add the Playwright MCP (once), attached to a debug Chrome on IPv4
Register a Playwright MCP server with the CDP endpoint in your agent's MCP config. Command/args:
```
npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222
```
Use `127.0.0.1`, NOT `localhost` (localhost → IPv6 `::1`, which the debug port refuses).
- **Claude Code:** `claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222`
- **Cursor / VS Code / Windsurf / Codex / OpenCode:** add the same command+args in that agent's MCP
  config (`.cursor/mcp.json`, `.vscode/mcp.json` [key `servers`], `~/.codeium/windsurf/mcp_config.json`,
  `~/.codex/config.toml` [TOML], `opencode.json` [`mcp`, `command` as an array]). See the top-level
  README for a ready-to-paste block per agent.

Restart the agent so it spawns the server.

## 3. First run
Just ask, e.g. *"check Teams"* or *"message the X chat"*. The skill will:
- Launch debug Chrome itself via `teams-chrome.ps1` (dedicated profile — never touches your main
  Chrome; binds `127.0.0.1:9222`).
- First time only: it opens Teams' sign-in — **log in once** in that Chrome window. The session
  persists for every later run.

## 4. Seed your chats (optional, recommended)
Say **"set up my chats"**. The skill reads your own chat list (favorites + recents) from the browser
— no API — and writes your most-used chats into `chats.md` so it can jump to them by name. You can
also just reference any chat by name anytime and let it append the id.

## Notes
- **Chrome launcher is auto-selected by OS:** `teams-chrome.ps1` (Windows) / `teams-chrome.sh`
  (macOS/Linux, also finds Chromium/Edge). The skill runs the right one; you don't launch anything.
- **Safety**: the skill confirms with you before every send/react/@mention (they notify real people).
  Test safely against **Notes to self** (`48:notes`).
- **Selectors** in `selectors.md` were verified 2026-07-27; Teams updates its DOM occasionally — the
  skill self-heals (re-discovers a `data-tid` and updates the file) when one misses.
- `chats.md` starts empty — it fills in as you use named chats, or seed it from `teams_list_chats`.
