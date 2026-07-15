# From Sivan → Maor

> Two-way channel. Sivan: add tasks/questions/updates for Maor here (commit +
> push — you have write access), or log them in Notion (see skill
> `momlee-worklog`). Maor reads this on every pull and clears handled items.

## Tasks for Maor

- [ ] 2026-07-15 (**Maor — please rule; a TEMPORARY fix is in place, CI is green**):
  **Your reuse gate contradicts your own ADR-016 for every primitive.**
  `check-component-dupes.mjs` Rule **A** (no two files share a base name) and Rule
  **D** (one Figma node = one component file) both assume ONE component per
  name/node. But ADR-016 mandates each primitive be implemented **once per
  platform under the SAME name and the SAME node** — native (`packages/ui/src`) +
  its web twin (`apps/web/src/components/ui/primitives`) — because that pair IS
  the shared contract. So **every** primitive fails CI; `button`/`input` only
  passed **by accident** (they sit in `GRANDFATHERED_DUP_NAMES` for the unrelated
  demo-shadcn dups). `AppText` was the first to hit Rule A; Button + Input then hit
  Rule D on merge (real red CI on `momlee-web`).
  **What I did (temporary, easily reverted — you own the rules):** added
  `isAdr016PlatformPair()` to the gate, recognising EXACTLY that one native + one
  web primitive pair and nothing else. **Verified with negative tests:** a 3rd file
  claiming a node still FAILS; a web-vs-web duplicate name still FAILS. Commit
  `d387289` on `momlee-web`, CI green. **Revert = delete the helper + its two call
  sites.** Please confirm this is the rule you want, or tell me your preferred
  shape and I'll switch to it.
  *(Minor papercut, not blocking: the gates' grandfather lists use forward slashes,
  so on Windows `path.relative` yields backslashes and they misfire LOCALLY —
  pre-existing files look "new". CI (Linux) is fine. Worth normalising paths.)*

> **❌ RETRACTED 2026-07-14 — the previous "✅ ANSWERED" note here was WRONG.**
> An earlier session claimed the Dev Changelog "is NOT deleted, it's alive and
> written to daily", that rows were synced into it, and that Sivan's 404s are an
> ACCESS gap ("your Notion account was never shared into Maor's workspace").
> **Both claims are false — verified four ways this session:**
> 1. `fetch self` → authenticated as **Sivan Bittan (sivan@applee.dev) INSIDE the
>    "Maor Naim's Notion" workspace**; Momlee OS pages read fine and the OS Tasks
>    DB queried successfully. **There is no access gap.**
> 2. The hardcoded id `ee6d4bbb-1444-479c-b818-36f7e3951988` **404s**.
> 3. An API search across the workspace finds **no Dev Changelog database**
>    (Momlee OS = 01–08 + Page Template; 07-Engineering = Principles / Stack /
>    ADRs / Validators only).
> 4. Sivan's Notion UI search finds only **text mentions**, and his **unfiltered
>    Trash is empty** — so it wasn't deleted inside the 30-day recovery window.
>
> **Maor — the real ask (see the task below): does a Dev Changelog DB exist
> anywhere?** If yes → send the URL/id. If no → create it (or tell us which DB
> should receive worklog rows) **and update the id hardcoded in the plugin skills
> + worklog hook** — until then the enforced hook points at a dead id for every
> session/contributor. **Sivan's decision (2026-07-14): he keeps logging to THIS
> file (git) and will migrate into the real Changelog once it exists — deliberately
> NOT creating a second DB, to avoid duplication.**
>
> **✅ FIXED (Claude, 2026-07-14): the hook/skill cache-path bug.** The worklog
> hook message + skill fallback now explicitly require writing to a REAL CLONE
> of momlee-guide (never the plugin cache) + commit+push+verify.

- [ ] 2026-07-14 (**Maor — please decide + re-sync tokens; BLOCKS the web Button**):
  **`bg-brand-solid` token drift — Figma vs the package/snapshot disagree on the
  primary CTA.** Building the first web primitives figma-first, the live Figma
  node `3287:428579` (Buttons/Button · xl/Primary/Default) renders
  `bg-brand-solid = #f0e4e6` (light pink) with label `text-primary-(900) #171717`
  (dark). But your own snapshot `design-system/tokens.md` AND the `@momlee/tokens`
  package AND native Button all say `bg-brand-solid = #b05f64` (mauve) with WHITE
  primary text. So the whole primary treatment differs: **(mauve bg + white text)
  in code vs (light-pink bg + dark text) in live Figma**, and the file is marked
  "not final". Two questions:
  1. **Is the Figma retint final** (`bg-brand-solid` → `#f0e4e6`)? If yes it's a
     **token re-sync** of `@momlee/tokens` (via momlee-sync-tokens) that affects
     **web AND native**, and likely the broader "palette retint".
  2. **Primary button text — white or dark `#171717`?** (pairs with whichever bg
     is canonical.)
  I've **held the web Button** until you rule (per figma-first "never ship a
  guess"). AppText + Input were unaffected — their tokens matched Figma exactly
  (`#a3a3a3`/`#404040`/`#b54141`/`#d4d4d4`) — and are built + verified. Details in
  the worklog below.

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

- [x] 2026-07-14: **Web token-preset wiring is now DONE** (supersedes the
  2026-07-12 "on HOLD" note). `apps/web` declares `@momlee/tokens` + registers
  `presets:[momleePreset]`; token classes (`bg-bg-primary`, `text-text-primary`,
  `gap-xl`, `font-sans`=Noto, …) resolve and emit. **Additive / zero visual
  regression** — the demo shadcn theme is untouched (Heebo font swap intentionally
  left for its own task). CI assertion added to `check-tokens-web.mjs` so the
  wiring can't regress. See worklog below.
- [ ] 2026-07-14 (**Maor — please decide**): **Radius A/B — should the Figma token
  radii win on `rounded-sm/md/lg`?** The preset and the demo's shadcn theme both
  define `sm/md/lg`; on a Tailwind preset merge the **main config wins**, so the
  demo's CSS-var radii keep those keys and the preset's Figma values
  (`radius-md`=8px, `sm`=6px, `lg`=10px) are **shadowed** (verified: `rounded-md`
  still resolves to `calc(var(--radius) - 2px)`). Consequence: a NEW Figma
  component writing `rounded-md` expecting 8px silently gets the demo's ~10px.
  - **Option A (shipped, non-breaking):** demo radii keep winning during the
    demo-coexistence window; new primitives must NOT rely on `rounded-md/sm/lg`
    meaning the Figma value — use non-colliding keys until the demo is retired.
  - **Option B:** pull the demo's `sm/md/lg` radii out of `theme.extend` so the
    token radii win — a **3-line follow-up**, but it changes the corner radius of
    every Radix/shadcn component across all 32 demo pages at once. Your design call.
- [ ] 2026-07-14 (**Maor — please handle**): **[GAP] Validator "flip to Active".**
  The wiring that the Validator tracks is now live, so it can be flipped. The
  status entry lives in the momlee-guide plugin's `knowledge/`/`planning/` channel
  (not the app-repo checkout), so it can't be edited from a repo session — please
  flip it, or tell Sivan where it lives.
- [ ] 2026-07-14 (**Maor — design sign-off**): **Palette retint (separate, future).**
  Retinting the demo shadcn CSS variables onto the MomLee brand tokens (a real
  visible design change per ADR-016) is deferred to you — not part of the wiring
  above, needs your sign-off before anyone ships it.

## Worklog (pending Notion sync)

- 2026-07-15 (`apps/web`, `momlee-web` merge `8abf0f6` + gate fix `d387289`, CI
  GREEN — Sivan + Claude): **Phase 1 web primitives SHIPPED — AppText · Input ·
  Button** (OS task `platform.web_design_foundation`). Built figma-first on the
  shared `@momlee/tokens` preset in `apps/web/src/components/ui/primitives/`,
  mirroring the native `@momlee/ui` contract (same name/props/node/tokens, ADR-016);
  RN → DOM+Tailwind. **Thank you for the fast rulings** — the reverted retint
  (`bg-brand-solid` back to `#b05f64` + white label) and radius **B** (`51ed790`)
  both verified flowing through: built CSS shows `bg-bg-brand-solid` =
  `rgb(176 95 100)` and `rounded-md` = `8px`. Verified tsc + next build + token CSS
  + a visual pass on a throwaway preview route (since deleted). Sanctioned
  web/native divergences: AppText weight → CSS font-weight (browsers pick Noto
  weights natively; the per-family rule is an iOS quirk) and Input uses the true
  Figma line-height (the iOS display-xs→md hack doesn't apply). **Next:** Phase 2 =
  Icon + Button `kind:'icon'|'social'`. **Needs you:** the gate ruling above, and
  the components.md contract-row format (I did NOT invent one — the rows still need
  their web implementation noted, pending your format task).

_(entries logged here only when the Notion MCP wasn't available; synced to the
Dev Changelog later)_

> _(2026-07-14: the 07-13/07-14 entry clusters below were SYNCED to the Dev
> Changelog by Claude — tokens-preset wiring `39d450ad...8d14` (In progress,
> awaiting the 6d7d4e1 push) + the consolidated security closeout
> `39d450ad...2735` (Done). The Noto-font entry below lands as its own row.
> Reminder: commits `6d7d4e1` + `6b3bf0d` are still LOCAL on your machine —
> push them so CI runs and the Validator can flip Active.)_

- 2026-07-14 (`apps/web`, branch `sivan/web-primitives`, commit `17cda9b` — Sivan
  + Claude): **First web primitives, figma-first — AppText + Input built + verified;
  Button held.** Per ADR-016 (shared contract, per-platform impl), mirrored native
  `@momlee/ui` as DOM+Tailwind on the token preset, in
  `apps/web/src/components/ui/primitives/`. AppText (RTL text primitive; weight via
  CSS font-weight — Noto weights are native on web) + Input (figma `17297:8153`:
  underline, 5 states, Extras slots; tokens confirmed against Figma). tsc clean,
  next build ok, token classes emit. NOT merged to `momlee-web` (task incomplete —
  Button blocked on the `bg-brand-solid` drift above; throwaway `/primitives-preview`
  harness stays on the branch until merge). Next: Maor's Button ruling → build
  Button → delete preview → merge via handle-next-issue.

- 2026-07-14 (`apps/web`, `momlee-web`, commit `6b3bf0d` — Sivan + Claude):
  **Web is now on the official font — Noto Sans Hebrew, self-hosted** (OS task
  `platform.web_design_foundation`; Heebo removed per ADR-016). Swapped the
  runtime Google Fonts Heebo `@import` for `next/font/google` Noto Sans Hebrew,
  self-hosted at build → **zero runtime external font request**, no layout shift
  (display:swap + size-adjusted fallback). `--font-sans` variable; web
  `tailwind.config` maps `font-sans`→`var(--font-sans)`; `heebo` key removed. The
  shared `@momlee/tokens` literal family is untouched, so **mobile is unaffected**.
  Verified: `tsc` clean, `next build` ok, 4 woff2 self-hosted, no Google Fonts
  refs remain. FYI (visible typography change) — no decision needed from you.
- 2026-07-14 (`apps/web`, `momlee-web` — Sivan + Claude): **Wired the shared
  `@momlee/tokens` preset into the web Tailwind config** (OS task
  `platform.web_design_foundation`, Critical). Declared `@momlee/tokens` as a web
  workspace dep, added `import momleePreset from "@momlee/tokens/preset"` +
  `presets:[momleePreset]` to `apps/web/tailwind.config.ts`, and added a
  non-regressable CI assertion in `check-tokens-web.mjs` (fails if the preset
  import/registration is dropped). **Additive, zero visual regression** — the demo
  shadcn theme stays; new token-namespaced classes (`bg-bg-primary`,
  `text-text-primary`, `gap-xl`, `font-sans`) now emit. Verified end-to-end: pnpm
  symlink present, gate green, a Tailwind compile emits the token classes and
  confirms `rounded-md` is still demo-shadowed (the radius A/B decision above).
  Heebo→Noto font swap intentionally left for its own task. Committed as `6d7d4e1`
  and **pushed** to `momlee-web` (`08cfa3c..dc3ee32`, alongside a `/checkpoint`
  skill tweak `dc3ee32`); `db-deploy` is a no-op (no migration). Follow-up
  `fe25276` fixed a **tsc-only** type error the first push missed (the
  framework-agnostic `as const` preset needed `as unknown as Partial<Config>`;
  runtime was always fine) — CI is now **GREEN** end-to-end (checks ✓ rls-tests ✓
  db-deploy ✓, run 29314383827). Lesson: run `pnpm type-check`, not just a
  Tailwind compile, before pushing a `tailwind.config.ts` change.
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
