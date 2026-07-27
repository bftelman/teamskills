# Chats (per-user — fill this in)

This file is a per-user lookup: friendly name → Teams chat id, so the skill can jump straight to a
chat. **It ships empty on purpose** — chat ids are personal. Populate it for the current user.

## How to populate
- **Recommended — one-shot setup (no API):** say **"set up my chats"** (or, on first use, the skill
  offers it when this file is empty). It reads your own Teams chat list (favorites + recents) from the
  browser, shows you the name→id list, and writes the ones you pick here. See SKILL.md → "Setup —
  seed chats.md from the chat list".
- **Organic:** just ask for a chat by name — the skill searches, opens it, and offers to append the
  resolved id here.
- **Bulk seed (optional):** if the MS365 `teams_list_chats` MCP tool is available, list chats once
  and copy the ids in. (Runtime does NOT need this tool — browser only.)

## Map
| Friendly name | Type | chat id |
|---|---|---|
| _(example)_ Team Standup | group | `19:xxxxxxxx@thread.v2` |
| Notes to self (safe test target) | notes | `48:notes` |

- Group/meeting chat ids look like `19:…@thread.v2`.
- 1:1s open via the **person** search hit `[data-tid$="8:orgid:<userGuid>"]` — no id needed here.
- `48:notes` is the current user's "Notes to self" — the safe target for testing send/react without
  messaging anyone.

## Aliases (optional)
Map informal names, e.g. **"release chat"** → the newest release group. Note ids that rotate (a
per-release chat gets a new id each release) so they get refreshed.
