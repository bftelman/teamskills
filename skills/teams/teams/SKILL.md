---
name: teams
description: Use when the user wants to read, catch up on, send, reply to, react to, or @mention in Microsoft Teams chat messages ("check Teams", "any messages in <chat>", "reply in <chat>", "message the <X> chat", "react to that message", "@mention someone", "catch me up on Teams"). Drives the new Teams web (teams.cloud.microsoft) in Chrome via the Playwright MCP — no Graph/API. Chat ids in chats.md; verified selectors in selectors.md.
---

# Teams (Playwright MCP)

Operate Microsoft Teams by driving Chrome/Chromium with the **Playwright MCP**. No Graph / MS365 API.
Runs on the **new Teams web** (`https://teams.cloud.microsoft/v2/`).

**Capabilities (verified working):** read / catch-up, send, quote-reply, react (emoji), @mention.
**Not built:** calls, file share/download, channel (team) posts. Chats only.

**Cross-agent / cross-platform.** The steps use the Playwright MCP's own tool names
(`browser_navigate`, `browser_snapshot`, `browser_evaluate`, `browser_type`, `browser_press_key`,
`browser_hover`, `browser_wait_for`). Your agent may namespace them — e.g. Claude Code exposes them
as `mcp__playwright__browser_*`, other agents (Cursor, Copilot, Codex, Windsurf…) expose the same
tools under their own prefix. Map the bare names below to whatever your runtime calls them. The
`browser_evaluate` (run-JS) tool is required — most flows here act via `document.querySelector(...)`
to stay reliable and cheap. Chrome launch is OS-specific (see Step 0); everything else is identical
on Windows, macOS, and Linux.

## Core rules

1. **Safety Gate — never send/react without confirmation.** Show the target chat + exact text (or
   emoji + which message) and WAIT for explicit approval. Override only if the user said "just send
   it" for that specific message. Re-confirm each new message. Reactions and @mentions notify people
   too — gate them. Safe end-to-end test target: **Notes to self** (`48:notes`).
2. **Navigate by search-and-click**, not hash-URL (the SPA ignores `#`-only navigations).
3. **Prefer targeted `browser_evaluate` over `browser_snapshot`.** Teams snapshots are huge and the
   page animates constantly (Playwright `click`/`hover`/`type` often time out on the "stable" check).
   The reliable pattern is `browser_evaluate` with `document.querySelector(...).click()` — it bypasses
   the stability wait. Use `browser_type` only for the compose box and search input; use screenshots
   only when the user wants to see something.
4. **Verify the open chat via `document.title`** — the segment before ` | Microsoft Teams` is the
   chat name. (`[data-tid="chat-header-title"]` does NOT exist — don't rely on it.)

## First-time setup (one command)

New user / empty `chats.md`? Don't walk them through steps manually — run the automated setup:
- **Claude Code:** `/setup-teams` (command bundled with this skill).
- **Any agent:** the user says **"set up Teams"** → do it yourself: perform Step 0 (configure MCP if
  needed, launch Chrome, verify, one-time login), then the chat-seeding bootstrap
  ("Setup — seed chats.md…"), then confirm. Only pause for the two unavoidable human moments: an
  agent restart after adding the MCP server, and the one-time Teams login. Everything else is
  automatic and idempotent.

## Step 0 — Connect (before anything, every run)

1. **Ensure debug Chrome is up and attached.** Try a light `browser_navigate` to
   `https://teams.cloud.microsoft/v2/`. If it errors with `ECONNREFUSED ...:9222`, the browser's
   debug port is down — **launch it yourself** (no user action), picking the launcher for the OS
   (detect via the environment / `uname`):
   - **Windows:** `pwsh -File "<skill-dir>/teams-chrome.ps1"` (or `powershell -File …`).
   - **macOS / Linux:** `bash "<skill-dir>/teams-chrome.sh"` (`chmod +x` once if needed).
   Both are idempotent (exit 0 if already up), use a **dedicated** profile that never touches the
   user's main browser, bind IPv4 `127.0.0.1:9222`, and wait for the port. Then retry `browser_navigate`.
   `<skill-dir>` is wherever this skill is installed (Claude Code: `~/.claude/skills/teams`; other
   agents use their own skills path).
   - The Playwright MCP server must be configured to attach over CDP:
     `--cdp-endpoint http://127.0.0.1:9222` (IPv4, not `localhost` → resolves to IPv6 `::1` and
     refuses). This is a one-time per-agent MCP config the skill can't self-heal. Examples:
     Claude Code → `claude mcp add -s user playwright -- npx @playwright/mcp@latest --cdp-endpoint
     http://127.0.0.1:9222`; other agents → add the same command/args in their MCP config
     (`.cursor/mcp.json`, `.vscode/mcp.json`, `~/.codeium/windsurf/mcp_config.json`, etc.). If the
     endpoint is still `localhost`, tell the user to switch it to `127.0.0.1` and restart.
   - **The connection drops when the user closes that browser window.** If any action mid-run throws
     `ECONNREFUSED`, re-run the launcher and re-navigate — the dedicated profile stays logged in.
2. **Confirm sign-in.** After navigate, wait for load (`browser_wait_for` textGone
   "We're setting things up for you", or ~5s) and check `document.title`. If it's the Microsoft
   sign-in page / no chat list, the dedicated profile isn't signed in yet: ask the user to log into
   Teams once in that Chrome window (persists for every later run). **Never enter credentials.**

## Setup — seed chats.md from the chat list (first run, no API)

When `chats.md` is empty (or the user says "set up my chats" / "import my chats"), bootstrap it
straight from the browser — no MS365/Graph needed. The left chat list (favorites + recents) exposes
each chat's id in its DOM. `browser_evaluate`:
```js
() => {
  const idRe=/19:[^"'\s]+@thread\.v2|48:notes|19:[0-9a-f-]+_[0-9a-f-]+@unq\.gbl\.spaces/;
  return [...document.querySelectorAll('[role="tree"] [role="treeitem"]')].map(el=>{
    let id=null;
    for(const n of [el,...el.querySelectorAll('*')]){ for(const a of n.attributes||[]){ const m=(a.value||'').match(idRe); if(m){id=m[0];break;} } if(id)break; }
    const leaf=[...el.querySelectorAll('*')].find(n=>n.children.length===0 && n.textContent.trim());
    return {name: leaf?leaf.textContent.trim():'', id};
  }).filter(x=>x.id && x.name);
}
```
Show the user the resulting name→id list, let them keep the ones they want (their "most used"), and
write those rows into `chats.md`. Scroll the chat list and re-run to capture more. This is the
recommended onboarding step; the MS365 `teams_list_chats` tool is an optional alternative when present.

## Navigate to a chat (search-and-click — PRIMARY)

1. Type the chat/person name into the search box with **real keystrokes**:
   `browser_type` target `input[data-tid="AUTOSUGGEST_INPUT"]`, `slowly: true` (a plain `fill()`
   does NOT trigger suggestions). If the box has leftover text, `fill('')` first, then type slowly.
2. `browser_wait_for` ~2s, then read results with `browser_evaluate`:
   `[...document.querySelectorAll('[data-tid^="AUTOSUGGEST_SUGGESTION_"]')]` → each result's
   `data-tid` **embeds the target id**:
   - Group/meeting chat: `…TOPHITS<chatId>` or `…GROUPCHATS<chatId>` where `<chatId>` = `19:…@thread.v2`.
   - 1:1 with a person: the person hit `…TOPHITS8:orgid:<userGuid>` (opens/creates the 1:1).
3. Click by **id suffix** (matches both result types):
   `document.querySelector('[data-tid$="<id>"]').click()` via `browser_evaluate`.
4. **Confirm** `document.title` shows the expected chat. Then dismiss the search flyout (below).
5. Fallback if a name isn't in `chats.md`: same flow, just read the human-readable result labels to
   pick the right one; offer to add the resolved id to `chats.md`.

Deep-link (`…/#/conversations/<id>?ctx=chat`) works ONLY on a fresh full page load, not for
switching chats in a loaded session — don't rely on it.

### Dismiss the search flyout (after selecting)

The suggestions flyout lingers over the compose box. Clear it via `browser_evaluate`:
```js
const s=document.querySelector('input[data-tid="AUTOSUGGEST_INPUT"]');
if(s){ s.blur(); ['keydown','keyup'].forEach(t=>s.dispatchEvent(new KeyboardEvent(t,{key:'Escape',keyCode:27,bubbles:true}))); }
document.querySelector('[data-tid="ckeditor"]')?.focus();
```

## Read / catch up

`browser_evaluate`: scroll the pane to bottom and read `innerText`, e.g.
`document.querySelector('[data-tid="message-pane-list-viewport"]')` (fallback `[role="main"]`) →
`.scrollTop = .scrollHeight`, then return the last ~30–40 lines of `.innerText`. Rendered text
interleaves author + timestamp + body ("Ruslan Hagverdi 5:43 PM Qosdun?"), which is enough to
summarize who said what and what needs a reply. Reading marks messages read — say so if it matters.

## Send a message

1. Navigate to the chat; confirm via `document.title`.
2. `browser_type` into `[data-tid="ckeditor"]` (contenteditable, aria "Type a message").
3. Verify text via `browser_evaluate` (`ckeditor.textContent`).
4. **SAFETY GATE.**
5. Send: focus editor, `browser_press_key` `Enter`.
6. **Verify posted:** `browser_evaluate` — editor is empty AND the message text appears in the pane.

## Quote-reply to a specific message

1. Find the message: `[...document.querySelectorAll('[data-tid="chat-pane-message"]')]`, match on
   text (last match = most recent); tag it (`el.setAttribute('data-cc-target','x')`), `scrollIntoView`.
2. Open its menu: click `[data-tid="message-actions-menu-hidden-button"]` (dispatch
   `mouseenter/over/move` first). Via `browser_evaluate .click()`.
3. Click the menu item whose label matches `/reply with quote|^reply/i` (role `menuitem`).
4. The compose box now shows the quoted reference. Type the reply into `[data-tid="ckeditor"]`,
   Safety Gate, then send (Enter) and verify.

## React with emoji

1. Find + tag the target message (as above). Open `[data-tid="message-actions-menu-hidden-button"]`.
2. The menu's top row is the reactions bar. Click one via `browser_evaluate .click()`:
   `[data-tid="message-actions-like"]` (👍) / `-heart` / `-laugh` / `-surprised`, or
   `[data-tid="add-reaction-picker-entry-point-button"]` for the full picker.
3. Safety Gate (confirm target + emoji) before clicking.
4. **Verify:** the message now has `[data-tid="diverse-reaction-summary"]` / "N … reaction" text.

## @mention someone

1. Focus `[data-tid="ckeditor"]`. Type `@<name>` with **real keystrokes**
   (`browser_type slowly: true`) — the picker needs keystrokes.
2. `browser_wait_for` ~2s. The picker is `[data-tid="AutocompletePopup-Mentions"]`; each person is
   `[data-tid="autocomplete-picker-item-<Full Name>"]` (role `option`); `…-Everyone` notifies all.
3. Click the person option (`browser_evaluate .click()`). It inserts a real tagged mention:
   `<readonly itemtype="http://schema.skype.com/Mention"><mention mri="8:orgid:<guid>">Name</mention></readonly>`
   — not plain text.
4. Continue typing the rest of the message, Safety Gate, send + verify. (If asked NOT to send, leave
   it or `fill('')` to clear the draft.)

## Red flags — STOP

- About to send/react/insert-a-notifying-mention before the user said yes → STOP, run the gate.
- `ECONNREFUSED …:9222` → relaunch `teams-chrome.ps1`, re-navigate; don't give up.
- Sign-in page after navigate → STOP, ask the user to log into Teams in that window (never auto-login).
- A hardcoded selector missed → discover the real one via `browser_evaluate` and self-heal `selectors.md`.

## Files

- `chats.md` — friendly-name → chat-id map (per-user; seed via the "set up my chats" bootstrap).
- `selectors.md` — verified `data-tid` selectors + the flows above.
- `teams-chrome.ps1` — Windows launcher for debug Chrome (dedicated profile, IPv4 9222).
- `teams-chrome.sh` — macOS/Linux launcher (same behavior).
