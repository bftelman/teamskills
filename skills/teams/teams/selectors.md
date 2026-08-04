# Selectors — VERIFIED 2026-07-27 (new Teams web, teams.cloud.microsoft)

All verified live unless marked. Teams animates constantly, so **act via `browser_evaluate`
(`document.querySelector(...).click()`)** rather than `browser_click`/`hover` (which time out on the
"stable" check). Use `browser_type` only for the compose box and search input.

## Confirm which chat is open
`document.title` → the segment before ` | Microsoft Teams` is the chat name.
(There is NO working `chat-header-title` data-tid — don't use it.)

## Compose & send
| Element | Selector | Notes |
|---|---|---|
| Compose box | `[data-tid="ckeditor"]` | contenteditable, role textbox, aria "Type a message" |
| Send | press `Enter` in the editor | or `[data-tid="sendMessageCommands-send"]` |

## Search & navigate (PRIMARY nav)
| Element | Selector | Notes |
|---|---|---|
| Search input | `input[data-tid="AUTOSUGGEST_INPUT"]` | type with real keystrokes (`slowly:true`); `fill()` won't trigger suggestions |
| Result (group/meeting) | `[data-tid$="<chatId>"]` | prefix `AUTOSUGGEST_SUGGESTION_TOPHITS`/`GROUPCHATS` + `19:…@thread.v2` |
| Result (person → 1:1) | `[data-tid$="8:orgid:<userGuid>"]` | prefix `…TOPHITS`; opens/creates the 1:1 |
| Dismiss flyout | blur input + dispatch Escape keydown/keyup, then focus `[data-tid="ckeditor"]` | flyout lingers otherwise |

## Messages
| Element | Selector | Notes |
|---|---|---|
| Message row | `[data-tid="chat-pane-message"]` | match on text; last match = most recent |
| Message pane (scroll) | `[data-tid="message-pane-list-viewport"]` | fallback `[role="main"]`; read `.innerText` |
| More-options menu (per msg) | `[data-tid="message-actions-menu-hidden-button"]` | dispatch `mouseenter/over/move` first, then `.click()` |

## Quote-reply
Open the message's more-options menu → click the `role="menuitem"` matching `/reply with quote|^reply/i`
(label "Reply with quote (Alt+R)"). Compose box then shows the quoted reference.

## Reactions (in the more-options menu's top row)
| Element | Selector |
|---|---|
| Like 👍 | `[data-tid="message-actions-like"]` |
| Heart | `[data-tid="message-actions-heart"]` |
| Laugh | `[data-tid="message-actions-laugh"]` |
| Surprised | `[data-tid="message-actions-surprised"]` |
| Full picker | `[data-tid="add-reaction-picker-entry-point-button"]` (also `expanded-reactions-picker-entry`) |
| Verify applied | `[data-tid="diverse-reaction-summary"]` present / "N … reaction" text on the message |

## @mention
| Element | Selector | Notes |
|---|---|---|
| Trigger | type `@<name>` into `[data-tid="ckeditor"]` with real keystrokes | picker needs keystrokes |
| Picker | `[data-tid="AutocompletePopup-Mentions"]` | |
| Person option | `[data-tid="autocomplete-picker-item-<Full Name>"]` (role option) | `…-Everyone` notifies all |
| Inserted chip (verify) | `[itemtype="http://schema.skype.com/Mention"]` wrapping `<mention mri="8:orgid:<guid>">` | a real tagged mention, not text |

## CDP / connection
- Debug Chrome must listen on IPv4 `127.0.0.1:9222`; MCP endpoint must be `http://127.0.0.1:9222`
  (not `localhost` → IPv6 `::1` refuses).
- `ECONNREFUSED …:9222` mid-run = the debug Chrome window was closed → relaunch `teams-chrome.ps1`.
