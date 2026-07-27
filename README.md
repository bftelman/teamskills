# teamskills - Microsoft Teams skill for agents

Drive Microsoft Teams (new web client) from your coding agent via the Playwright MCP and a real
Chrome, with no Microsoft Graph / API. Read and catch up, send, quote-reply, react (emoji), and
@mention. Chats only (no calls, files, or channel posts yet).

Cross-agent (Claude Code, Cursor, Copilot / VS Code, Codex, Windsurf, OpenCode) and cross-platform
(Windows, macOS, Linux). Packaged for the [`skills` CLI](https://skills.sh/) (`npx skills`).

## Install (MCP first, so it is ready before first use)

### Step 1: add the Playwright MCP

This is the skill's engine and must exist before you run the skill. Register a Playwright MCP server
pointed at a CDP debug endpoint. Always use `127.0.0.1`, never `localhost` (localhost resolves to
IPv6 `::1`, which the debug port refuses).

**Claude Code**
```bash
claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222
```

**Cursor** (`.cursor/mcp.json`), **Windsurf** (`~/.codeium/windsurf/mcp_config.json`), or
**Claude Desktop** (`claude_desktop_config.json`)
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--cdp-endpoint", "http://127.0.0.1:9222"]
    }
  }
}
```

**VS Code / GitHub Copilot** (`.vscode/mcp.json`, note the key is `servers`)
```json
{
  "servers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--cdp-endpoint", "http://127.0.0.1:9222"]
    }
  }
}
```

**Codex** (`~/.codex/config.toml`)
```toml
[mcp_servers.playwright]
command = "npx"
args = ["@playwright/mcp@latest", "--cdp-endpoint", "http://127.0.0.1:9222"]
```

**OpenCode** (`opencode.json`, or `~/.config/opencode/opencode.json`; note `command` is one array)
```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "playwright": {
      "type": "local",
      "command": ["npx", "@playwright/mcp@latest", "--cdp-endpoint", "http://127.0.0.1:9222"],
      "enabled": true
    }
  }
}
```

### Step 2: install the skills
```bash
npx skills add bftelman/teamskills
```
This installs two skills: `teams` (the capability) and `setup-teams` (the one-shot installer). In
Claude Code both are then invocable as `/teams` and `/setup-teams`. Options: add `-s teams` for just
the main skill, or pass a hosted URL to a `SKILL.md`. Manual alternative: copy the `teams/` and
`setup-teams/` folders into your agent's skills directory. (The root `commands/setup-teams.md` is an
optional native Claude Code slash-command variant; not needed if you installed the `setup-teams`
skill above.)

### Step 3: restart the agent
Once, so it loads the new MCP server and the skill.

## Setup: one command
Run `/setup-teams` (Claude Code) or say "set up Teams" (any agent). It launches Chrome, walks you
through the one-time Teams login, and seeds your most-used chats into `chats.md`. Because the MCP was
added in Step 1, this runs straight through with no mid-setup restart.

## What is in the package
| File | Purpose |
|---|---|
| `teams/SKILL.md` | The skill: connect, navigate, read, send, reply, react, @mention, safety gate. |
| `teams/selectors.md` | Verified Teams `data-tid` selectors and flows. |
| `teams/chats.md` | Per-user friendly-name to chat-id map (ships empty; seed via "set up Teams"). |
| `teams/teams-chrome.ps1` | Windows launcher for debug Chrome (dedicated profile, IPv4 :9222). |
| `teams/teams-chrome.sh` | macOS / Linux launcher (Chrome, Chromium, or Edge). |
| `teams/INSTALL.md` | Setup detail: per-agent MCP config, first-run login. |
| `setup-teams/SKILL.md` | The one-shot installer skill (auto-installed; invoke as `/setup-teams`). |
| `commands/setup-teams.md` | Optional native Claude Code slash-command variant (manual copy). |

## Safety
The skill confirms before every send, react, and @mention (they notify real people). Test safely
against Notes to self (`48:notes`).

## Requirements
The `skills` CLI needs Node 22.20 or newer. Runtime needs the Playwright MCP from Step 1 and a
Chrome/Chromium/Edge install.
