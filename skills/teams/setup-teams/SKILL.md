---
name: setup-teams
description: Use to set up or reconfigure the Teams skill in one shot - configures the Playwright MCP, launches debug Chrome, handles the one-time Teams login, and seeds the user's most-used chats. Triggers on "set up Teams", "setup teams", "/setup-teams", "configure the teams skill", or when the teams skill's chats.md is empty on first use.
---

# Setup Teams (one-shot)

Set up the companion **teams** skill end-to-end with as few asks as possible. Detect the OS and host
agent; adapt paths. The teams skill lives at `<skill-dir>` (Claude Code: `~/.claude/skills/teams`);
read its `SKILL.md` for detail. Run in order, skipping anything already satisfied (idempotent).

1. **Playwright MCP present?** Check whether Playwright browser tools are available this session
   (try to see/load `browser_navigate`).
   - If NOT available: register the server, then STOP and tell the user to restart the agent and
     re-run setup (MCP servers load at startup).
     - Claude Code: `claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222`
     - Other agents: add that command+args under `mcpServers` (VS Code: `servers`) in their MCP
       config, endpoint `http://127.0.0.1:9222` (IPv4, never `localhost`).
   - If available: continue.

2. **Launch/ensure debug Chrome** by running the OS launcher (no user action):
   - Windows: `pwsh -File "<skill-dir>/teams-chrome.ps1"` (fallback `powershell -File`).
   - macOS/Linux: `bash "<skill-dir>/teams-chrome.sh"`.
   Idempotent; waits for `127.0.0.1:9222`.

3. **Verify connection + sign-in.** `browser_navigate` to `https://teams.cloud.microsoft/v2/`, wait
   for load, check `document.title`.
   - `ECONNREFUSED …:9222` -> re-run the launcher once, retry.
   - Microsoft sign-in page -> ask the user to log into Teams once in that Chrome window (dedicated
     profile; persists after). Wait, then continue. Never enter credentials yourself.

4. **Seed chats.md** via the teams skill's browser bootstrap ("Setup - seed chats.md from the chat
   list"): read the user's favorites+recents from the DOM (no API), show the name->id list, ask which
   to keep (default all), and write those rows into `<skill-dir>/chats.md`.

5. **Confirm done.** Summarize: MCP configured, Chrome connected, N chats seeded. Note the user can
   now say "check Teams", "message the X chat", "react to that", "@mention Y", and that every
   send/react/mention is confirmed first (safe test target: Notes to self).

Only pause for the two things that truly need the human: the restart after adding the MCP (step 1)
and the one-time Teams login (step 3).
