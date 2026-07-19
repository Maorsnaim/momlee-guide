# From Sivan → Maor

> Two-way channel. Sivan: add tasks/questions/updates for Maor here (commit +
> push — you have write access), or log them in Notion (see skill
> `momlee-worklog`). Maor reads this on every pull and clears handled items.

## Tasks for Maor

> # 🔴 MAOR — ALL WEB UI WORK IS BLOCKED ON YOU (2026-07-15)
>
> **Sivan is a VIEWER on your Figma account — she cannot make any of these calls.
> Every item below needs YOU.** The web primitive foundation is shipped and CI is
> green (AppText · Input · Button(text|icon) · Icon), but **UI work cannot continue
> until these are answered.** Ordered by how much they block:
>
> 1. **⛔ FIGMA PLAN / SEAT — blocks EVERYTHING UI.** Code Connect needs
>    **Organization/Enterprise**; the account is **Professional** and Sivan's seat is
>    **View**. Worse, the View seat has a **Figma MCP call quota** that also throttles
>    `get_design_context` — today's primitives work **exhausted it**, so no design can
>    be pulled at all right now (a new session does NOT reset it; it's server-side per
>    seat). **You own the account — please upgrade the seat/plan, or tell us to drop
>    Code Connect permanently and work within the quota** (screens would then need the
>    instance-map done manually: grep `figma: <node>` + components.md).
> 2. **⛔ `Buttons/Social button` NODE ID — blocks Button `kind='social'`** (the
>    Google/Facebook/Apple login buttons). `search_design_system` returns only a
>    componentKey (`cfa87e645c6c872eef7866b4f5dc367153914e96`); `get_design_context`
>    needs a **nodeId**. Please open the component in Figma and send the URL — it ends
>    in `?node-id=...`. (Sivan can't hunt for it: no quota.)
> 3. **⛔ `components.md` CONTRACT-ROW FORMAT** (your OS task, High) — the web
>    implementations of AppText/Input/Button/Icon still need their rows. **We did NOT
>    invent a format** — it's your call.
> 4. **⚠️ REUSE-GATE RULING** — your `check-component-dupes` Rules A+D contradict your
>    own ADR-016 (see the task below). A temporary `isAdr016PlatformPair()` keeps CI
>    green; please confirm or replace it.
> 5. **⚠️ `@svgr/webpack` DEPENDENCY** — so web icons single-source from your
>    `packages/ui/assets/icons/*.svg` instead of inlined copies (drift risk).
> 6. **📋 Dev Changelog DB** — still missing, so the worklog stays in this file.
> 7. **📋 PLAN C (schema re-baseline)** — still awaiting your decision; it blocks the
>    product-core service work (meetups/favorites/forum).
>
> **Net effect: Sivan is idle on UI.** Items 1 + 2 unblock the most. If you'd rather
> she work on something else meanwhile, say which — the remaining board items are
> provider/admin-side (not MVP) or blocked on Plan C.

- [ ] 2026-07-15 (**Maor + Sivan — business decision: Figma plan/seat. BLOCKS Code
  Connect, and THROTTLES all figma-first work**): **Code Connect needs an
  Organization/Enterprise Figma plan — the account is Professional with a View seat.**
  Next after the primitives was the OS task *"Set up Figma Code Connect mappings for
  the primitives"* (it's the right order — mapped nodes make `get_design_context`
  hand back the existing component for every screen instance = the natural
  re-implementation prevention, before we build screens). It cannot be done:
  - Figma's own `figma-code-connect` skill states: *"Organization or Enterprise plan
    required — Code Connect is not available on Free or Professional plans."*
  - Verified empirically — `get_code_connect_suggestions` returns: *"You've reached
    the Figma MCP tool call limit for your **View seat on the Professional plan**.
    Upgrade your seat or plan for more tool calls."*
  - `whoami`: `Sivan Bittan's team` (tier **starter**), `Webecy` (tier **pro**) —
    **seat = View on both**.
  **Impact beyond this task:** the same View-seat quota throttles `get_design_context`
  — so it limits **all** figma-first UI work (incl. rebuilding the Mom onboarding
  screens), not just Code Connect. Today's primitives work exhausted the quota.
  **Decision needed:** upgrade the seat/plan (cost), or accept: no Code Connect + a
  hard cap on Figma MCP calls per session. If we skip Code Connect permanently, screen
  work must do the instance-map manually (grep `figma: <node>` + components.md per
  momlee-figma-first step 5) — doable, just less safe.
  Notion task marked **Blocked** with this reason.

- [ ] 2026-07-15 (**Maor — need the Figma node id; blocks Button `kind='social'`**):
  **What is the node id of `Buttons/Social button`?** Building the web Button's
  social kind figma-first, `search_design_system` returns only a `componentKey`
  (`cfa87e645c6c872eef7866b4f5dc367153914e96`) — `get_design_context` needs a
  **nodeId**, and `components.md` records the set by name but no node. Rather than
  mirror the native impl blind, I **deferred the social kind** and am asking. Send
  a node-specific URL (`…?node-id=1-2`) and I'll build it. Same for `Buttons/Social
  button group` if it's in scope. *(For reference the nodes I do have and used:
  Button set `3287:427074`, icon-only `3287:428811`, Input `17297:8153`, Icons
  library `3463:407484`.)*
- [ ] 2026-07-15 (**Maor — dependency approval, low priority; a workaround is in
  place**): **`@svgr/webpack` for web, so icons are single-sourced.** Native
  imports the Figma glyphs from `packages/ui/assets/icons/*.svg`; web cannot import
  `.svg` as a component without svgr, so I **inlined the identical Figma paths**
  into the web `Icon` (explicitly NOT lucide — your rule "never substitute
  look-alike glyphs" is respected). The cost is **drift risk**: if you update an
  icon in Figma, native's `.svg` updates and web's inlined copy silently doesn't.
  Proper fix = add `@svgr/webpack` (devDep, web only) so both platforms consume the
  SAME files → DEPENDENCY GATE, your call. Not urgent — 5 glyphs, marked
  "keep in sync" in the file.

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

- [ ] 2026-07-15 (**FYI + one small UI question at the end**): **Identity Verification
  is moving — Sivan answered your "Open Questions (Owner: Sivan)" and we've broken it
  into tasks.** *Why now:* all web UI work is blocked on your Figma decisions (see the
  red block above), so Sivan moved us onto the **backend of the mom onboarding's
  verification step**, which is decidable without you. Nothing was invented — your
  model already covered it (we checked first and did **not** create duplicate
  Features/Stories).

  **Sivan's decisions** (recorded on the Feature
  [Identity Verification](https://app.notion.com/p/38e450ad0ae681e9894beab124a12942),
  new section 11):
  1. **Vendor = Didit** (supersedes **Persona** everywhere in the spec) — a
     backend/integration call, hers.
  2. **Selfie only, NO ID document.** Rationale: moms on maternity leave holding a
     baby won't go find a wallet mid-flow — they'd abandon onboarding.
  3. **Retry = max 2 attempts** (answers your "כמה ניסיונות?").

  **New implementation tasks — IDV-1 … IDV-10** in the Tasks DB, ordered, all linked
  to your story *"Mom completes identity verification after the children step"*:
  [IDV-1 Didit spike](https://app.notion.com/p/39e450ad0ae6811fa57be0563816da56) ·
  [IDV-2 migration](https://app.notion.com/p/39e450ad0ae681e59f70d720aa823f24) ·
  [IDV-3 session+webhook](https://app.notion.com/p/39e450ad0ae6813fb6e4ea672f86a107) ·
  [IDV-4 policy engine](https://app.notion.com/p/39e450ad0ae681d288dbdc6a0b339f89) ·
  [IDV-5 admin queue](https://app.notion.com/p/39e450ad0ae6812f8430fa15d2be1688) ·
  [IDV-6 approval email](https://app.notion.com/p/39e450ad0ae68102b0a9eac3081791ea) ·
  [IDV-7 eligibility/permissions](https://app.notion.com/p/39e450ad0ae6816481ebcae6f7054b64) ·
  [IDV-8 analytics](https://app.notion.com/p/39e450ad0ae6818c9264d50022caafe4) ·
  [IDV-9 privacy/biometric](https://app.notion.com/p/39e450ad0ae681df9b2bfe33918fdef2) ·
  [IDV-10 screens (Blocked)](https://app.notion.com/p/39e450ad0ae681c5997ecd5f5cfac1f7)

  **IDV-1..IDV-9 need no Figma** — that's the work that can proceed. We kept your
  entity's core principle intact: *the vendor result is evidence only; the decision is
  Momlee policy.*

  **✅ The admin-queue UI question is WITHDRAWN — Sivan already ruled it (2026-07-15):**
  the admin dashboard **does not need your design**. It's fully blocked from non-admin
  users, so it isn't part of the designed product surface — we'll reuse the existing
  admin patterns and skip design for the MVP. She says she'd already told you this.
  **Nothing for you on IDV-5.**

- [x] 2026-07-15: **✅ Dev Changelog — verified and synced, thank you.** Your archived-DB
  diagnosis reconciles everything (my "does not exist" was a correct observation with a
  wrong conclusion — archived ≠ deleted, and Notion search doesn't index archived pages;
  my Trash check couldn't see it either). Fetch works on the same id. **Synced 7 pending
  entries as rows** (Noto self-host; primitives Phase 1; primitives Phase 2; the reuse-gate
  fix; Sivan's Didit decisions; the Didit spike result; the IDV-1..10 breakdown).
  **Logging directly to Notion from now on**; this file stays for outages + for things
  that need you.

- [x] 2026-07-15: **✅ Thank you for the ADR-016 ruling + the Windows path fix.** Nothing
  to change on our side; `isAdr016PlatformPair` stays as shipped and the gates now match
  CI locally. Also noted: `Buttons/Social button` node still pending (Claude hit the same
  plan limit server-side — `kind='social'` stays deferred, agreed), and `@svgr` is parked
  behind your bigger icon-library question. **The Figma plan/seat decision is still the
  single biggest blocker on all UI work** (red block at the top).

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

- 2026-07-15 (`apps/web`, `momlee-web` merge `86246d1`, CI GREEN — Sivan + Claude):
  **Phase 2 — Icon primitive + Button `kind='icon'` SHIPPED.** `Icon` (figma
  `3463:407484`): semantic `forward`/`backward` mirrored per layout direction,
  sizes, `currentColor` tint — built on the **same Figma Icons library glyph paths
  as native**; **no lucide** (your "never substitute look-alike glyphs" rule caught
  a planned lucide wrapper before it shipped — thank you, that rule earned its
  keep). `Button` now a discriminated union with `kind:'icon'` (figma
  `3287:428811`): forward = 48 brand-solid CTA, back = outlined, and the **disabled
  back arrow uses the `fg-quaternary` token** (explicit token, not an opacity wash,
  per your 12-15 note). Verified: tsc + next build + gates + visual pass on a
  throwaway preview (deleted). The `isAdr016PlatformPair` gate fix generalised —
  Icon's native/web pair passed with no further change.
  **Open for you:** the `Buttons/Social button` **node id** (blocks `kind:'social'`)
  and the optional `@svgr` dependency — both above. Web primitives now: **AppText ·
  Input · Button(text|icon) · Icon**.

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

## 2026-07-16 (Sivan → Maor) — two small asks

1. **Please bump the plugin version when you push content.** Plan C + the new
   `knowledge/target-data-model.md` landed on the marketplace at commit
   `ba0e443`, but `plugin.json` still reads `0.16.1` — so
   `/plugin install momlee-guide@momlee` sees "same version, already
   installed" and never refreshes the local cache. The plugin skills read
   `../../planning` and `../../knowledge` FROM that cache, so a Claude session
   running the skills straight gets the STALE (pre-Plan-C) files. I've manually
   synced my cache as a stopgap, but a version bump (e.g. `0.16.2`) on the next
   content push is the real fix so nobody reads stale directives.

2. **June "pending decisions" row (Dev Changelog) — one item is genuinely
   yours, the other two are mine.** Reclassifying by our ownership split:
   feature-flags kill-switch (infra) and the account-deletion cascade map
   (data policy) are Sivan's calls — I'm taking those (the cascade map will
   also fall out of Plan C's per-area retention work + IDV-9). The only piece
   that needs you: **the shared Empty/Error state designs in Figma**, which
   block building the four-states resilience component. No rush while Plan C is
   backend-only — just flagging it's the remaining UX dependency on that row.

## 2026-07-19 (Sivan → Maor) — MUST-READ: updated MVP strategy + a Feature/Story to model + a plugin update

**1. The MVP is re-scoped — please align your OS page designs to this.**
MVP ships with **admin + mom users ONLY. There is NO provider user in the MVP**
(no provider profile, no provider dashboard, no in-app subscriptions/billing, no
provider-created content).
- **Mom meetups** — created by moms (as today).
- **PRO meetups** — **IN the MVP, but published by the ADMIN** on a provider's
  behalf. When the admin creates a PRO meetup they enter, from info the provider
  supplied out-of-band: provider **photo**, a short **"who am I"**, **what the
  meetup covers**, a **Paybox** payment link, and the **attendee fee**.
- **Phased provider rollout:** grow mom traffic -> providers ask to self-publish
  -> introduce a **99 ILS/mo** provider fee (billed on a **separate platform**,
  not in-app) -> once enough pay, build the **full provider epic** (profile,
  dashboard, subscription tiers, self-serve pro meetups).
- **Organizations** = out of MVP (Phase-2). **Waitlist** = Post-MVP.
  **`qa` meetup type dropped** — Q&A/AMA is just a PRO meetup. Only **`mom | pro`**
  types. **Pro price band = 20-180 ILS.**

**2. Please formally model a new MVP Feature + Story in the OS (your lane).**
Feature: **Admin-published PRO meetup** (under the **Meetups** epic, Scope = MVP).
Story: *"Admin publishes a PRO meetup for a provider."* Actor = Admin. Flow: admin
fills provider photo / who-am-I / what-it-covers / Paybox link / fee +
location/date/time/capacity -> publishes -> it appears in discovery like any
meetup, tagged PRO with the host block + price + a register-and-pay affordance.
Wire its registries (Permissions: admin_create_pro_meetup; Events; Product Rules
for pricing). The 4 build Tasks already exist in the Tasks DB (migration + admin
publish UI + mom registration + admin approve/mark-paid), ready to link.

**3. Mom registration + interest model (so the designs match):**
- **Heart icon = "interested"** — adds the meetup to her "interested" list in her
  dashboard; NOT a registration request. The meetup shows **"X are interested in
  this meetup"** = count of moms who hearted it.
- **"Ask to register"** = a request the admin sees. She pays the provider via the
  **public Paybox group**; the **admin** sees the payment and manually confirms
  her (going + paid). No provider-confirm step.

**4. Meetups schema (the migration I am landing now):**
- `baby_meetups` += `meetup_type(mom|pro)`, `status(open|full|completed|cancelled)`,
  `capacity(2-100)`, `price(pro:20-180)`, `payment_link(pro:Paybox)`,
  `host_name/host_bio/host_photo_url(pro)`.
- `meetup_attendees`: `status(interested|asked_to_go|going|cancelled)` +
  `payment_status(pro: pending|paid|cancelled)`.

**5. Please update the plugin `knowledge/target-data-model.md` to this MVP plan**
(it still says `qa` type + 25-120 + provider-self-serve pro meetups). Drop `qa`
(only `mom|pro`), price 20-180, note PRO meetups are admin-published in MVP (no
provider user), organizations Phase-2, waitlist Post-MVP. Plus the **plugin
version bump** so the cache refreshes (still open from 07-16).

**Already reflected in Notion by me:** Meetups epic annotated (admin-published PRO
into MVP, Waitlist->Post-MVP); Provider Profile & Services / Subscriptions /
Business Insights / Organization & Team epics + the Professional user type marked
Post-MVP; duplicate provider page archived; 4 build Tasks created;
provider-dashboard-queries task deferred; a Dev Changelog row logged.

## 2026-07-19 (Sivan → Maor) — meetups-core migration is LIVE; mom-facing meetup UI is now the Figma critical path

- The Plan C **meetups-core migration is APPLIED TO LIVE** (db-deploy green,
  momlee-web 75bbc22). `baby_meetups` gained `meetup_type(mom|pro)` + the
  admin-published-PRO fields (host photo/bio, Paybox link, fee, capacity, status);
  `meetup_attendees` gained the registration lifecycle
  (`asked_to_go|going|cancelled`) + `payment_status`. OS Database Tables registry
  rows Meetups + Meetup Registrations flipped to **Aligned**.
- **What can proceed now with NO Figma:** the backend/data layer + the **admin**
  tooling (publish a PRO meetup, approve/mark-paid) — the admin surface needs no
  design per Sivan. **What now waits on YOUR Figma:** the **mom-facing** meetup
  screens (browse, meetup detail with the host block, the heart/favorite, the
  "ask to register" flow). That is the critical path on the meetups feature.
- Clarification to my 07-19 brief: the **heart = the EXISTING `favorites` table**
  (a like); a meetup's "X are interested" = `COUNT(favorites)`. `meetup_attendees`
  holds only the registration lifecycle. No change to your designs — just the data
  source of the interest count.
- Still open for you (from the earlier brief): formally model the MVP Feature/Story
  **"Admin-published PRO meetup"**, and the mom-facing meetup screens in Figma.
