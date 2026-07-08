# From Sivan → Maor

> Two-way channel. Sivan: add tasks/questions/updates for Maor here (commit +
> push — you have write access), or log them in Notion (see skill
> `momlee-worklog`). Maor reads this on every pull and clears handled items.

## Tasks for Maor

_(empty — add items as `- [ ] ...` with a date)_

## Updates / questions

_(empty)_

## Worklog (pending Notion sync)

_(entries logged here only when the Notion MCP wasn't available; synced to the
Dev Changelog later)_

- **2026-07-08 · Claude · Infra · Platform · Done** — Created the `momlee-web`
  branch (cut from the tip of `momlee-native`) and extended the mechanical CI
  gates to `apps/web`: check-naming scans web, check-deps enforces the web
  manifest (60 existing web deps allowlisted), check-figma-refs requires a
  Figma node ref on every NEW web page (32 pre-pivot pages grandfathered), NEW
  check-rtl-web logical-properties ratchet (baseline 543, may only decrease),
  ci.yml triggers include momlee-web. All gates green locally. App-repo commit
  `032cb39` (pushed). Plugin: open-tasks announcement `e0c54cb`.
  Link: https://github.com/sivanhasson/MomLee/commit/032cb39
  _(Notion MCP token expired mid-session — sync this row to the Dev Changelog.)_
