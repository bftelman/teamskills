---
description: One-shot setup for the Teams skill — configures the Playwright MCP, launches debug Chrome, handles first-run login, and seeds your most-used chats.
allowed-tools: Bash, PowerShell, Read, Edit, Write, Skill
---

You are running the **Teams skill setup**. Do it end-to-end with as few asks as possible. Detect the
OS and the host agent; adapt paths accordingly. The skill lives at `<skill-dir>` (Claude Code:
`~/.claude/skills/teams`). Read `<skill-dir>/SKILL.md` if you need detail.

Run these steps in order, skipping any already satisfied (idempotent):

1. **Playwright MCP present?** Check whether Playwright browser tools are available this session
   (try to load/see `browser_navigate`). 
   - If NOT available: register the server, then STOP and tell the user to restart the agent and
     re-run `/setup-teams` (MCP servers load at startup).
     - Claude Code: `claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint http://127.0.0.1:9222`
     - Other agents: add that command+args under `mcpServers` in their MCP config, endpoint
       `http://127.0.0.1:9222` (IPv4 — never `localhost`).
   - If available: continue.

2. **Launch/ensure debug Chrome** by running the OS launcher (no user action):
   - Windows: `pwsh -File "<skill-dir>/teams-chrome.ps1"` (fallback `powershell -File`).
   - macOS/Linux: `bash "<skill-dir>/teams-chrome.sh"`.
   It's idempotent and waits for `127.0.0.1:9222`.

3. **Verify connection + sign-in.** `browser_navigate` to `https://teams.cloud.microsoft/v2/`, wait
   for load, check `document.title`.
   - If `ECONNREFUSED …:9222`: re-run the launcher once, retry.
   - If the Microsoft sign-in page shows: ask the user to log into Teams **once** in the Chrome
     window that opened (dedicated profile; persists after). Wait for them, then continue. Never
     enter credentials yourself.

4. **Seed chats.md** using the browser bootstrap in SKILL.md → "Setup — seed chats.md from the chat
   list" (reads the user's favorites+recents from the DOM; no API). Show the name→id list, ask which
   to keep (default: all), and write those rows into `<skill-dir>/chats.md`. Offer to scroll for more.

5. **Confirm done.** Summarize: MCP configured, Chrome connected, N chats seeded. Tell the user they
   can now say things like "check Teams", "message the X chat", "react to that", "@mention Y" — and
   that every send/react/mention is confirmed first (safe test target: Notes to self).

Keep it tight. Only pause for the two things that genuinely need the human: the restart-after-MCP-add
(step 1) and the one-time Teams login (step 3).
