---
name: momlee-standup
description: Use at the START of every MomLee work session, after a plugin refresh, or whenever Sivan/Maor asks "what's new" / "מה חדש" / "תעדכן אותי" / "standup" / "מה מחכה לי". Pulls the open work from BOTH channels - the plugin's git channel (planning/open-tasks.md) AND live Notion (Dev Changelog open rows + the Momlee OS Tasks DB) - and returns one short, prioritized Hebrew summary: action items, what changed since the last session, and open/blocked tasks.
---

# Momlee Standup — one summary of everything waiting for you

The goal: opening a session never starts blind. One ask ("מה חדש?") returns
everything relevant from every channel, prioritized.

## The protocol (in order)

1. **Refresh the plugin first** (once per work session), then continue:

   ```
   /plugin marketplace update momlee
   /plugin install momlee-guide@momlee
   ```

2. **Git channel — `../../planning/open-tasks.md`** (Maor → Sivan): read it
   and pick out (a) any ⛔ / ACTION REQUIRED items, (b) sections dated since
   the last session. This file is the authoritative "what Maor set up /
   changed" feed.

3. **Notion — open action items** (requires the Notion MCP connected; if it
   is not, SAY SO and continue with git channels only):
   - **Dev Changelog** (`collection://ee6d4bbb-1444-479c-b818-36f7e3951988`):
     query rows with `Status` in `Planned` / `In progress` — these are open
     action items (rows titled "MAOR: ..." are his; the rest may be yours).
     Also pull the latest ~5 `Done` rows so the user sees what happened since
     they last looked.
   - **Momlee OS Tasks DB** (`collection://e8d450ad-0ae6-8225-a4f8-87288039ecd2`):
     query rows with `Status` in `Not Started` / `In Progress` / `Blocked`,
     with their `Story` relation and `Priority` — these are the open build
     tasks. Highlight `Blocked` ones with their reason (in `Description`).
   - Use `notion-query-data-sources` (SQL). Watch for 429 rate limits — space
     the two queries; on persistent 429 fall back to `notion-search` or
     report "Notion rate-limited, git channels only this time".

4. **Summarize in Hebrew, prioritized, SHORT** (this is a standup, not a
   report):
   - **פעולות שחוסמות / ACTION REQUIRED** — e.g. a missing repo secret.
   - **מה השתנה מאז הפעם הקודמת** — new gates, new branch, new decisions.
   - **משימות פתוחות** — from the Tasks DB (with priority; Blocked flagged).
   - **הודעות מהצד השני** — open "MAOR:"/"SIVAN:" rows.
   Link each item to its source (open-tasks section / Notion page URL).

5. **Do not mark anything Done here.** Standup is read-only; completing an
   item goes through the normal flow (work + momlee-worklog logging).

## Why both channels

`open-tasks.md` moves via git (survives Notion outages, versioned, always
readable); Notion holds the LIVE state (task statuses, changelog). One
without the other gives a stale or partial picture - the standup merges them.
