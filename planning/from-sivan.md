# From Sivan → Maor

> Two-way channel. Sivan: add tasks/questions/updates for Maor here (commit +
> push — you have write access), or log them in Notion (see skill
> `momlee-worklog`). Maor reads this on every pull and clears handled items.

## Tasks for Maor

- [ ] 2026-07-12 (**Maor — please handle**): **Dev Changelog DB appears GONE.**
  The standup skill's collection id (`ee6d4bbb-1444-479c-b818-36f7e3951988`)
  and the 2026-07-08 changelog row (`397450ad0ae681c7bceee1cfae7414ac`) both
  404, and no search hit surfaces the DB (only a "Decision Log"). The worklog
  can't write to Notion — using this git fallback. Please restore it (Notion
  Trash keeps deleted DBs ~30 days) or point the worklog + standup skills to
  the current Dev Changelog data-source id (the id is hard-coded in the
  momlee-guide plugin, so a brand-new DB needs a plugin update to be picked up
  automatically).
  ⚠️ **Take into account: the LIVE DB was changed this session.** Migration
  `20260712000000` is applied to production — it dropped the 17 provider-group
  (M03.6) `audit_action` enum values and deleted 16 pre-launch audit_log rows
  that used them. When you restore/reconcile the changelog (and any OS
  Schema-Registry / Decision-Log entries that reference provider-group audit
  actions), note those values no longer exist in the DB. Details: commits
  `f6cced7` + `08cfa3c` on `momlee-web`.
- [x] 2026-07-12: **LIVE `audit_log` had 16 provider-group (M03.6) rows** —
  RESOLVED. Inspected: 16 rows across 5 actions, all 2026-06-01..09 (the M03.6
  build window; DB is pre-launch/seed-only). Sivan confirmed disposable, so
  migration `20260712000000` was amended (commit `08cfa3c`) to `DELETE` those 16
  rows first (destructive-approved), then trim the enum. FYI only — no action
  for Maor.
- [ ] 2026-07-14 (**Maor — please fix**): **The worklog HOOK's git fallback
  writes to the plugin CACHE, so entries never reach you.** The momlee-worklog
  hook/skill fallback appends to `<plugin-cache>/planning/from-sivan.md`, but the
  cache dir is NOT a git repo — so weeks of worklog/task entries piled up locally
  and never reached this repo (this is why the entries above only landed now, via
  a manual flush — commit `b9f77e5`). Sivan's `/checkpoint` skill now writes to a
  real clone (`C:\Users\sivan\workspace\momlee-guide`) + commits/pushes, but the
  ENFORCED hook still points at the cache for every session/contributor. Please
  repoint it (write to a cloned repo + commit/push, or another synced location).

## Updates / questions

- [ ] 2026-07-12 (**Maor — please decide**): **Web token-preset wiring is on HOLD
  per Sivan.** The task "Wire `@momlee/tokens` preset into `apps/web` Tailwind"
  was scoped and planned (dep declare → `presets:[momleePreset]` → Heebo→Noto
  swap → CI assertion, all zero-visual-regression; plus an optional Step 4 that
  retints the shadcn CSS variables to the token palette = a real visible design
  change). Sivan chose **Hold — don't edit yet**. Two items are yours:
  1. **[GAP] Validator "flip to Active":** the status entry almost certainly
     lives in the momlee-guide plugin's `knowledge/`/`planning/` channel (not in
     the app-repo checkout), so it can't be edited from a repo session. Please
     flip it when the wiring lands, or tell Sivan where it lives.
  2. **Step 4 (palette retint):** deferred to you — it changes how existing
     screens look (moves the demo shadcn palette onto the MomLee brand tokens
     per ADR-016). Needs your design sign-off before anyone ships it.

## Worklog (pending Notion sync)

_(entries logged here only when the Notion MCP wasn't available; synced to the
Dev Changelog later)_

- 2026-07-14 (security cleanup CLOSED OUT — Sivan + Claude): **All live
  security-audit cleanup is DONE.** Edge Functions deleted (curl → 404) after
  archiving (commit `8ffe06f`); Supabase test phone numbers configured with a
  "Valid Until" expiry and verified end-to-end via the Auth API (`/otp` → 200
  no-SMS, `/verify` with the fixed code → session). Added `LAUNCH_CHECKLIST.md`
  (app-repo root) with "clear test OTPs before launch". service_role + Resend
  keys marked Sensitive in Vercel (verified never leaked — no rotation). Mapbox +
  Twilio rotated (see prior entries). Remaining are Maor-only: delete the June
  temp Supabase access token + fix the worklog hook (see Tasks for Maor).
- 2026-07-13 (commit `8ffe06f`, `momlee-web` — not yet pushed): **Archived the 7
  legacy Edge Functions** to `supabase/_archived_functions/` (mirrors
  `_archived_migrations/`), downloaded from the Supabase dashboard before
  deleting the deployed copies. Verified the source is free of hardcoded secrets
  (all read via `Deno.env.get`); added a README with provenance + "do not
  redeploy". This backs up the audit's "delete legacy Edge Functions" item so the
  deletion is reversible. (Notion Dev Changelog still 404 — git fallback used.)
- 2026-07-13 (security ops, no repo commit — Sivan + Claude): **Closed most of
  the open live-security cleanup** from open-tasks.md "Still open after the live
  deploy". DONE: (1) **Mapbox rotated** — new URL-restricted browser token +
  unrestricted server token; old leaked "Default public token" refreshed
  (value invalidated). Also FIXED a latent prod bug — `NEXT_PUBLIC_MAPBOX_TOKEN`
  was MISSING from Vercel, so browser maps had no token in production; now added
  + verified rendering. (2) **Twilio Auth Token rotated** (secondary→primary via
  the zero-downtime flow; the token pasted in chat is now dead). Account SID +
  Verify Service SID unchanged; Supabase Phone (Twilio Verify) config updated.
  (3) **service_role + Resend keys** — full git-history search PROVED they were
  never committed; the Vercel "Needs Attention" flag was only an
  un-marked-Sensitive nudge, not a leak. Marked Sensitive in Vercel; no rotation.
  IN PROGRESS this session: deleting the 6-7 legacy Edge Functions + configuring
  Supabase test phone numbers. STILL ON MAOR: delete the temporary Supabase
  personal access token from the June manual db push (only you can — it lives
  under your account).
- 2026-07-12 (commit `f6cced7`, `momlee-web`): **Unblocked the DB deploy
  pipeline.** ci.yml `db-deploy` now has a `supabase link --project-ref` step +
  `SUPABASE_DB_PASSWORD` (both CI secrets set; access token validated vs the
  Mgmt API). Fixed the red `checks` type error by dropping the 17 frozen
  provider-group values from the `audit_action` enum (migration
  `20260712000000` — enum recreated without them, dependent RLS policy
  preserved, `Record<AuditAction,string>` guard + generated types kept
  exhaustive). Extended `check-destructive.mjs` to also flag DROP
  TYPE/POLICY/FUNCTION. CI result: `checks` ✓, `rls-tests` ✓ (migration valid
  on a fresh DB), `db-deploy` ✗ — the migration's safety guard correctly
  aborted because LIVE `audit_log` has provider-group rows (see Tasks above).
- 2026-07-12 (commit `08cfa3c`, `momlee-web`): amended migration
  `20260712000000` to `DELETE` the 16 pre-launch provider-group test rows from
  `audit_log` before trimming the enum (rows confirmed disposable — pre-launch
  seed data). `checks` + `rls-tests` were already green; this unblocks
  `db-deploy` (CI re-run in progress at handoff).

_(2026-07-08: the momlee-web row was synced to the Dev Changelog after Notion
reconnected - 397450ad0ae681c7bceee1cfae7414ac.)_
