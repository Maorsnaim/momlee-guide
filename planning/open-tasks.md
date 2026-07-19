# Open tasks — read me on every pull

> Maor-maintained. Flows to Sivan via git. This is the live "what's pending /
> what changed" channel between us. Check it whenever you update the plugin.

## 📌 DECISION OF RECORD (Maor, 2026-07-19) — the app font is now Google Sans (reverses Noto Sans Hebrew). No action for you yet.

Google released **Google Sans on Google Fonts under SIL OFL 1.1** (2026 —
after older docs/skills were written; verified: official download's OFL.txt +
the live specimen page). The released variable font (GRAD/opsz/wght)
**includes full Hebrew** (all alef-tav, verified in the TTF cmap) — there is
no separate "Google Sans Hebrew" family. The Momlee 2.0 Design System file
is built in it, and it is now the official app font.

- **No code change now.** Web stays on self-hosted Noto until the DS-closure
  token sync — the font flips then (next/font local files for web, static
  TTFs for native), through the `fontFamily.sans` role token as always.
  Plugin docs/skills that still say "Noto Sans Hebrew is official" are
  superseded by this note and will be updated in that same sync.
- **Figma caveat:** Figma's cloud font service has not synced the family yet,
  so it renders only where the font files are installed locally. When you
  return to UI work, install the OFL download (fonts.google.com → Google
  Sans, install the `static/` TTFs incl. the 17pt set) or the file will show
  missing-font warnings.

## 🚀 TOP PRIORITY (Maor, 2026-07-16) — Plan C APPROVED: re-baseline the product-core schema. This is the next block of work.

Maor has formally approved **Plan C**. Everything Plan-C-related is now the
priority track — it outranks the remaining component/icon items (those stay
parked below and resume later).

**WHY (the audit conclusion, in short).** The live DB is **pre-launch** (seed
data only) and **provider-heavy**: the provider / verification / admin /
audit / RLS subsystem is mature and is **KEPT as-is**. The product core —
meetups, organizations, subscriptions/entitlements, messaging, role/status
enums — is barely modeled or missing entirely. Patching it incrementally
would drag the old shapes forever; a full rewrite would throw away a good
backend. Plan C is the middle path: **keep the backend spine, cleanly
re-baseline the product core on top of it, with the Momlee OS as the spec —
now, while migration cost is near zero.** It also unblocks the tasks marked
"blocked on Plan C" (meetups queries/data layer) and gives every future
screen real tables to stand on.

**WHAT it requires (scope).**
1. **Preflight:** re-verify baseline == live (last verified 2026-07-02),
   regenerate `packages/supabase/src/database.types.ts` from live, confirm
   migration bookkeeping is clean before the first new migration.
2. **Reconcile the Mismatch rows** in the OS **Database Tables registry**
   (the 41-row honest mirror — it is the living map of this work): e.g.
   `baby_meetups` gains `meetup_type` / `capacity` / `price` / `status`;
   `meetup_attendees` gains `attendance_status`; enum hardening
   (`user_roles.role` TEXT → `app_role` enum — flagged top risk).
3. **Create the Planned tables** per the target model: organizations +
   `organization_members` (D1: IN MVP, M:N), subscriptions/entitlements
   (**D6 is still open — MODEL ONLY, implement no entitlement values**),
   messaging/conversations, `onboarding_progress`, notifications family,
   reports/safety. Deprecations (forum, provider CRM, dead mocks) per the
   same doc.
4. **After every landed migration:** regenerate types + flip the affected
   Database Tables registry rows to `Aligned`. The registry must stay honest.

**HOW (rules of engagement — all existing gates apply).**
- Spec sources in precedence order: **OS registries (Database Tables /
  Entities / Schema Registry) → `knowledge/target-data-model.md` (attached
  to the plugin today) → ask Maor.** Never invent a column, enum value, or
  behavior. Where the target model marks an open question, that is a task
  for Maor, not a guess.
- Work **area by area** (suggested order: meetups core first — it unblocks
  the queries task — then organizations, then subscriptions modeling, then
  messaging). For each area: draft the migration(s) + a short schema note in
  from-sivan/worklog BEFORE merging, so Maor can veto shapes cheaply.
- Every change ships as a **migration through the pipeline** (merge to
  `momlee-web` → green checks → `db-deploy`); Migration Gate applies in
  full (rollback + RLS impact + affected tables/APIs); destructive changes
  need the in-file `-- destructive-approved:` marker as usual.
- **RLS by default** (owner-or-admin, explicit public-read carve-outs only),
  soft-delete by default, `user_display_info` pattern for any user-facing
  projection, **do NOT rename live tables** (`baby_meetups` stays; product
  naming lives in code/UI). Privacy: DATA_INVENTORY updated per area.

## ✅ RULING (Maor, 2026-07-15) — your ADR-016 gate fix is APPROVED as PERMANENT (+ your Windows papercut is fixed)

Answers to today's tasks:

1. **`isAdr016PlatformPair` is the rule now, not a temporary unblock.** Maor
   confirmed: the gate simply predated ADR-016. Your implementation was kept
   exactly as-is (one native + one web-primitives file, same name/node; any
   3rd claim or web-vs-web dup still fails). We removed the TEMPORARY/revert
   comments and pushed to `momlee-web` (`2b20520`). Nothing for you to change.
2. **The Windows path papercut is fixed in the same commit:** the three gates
   that compare relative paths against forward-slash literals
   (check-component-dupes, check-web-arch SANCTIONED, check-figma-refs
   grandfather list) now normalize `path.sep` — your local runs should match
   CI. Pull `momlee-web` before your next gate run.
3. **`Buttons/Social button` node ids — received from Maor (2026-07-16) and
   PARKED.** `kind='social'` is not needed by any web screen yet, and Plan C
   (above) is now the priority track — the node ids will be posted here when
   component work resumes, so nothing will block you then.
4. **`@svgr/webpack` + the icon-library strategy + the Figma plan/seat
   question — PARKED behind Plan C** (Maor is weighing a broader decision:
   adopting one large published icon library in Figma as canonical vs
   growing the hand-curated set). The inlined-paths workaround stays
   acceptable meanwhile; no web screen work depends on it right now.

## ✅ SOLVED (2026-07-15) — the Dev Changelog mystery: it was ARCHIVED, now restored INTO Momlee OS. Log to Notion again (same id!)

Your retraction was excellent detective work and correct on every check —
and there was one missing piece neither of us had: the Dev Changelog lived
under **"Momlee Old" → 08 · Delivery**, and when the old command center was
archived after the OS pivot, the changelog was archived WITH it. That
reconciles everything:

- Your side: Notion search does not index archived pages, and fetching an
  archived id 404s on your surface → "the DB does not exist". Correct
  observation, wrong conclusion.
- Claude's side: the id kept resolving read/write on Maor's session surface,
  so rows kept landing there daily → "it's alive". Also correct, and blind
  to the archived flag.
- Nobody deleted data: all rows are intact, including your two synced
  worklog entries.

**Fix applied (Maor's call, 2026-07-15):** the database was MOVED to
**Momlee OS → 07 - Engineering**, which un-archived it. The id is
**unchanged**: `ee6d4bbb-1444-479c-b818-36f7e3951988` — so the worklog
skill, the hook, and the standup skill all keep working as-is, no edits
needed. Your decision to keep logging to this file rather than create a
second DB was right; now please: **verify you can fetch the changelog, then
sync your pending Worklog entries below into it as rows and log directly
from now on** (git fallback stays for outages only).

## ✅ RULING (Maor, 2026-07-15) — Button UNBLOCKED: bg-brand-solid drift resolved at the source

Answers to your two questions from the web-primitives worklog entry:

1. **The light-pink retint was NOT final.** The Figma was mid-edit; Maor has
   reverted the color in the file. VERIFIED against live Figma (node
   `3287:428579`, 2026-07-15): `bg-brand-solid = #b05f64` and the label is
   `text-white` again — Figma, `design-system/tokens.md`, `@momlee/tokens`,
   and the native Button all agree. No token re-sync needed.
2. **Primary button text = white** (pairs with the mauve), same as native.

So: **build the Button now, figma-first, on the TOKEN classes**
(`bg-brand-solid` + `text-white`, never a hex), mirroring the native
contract per ADR-016. Maor is still running design-system experiments in
Figma — if you ever catch a drift like this again, same protocol: stop,
log, ask (this catch was exactly right). When his palette work closes for
real, we'll announce it and run `momlee-sync-tokens` as one deliberate
sync (web + native together) — do not chase intermediate Figma states.

## ✅ DONE (2026-07-15, per Sivan) — Notion MCP reconnected to Maor's workspace (was: ACTION REQUIRED 2026-07-14)

Sivan confirmed she reauthenticated and picked Maor's workspace, and task
reads now resolve. Definitive proof pending: the NEXT worklog entry should
write straight to the Dev Changelog (no from-sivan.md fallback). If it
404s again, the step-by-step guide is preserved below.

## ~~⛔ ACTION REQUIRED (Sivan, ~2 minutes) — reconnect the Notion MCP to MAOR's workspace (2026-07-14)~~

Your "Dev Changelog is gone" 404s are solved: the DB is alive, you ARE a full
member of Maor's workspace (verified) — **your Notion MCP is simply authorized
against the WRONG workspace** (your personal one), so every Momlee OS id 404s.
Per Notion's docs the MCP "acts with your full Notion permissions" in the
workspace you authorize — so authorize Maor's. Step by step:

1. In Claude Code run `/mcp` → select the **Notion** server → **Disconnect /
   Logout** (clear the current auth).
2. Same `/mcp` menu → **Authenticate / Reconnect** → a browser window opens
   with Notion's consent screen.
3. **THE CRITICAL STEP:** at the TOP of that consent dialog there is a
   workspace dropdown showing which workspace you are granting. It defaults
   to YOUR personal workspace — **switch it to Maor's workspace** (the one
   holding "Momlee OS"; you're a Member there as sivan@applee.dev).
4. Approve ("Allow access").
5. **Verify:** back in Claude Code, ask Claude: *"fetch the Momlee OS page"*
   or run a standup — if the Dev Changelog resolves, you're on the right
   workspace. If it still 404s, repeat step 2 and double-check the dropdown.

From then on the worklog writes straight to the Dev Changelog (no git
fallback needed), and the standup skill can read your open tasks.

## ✅ DONE (2026-07-14) — GitHub invitation accepted; from-sivan.md is flowing (was: ACTION REQUIRED 2026-07-13)

Root cause found: your push to the plugin repo failed because **the
collaborator invitation to `sivan@applee.dev` is still Pending — you never
accepted it** (Maor verified in the repo settings). Until you accept, every
push to `Maorsnaim/momlee-guide` is rejected, so your from-sivan.md entries
exist only on your machine.

1. **Accept the invite:** check the GitHub invitation email to
   `sivan@applee.dev`, or open
   **github.com/Maorsnaim/momlee-guide/invitations** while logged in to your
   GitHub account. (Invites expire after 7 days — if it expired, tell Maor to
   re-send.)
2. **Deliver the stranded entries:** from your momlee-guide clone (not the
   plugin cache folder):
   ```
   git add planning/from-sivan.md
   git commit -m "from-sivan: worklog + tasks for Maor"
   git push origin main
   ```
3. **Verify:** `git log origin/main -1 -- planning/from-sivan.md` shows your
   commit. From now on the worklog skill requires commit+push+verify in the
   same step, so a silent failure like this cannot recur.

## ⛔ ACTION REQUIRED (Sivan, one-time, ~2 minutes) — add the deploy secret (2026-07-08)

The new DB pipeline (below) cannot reach Supabase until you add the repo
secret — only you have the access:

1. supabase.com → avatar → **Account → Access Tokens → Generate new token**
   (name: `momlee-ci-deploy`; copy it — shown once).
2. GitHub `sivanhasson/MomLee` → **Settings → Secrets and variables →
   Actions → New repository secret** → Name: **`SUPABASE_ACCESS_TOKEN`**,
   Value: the token.

Until then the `db-deploy` job fails on auth and migrations stay pending
(harmless, but nothing deploys).

## 🆕 NEW skill: `/momlee-standup` — session-start updates from git + Notion (2026-07-08)

Per Sivan's request: opening a session and asking **"מה חדש?" / "תעדכן
אותי" / "standup"** now pulls open work from BOTH channels — this file
(git) AND live Notion (Dev Changelog open rows + the Momlee OS Tasks DB,
including Blocked tasks) — and returns one short prioritized Hebrew summary.
Refresh the plugin and it is active (restart the session so hooks/skills
load).

## 🌿 Web MVP work happens on the NEW `momlee-web` branch (2026-07-08)

**Sivan: `git fetch && git checkout momlee-web`.** The branch was cut from the
tip of `momlee-native` (so it carries the migration baseline, the security
fixes, the CI gates, and fresh generated types — branching from
`improved_momlee_by_claude` would have lost all of those). `momlee-native`
stays frozen as the intact native reference for the future iOS return.

**Fact-check (Sivan asked): the plugin does NOT block web development.** The
plugin itself ships only the worklog hook. The CLI gates live in the app repo,
and until today they only scanned `apps/mobile` + `packages/*` — web was
UNCOVERED, not blocked (CI even excludes web lint explicitly). What changed on
`momlee-web` (commit `032cb39`): the gates now cover web too —

- **check-naming** scans `apps/web/src` (glossary synonyms, auto Figma names).
- **check-deps** enforces `apps/web/package.json`; the existing web stack
  (Next.js / Radix / Mapbox / testing, 60 packages) is allowlisted as the
  approved web set; NEW dependencies still require the DEPENDENCY GATE.
- **check-figma-refs**: every NEW web `page.tsx` declares its Figma source
  node (`// figma: <nodeId>`); the 32 pre-pivot pages are grandfathered until
  rebuilt from Figma.
- **NEW check-rtl-web**: a logical-properties RATCHET — the count of physical
  Tailwind direction classes (`ml-/mr-/pl-/pr-/left-/right-/text-left/...`) in
  `apps/web/src` may only DECREASE (baseline 543). New code uses logical
  utilities only (`ms/me/ps/pe/start/end/text-start/text-end`). When you lower
  the count, lower the BASELINE in the same change.
- `ci.yml` runs on pushes/PRs to `momlee-web` as well.

**Web gates round 2 (2026-07-08, commit `53cad5f`) — three more mechanical
gates, closing the full contract on web:**

- **check-component-dupes** — the mechanical half of the REUSE AUDIT: no two
  component files may share a normalized base name (3 legacy dups
  grandfathered: button / input / serviceslist), and no NEW primitive may
  enter an existing synonym family (Sheet/Modal/Drawer/Dialog · Badge/Chip/Tag
  · Toast/Snackbar · Select/Dropdown/Picker · Avatar · Tooltip) in
  `components/ui`, `components/shared` or `packages/ui` — extend the existing
  primitive (13 stock shadcn primitives grandfathered).
- **check-web-arch** — Supabase lives ONLY in the data layer
  (`services/`, `lib/`, `app/api/`, `app/actions/`): files touching it
  elsewhere are ratcheted (baseline 28, may only go down — move queries into
  services as you touch them); `select('*')` ratcheted (baseline 25);
  **direct PostHog is hard-banned (0)** — analytics only via the
  `@momlee/core` wrapper.
- **check-tokens-web** — hardcoded color literals (arbitrary Tailwind color
  classes `bg-[#...]` + `style` hex) ratcheted (baseline 101, may only go
  down) — new code uses design-system token classes only.

Ratchet rule reminder: when your change LOWERS a count, lower the matching
BASELINE constant in the same commit (the gate prints the number).

## 🏛️ ADR-016 (2026-07-08, Maor decided): per-platform primitives with a SHARED CONTRACT

Three facts every UI session must know:

1. **The existing web UI is a DEMO, not the product.** Every screen gets
   rebuilt Figma-first (that is why the 32 pages are grandfathered in the
   figma-refs gate). Do not extend demo components as if they were canonical.
2. **One design system, one token source.** The real web runs on
   **`packages/tokens`** (Figma-sourced; the same package that feeds native)
   and **Noto Sans Hebrew** (Heebo is wrong and gets removed). A [GAP]
   Validator tracks wiring the preset into the web Tailwind — until that
   task closes, treat any stock-shadcn value as demo debt.
3. **native<->web reuse = a SHARED CONTRACT, not shared render code**
   (universal/react-native-web was rejected): each primitive has a native
   implementation (packages/ui, RN+NativeWind) and a web implementation
   (apps/web components/ui, DOM+Radix+Tailwind) with the **same name, same
   props API, same Figma node, same tokens** — one row in components.md
   covers both. Full rationale: ADR-016 in the OS; the work: Feature
   `platform.web_design_foundation` (Critical) + 4 tasks.

**Round 2.2 (2026-07-08, per Maor): Figma node identity — one node, one
component.** A Figma screen is composed of nested component instances; the
danger is re-implementing a nested sub-component that already exists. Now:

- Every NEW component file declares its source node (`// figma: 17297:8153`;
  non-visual files: `// figma: none - <reason>`) alongside the reuse-audit
  receipt — CI fails it otherwise.
- **One Figma node = one code component**: a second file claiming the same
  node FAILS CI (this catches re-implementations even under a new name —
  verified against the real inventory: the native `Input` already carries its
  node, and a synthetic second claim was caught).
- Before building a SCREEN, Claude must enumerate its component instances
  from the Figma metadata and resolve each node against the code
  (`grep "figma: <node>"` + components.md): mapped → IMPORT (inline
  re-implementation is forbidden); unmapped → full REUSE AUDIT → CREATE with
  both headers. See momlee-figma-first step 5.
- Code Connect mappings (Figma MCP) added as a task under
  `platform.web_design_foundation` — once mapped, the design context itself
  hands Claude the existing component.

**Round 2.1 (2026-07-08, per Maor): the reuse gate is now SEMANTIC, not just
name-level.** Every NEW component file (not in the pre-pivot inventory of
185) must carry a **reuse-audit receipt** header proving the semantic REUSE
AUDIT ran before creation:

```
// reuse-audit: searched="badge, chip, pill" checked="ui/badge" verdict=CREATE reason="animated numeric pill not covered"
```

Claude runs the three searches (components.md + Figma + code, names AND
synonyms, by FUNCTION not just name), decides REUSE -> EXTEND -> CREATE, and
records the receipt; CI fails any new component without it. Writing a receipt
without running the audit violates momlee-design-system.

**DB changes — the pipeline is LIVE (2026-07-08, Maor approved).** You CAN do
everything yourself now, end to end — **no local DB required**:

1. Author a migration file in `supabase/migrations/`, commit, push/merge to
   `momlee-web`. That's it.
2. **CI tests it on a fresh database FOR you** (the `rls-tests` job spins up
   a clean Postgres in Docker, applies ALL migrations from scratch, runs the
   pgTAP invariants). Running `supabase db start` locally is optional — a
   faster feedback loop if you want it, never a requirement.
3. After `checks` + `rls-tests` are green, the **`db-deploy` job applies the
   new migrations to the LIVE database automatically** and prints
   `migration list` for verification.
4. **Nobody runs `supabase db push` against live by hand — the pipeline is
   the only path.** Standard prod discipline; the same rule binds Maor and
   Claude.
5. **Destructive migrations** (DROP TABLE/COLUMN, TRUNCATE, DELETE FROM) are
   paused by a guard until the file carries
   `-- destructive-approved: <your-name> <date> <reason>`. **You approve it
   yourself** — it is not a permission request, it is a deliberate stop,
   because destructive SQL deletes data with NO undo. Before adding the
   marker ask: is there a backup? is the data truly unneeded? would a
   soft-delete or rename-and-deprecate be safer? When unsure, consult Maor —
   but the call is yours.

**One-time setup (Sivan — you have the repo-settings access):**
1. Create a Supabase personal access token: supabase.com dashboard → your
   avatar → **Account → Access Tokens → Generate new token** (name it e.g.
   `momlee-ci-deploy`).
2. In GitHub `sivanhasson/MomLee` → **Settings → Secrets and variables →
   Actions → New repository secret**: name **`SUPABASE_ACCESS_TOKEN`**,
   value = the token.
Until the secret exists, the `db-deploy` job fails on auth and migrations
simply stay pending — nothing breaks.

## ✅ RESOLVED 2026-07-02 — migration baseline squash + repair (was ⛔ since 2026-06-22)

**The `supabase migration repair` was EXECUTED and verified on 2026-07-02
(Maor + Claude).** Live bookkeeping now equals exactly the 2 baselines
(`migration list` local==remote, `db push --dry-run` = "Remote database is up
to date", direct table read = 2 rows, data untouched). **Sivan: nothing to run
anymore** — just `git pull` on `momlee-native`; future `db push` is safe. Full
detail + rollback backup: `docs/planning/MIGRATION_BASELINE_HANDOFF.md` in the
app repo. The original context is kept below for history.

## ~~⛔ ACTION REQUIRED~~ — migration baseline squash (2026-06-22)

The app-repo migrations were **squashed to a baseline** because the old 85
incremental migrations could not rebuild a clean DB from scratch (the
Lovable-origin tables `users` / `parent_profiles` / `provider_profiles` / the
`provider_groups` family had no `CREATE TABLE`, so `supabase db start` failed).
Old migrations are archived in `supabase/_archived_migrations/`; two baselines
(`20250101000000_baseline_public_schema.sql`,
`20250101000001_baseline_storage_policies.sql`) now equal the live schema.

**Sivan: before the next `supabase db push` you MUST run `supabase migration
repair`** (bookkeeping only — it does NOT change the live schema), otherwise the
push will try to re-create objects that already exist on live. Exact commands +
rollback + verification: **`docs/planning/MIGRATION_BASELINE_HANDOFF.md`** in the
app repo. No live database changes have been made.

> **Reaffirmed 2026-07-01 — STILL PENDING.** Sivan has not yet run
> `supabase migration repair`; it remains a **hard prerequisite before any
> `supabase db push`**. It also aligns with the direction we're leaning (a clean
> schema re-baseline — see the "Notion OS" note below): the squash already
> produced clean baselines that equal live, so the repair is the bookkeeping step
> that makes the clean baseline and the live DB agree.

> **PRE-FLIGHT DONE 2026-07-02 — the handoff commands were INCOMPLETE; use the
> UPDATED doc.** A read-only verification against live confirmed the baseline
> equals the live schema object-for-object (tables/policies/views/triggers/
> functions/enums/indexes all reconcile — safe to repair). BUT live bookkeeping
> holds 91 versions vs 85 archived files: **nine provider_groups versions
> (20260602100000..20260604120000) have no local file**, so the original repair
> loop leaves them dangling and `db push` would still complain. The app-repo
> handoff doc (`docs/planning/MIGRATION_BASELINE_HANDOFF.md`, commit `537a9a7`)
> now carries the corrected sequence: step 3b (revert those 9), a final
> `supabase db push --dry-run` gate ("Remote database is up to date" or STOP),
> and a rollback recipe from
> `docs/planning/live_schema_migrations_backup_2026-07-02.csv` (a backup of all
> 91 live bookkeeping rows). **Sivan: `git pull` on `momlee-native` first** (Maor
> pushes it), then follow the UPDATED section 3 end-to-end. Repair edits
> bookkeeping only — it cannot touch schema or data.

## 🧭 Notion OS — now the planning source of truth (2026-07-01)

The product is planned in a structured **Notion "Momlee OS"**, documented in this
plugin under **`knowledge/os/`** (`00-foundations` → `90-dispositions`). Read
`00-foundations.md` first.

**Structure (the spine):** **Epic → Feature → Story → Task**, with registries
hanging off Stories — Permissions, Roles, User Types, Eligibility, Events,
Automations, Product Rules, Schema Registry, Entities, Communication Channels,
Message Categories, Design Screens / Components. A Story carries a small
human-authored core; Claude proposes the wiring; the approved result compiles to
an unambiguous spec for dev.

**Recent structural work (2026-06-29 → 07-01), all reflected in the book:**

- Epic **boundary rules LOCKED** (one capability = one owning Epic; surface ≠
  ownership; actor ≠ ownership; lifecycle boundary).
- New Epics: **Profile & Settings** (ongoing profile + cross-actor settings
  layer), **Browse & Discovery** (home / index / discovery layer; renamed from
  "Home / Discover"), **Provider Profile & Services** (Post-MVP stub reserving the
  professional-profile domain).
- MVP decision: meetup discovery = a simple index scoped to a **single experiment
  neighborhood**, **no proximity ranking** (Post-MVP; per Sivan's experiment).
- Naming locked to the **live code terms** for keys: **`parent` / `provider`**
  (not mom / professional). Names stay Mom / Professional for UI.
- Conventions: Decision Log titles in English + bilingual bodies; one decision per
  row; canonical rows first; no emojis / em-dashes; Hebrew-lead everywhere except
  Decision Log titles.
- Cleanups **verified**: Permissions + Privacy Notes DBs are dedup-clean.
  **UI-only leftovers (do in the Notion UI):** delete the 4 deprecated
  Communication Channels rows (marked "- delete"), remove the stray empty data
  source inside the Stories DB, remove the `.is` auto-links in Eligibility.

**⚠️ OPEN architecture decision (pending Maor's final go) — "Plan C":** a broad
audit (live DB + governance docs + client code + the OS model) concluded the live
DB is **pre-launch** (seed data only) and **provider-heavy**: the
provider / verification / admin / audit / RLS subsystem is mature and worth
**KEEPING**, but the product core (meetups, organizations, subscriptions,
messaging, role / status enums) is barely modeled and effectively greenfield.
Recommendation: **not a full rewrite and not incremental patching, but a clean
schema RE-BASELINE** of the product core on top of the kept backend, driven by the
OS as the spec — done now while pre-launch (near-zero migration cost). **Not
started; awaiting Maor's decision.**

## 🔐 Security audit (2026-06-09) — STATUS

**Maor ran a security audit and FIXED the holes in code** (app repo, commit
`c432dad` on `momlee-native`, report: `docs/audit/SECURITY_FIXES_2026-06-09.md`
in the app repo). What was fixed:

1. Public storage buckets → **private + signed URLs**.
2. **CRITICAL:** a broad `users` RLS policy exposed every mom's email+phone →
   removed; safe view **`public.user_display_info`** (id, display_name,
   avatar_url ONLY) is now the only way to show users to other users.
   (Pattern codified in `../knowledge/security.md` §24.)
3. `.env` was tracked in git → untracked + `.gitignore` fixed.
4. 16 hardcoded Mapbox tokens → moved to env.
5. 6 legacy Edge Functions (service-role + open CORS) → **deleted from the repo**.
6. `/api/geocode` had no auth/rate-limit → now authenticated + 30 req/min.

### ✅ DEPLOYED TO LIVE (2026-06-12, via Maor's token)

The security migration (20260609200000) + onboarding_events (20260610120000)
+ phone-first signup (20260612210000) are APPLIED and VERIFIED on production:
buckets private, users RLS + user_display_info live, events table with RLS,
email/display_name nullable, phone-aware handle_new_user. A real OTP SMS
request returned 200. database.types.ts regenerated from live.

### ⚠️ Still PENDING from the original list

The code is fixed, but the fixes are not live until these run against
**production Supabase** (coordinate with Maor before touching live):

- [ ] Apply migration `20260609200000` to the **live** database
      (buckets + `users` RLS + `user_display_info` view).
- [ ] **Delete the 6 legacy Edge Functions from the Supabase dashboard**
      (they're gone from the repo, but still deployed and callable).
- [ ] **Rotate the exposed tokens** (Mapbox; any key that lived in the
      committed `.env`), update env/secret stores accordingly.

**Until then:** treat the live DB as if the holes are still open — don't build
anything that depends on the old public buckets or on querying `users` directly.
New code MUST follow the fixed patterns (signed URLs, `user_display_info`,
env tokens) so it's correct the moment the deployment lands.

## Machine enforcement: full map (2026-06-11, commits ce1147a/940a6eb/921f9b1)

Three layers now run on every push/PR in the app repo: **eslint** (12+ rule
families: architecture, RTL incl. raw-Text ban, tokens-only incl. font-weights
+ fontFamily + arbitrary px, permissions, analytics SDK + PII payloads + event
format, select('*'), dates) and **CI scripts** (deps allowlist, migration
RLS/retention/rollback, RTL manifest flags, naming auto-layer-names +
synonyms, secrets/.env/public-env, figma screen-node refs). All verified
positive + negative.

- [ ] **Backfill Figma node refs** on the 5 grandfathered screens (index,
      welcome, phone, otp, name): add `// figma: <nodeId>` with the REAL node
      ids (don't guess — pull from the Figma map), then empty GRANDFATHERED in
      scripts/momlee/check-figma-refs.mjs.
- [ ] **Consider eslint-plugin-react-native-a11y** (accessibility gate at lint
      level: roles/labels/touchables) — new devDependency, so it needs the
      DEPENDENCY GATE + allowlist + verification it works with the Expo flat
      config. Recommended next mechanical step alongside RLS tests in CI.

## Worklog is now MECHANICALLY enforced (2026-06-11) — hooks in the plugin

The plugin now ships Claude Code **hooks** (`hooks/hooks.json`, Node script):
commit in a MomLee repo → session marked worklog-pending → ending the turn is
blocked once until a Dev-Changelog row is logged (or the git fallback /
explicit "trivial" note). This is harness-level enforcement — independent of
the model's memory.

- [ ] **Sivan: after the next `/plugin install momlee-guide@momlee`, restart
      the session** so the hooks load (Claude Code asks to approve plugin
      hooks on install — approve). Node is required (already in your env).

## Ten resilience & boundary gates (2026-06-11, Maor) — NEW

A second wave of gates is live (plugin 0.13.0):

1. **Module Boundaries** — onboarding ↛ meetups ↛ subscriptions ↛ moms
   directly; cross-module = the other module's PUBLIC service only
   (architecture.md + momlee-react-native).
2. **Error Handling Standard** — every error: specific Hebrew user message +
   dev log (no PII) + taxonomy analytics event when relevant + recovery
   action. Never "Something went wrong" (new skill **momlee-resilience**).
3. **Four states** — Loading/Empty/Error/Success on every data screen, via the
   shared pattern; missing Empty/Error designs in Figma = blocked part.
4. **Device Permission Gate** — location/notifications/camera/photos/contacts
   ONLY via the `@momlee/core/permissions` wrapper; in-context requests;
   denial is a designed state (momlee-privacy).
5. **Feature Flags** — every large/sensitive feature behind `<feature>_enabled`
   (provider_subscriptions_enabled, mom_discovery_enabled,
   identity_verification_enabled); kill = flip flag, never delete code.
6. **Copy source of truth** — new channel `knowledge/copy-guidelines.md`
   (feminine second person, warm+specific, Figma word-for-word, sensitive
   areas need Maor); no invented user-facing text, ever.
7. **Authorization ≠ UI visibility** — hiding a button is not a permission;
   DB/RLS/service enforce even when the screen is hidden (momlee-security #5).
8. **Offline/Network failure** — OTP, meetup, message, upload, location all
   define failure behavior; input never lost; no auto-retry on writes;
   airplane-mode test before done (momlee-resilience).
9. **Upload Gate** — every upload declares size limit, type allowlist, privacy
   class, bucket, access policy, delete policy (+ EXIF/GPS stripping)
   (momlee-privacy).
10. **Deletion/Retention** — no new data model without retention/deletion
    behavior; SOFT delete by default; new mandatory Retention field in the
    Migration Gate (momlee-privacy + momlee-data-inventory + momlee-migration).

Pending decisions/tasks:

- [ ] **Maor: feature-flags kill-switch mechanism** (server-controllable —
      e.g. a Supabase config row vs build-time only).
- [ ] **Maor: account-deletion cascade map** — decide per data type (photos,
      messages, children records, meetups, verification refs): cascade /
      anonymize / retain — fill the retention column in data-inventory.md.
- [ ] **Maor: Figma designs for shared Empty/Error states** (the four-states
      pattern needs designed visuals before the shared component is built).
- [ ] **Sivan: scaffold `@momlee/core/permissions`** (wrapper like analytics)
      when the first permission feature lands.

## Naming Gate + Glossary + milestone audits (2026-06-11) — NEW

- New skill **momlee-naming**: fires before naming ANYTHING new (file, folder,
  component, hook, service, route, table, column, enum, analytics event) and
  before translating Figma layers to code. Glossary terms only, format per
  artifact kind, printed NAMING check for tables/components/events.
- New channel **`knowledge/glossary.md`**: the canonical terms (mom, provider,
  child, meetup, organization, subscription, verification) with the frozen DB
  mappings and FORBIDDEN synonyms. **No synonyms, ever** — a new entity term
  enters only by Maor adding it to the glossary.
- **Figma Layer Naming Guard** (inside momlee-naming): auto-generated layer
  names (`Frame 12`, `Group`, `Rectangle`, `Component 1`) are never copied to
  code — resolve via component set → annotation → parent context → glossary,
  or STOP and ask.
- **`/momlee-audit` is now MANDATORY before closing any milestone** (sprint /
  complete flow / release) — see momlee-worklog. Milestone isn't Done until
  the audit ran and High findings are fixed or explicitly accepted by Maor.
- Drift cleanup: the stale live reference to `UnderlineField` in momlee-rtl
  now reads `Input` (historic note only).

## Bind every session + retro-audit (2026-06-11) — NEW

Two new pieces close the enforcement loop:

- [x] **DONE (2026-06-11, commit `d60a34f` on momlee-native):** the session
      contract was MERGED into the app repo's existing `CLAUDE.md` as a
      "READ FIRST" section (the web-era guide below it was kept; the plugin
      wins on conflicts). Awaiting push by Maor. Every Claude session in the
      repo now reads the contract at session start.
- [ ] **Run `/momlee-audit` once** in an app-repo session (after updating the
      plugin) — full compliance review of everything built so far against all
      the new gates. Report only; fixes get approved by Maor first.

## Architecture & Naming Review Obligation (2026-06-11) — NEW skill

New skill **momlee-architecture-review**: during ANY task, existing
inconsistencies (naming conflicts, duplicate concepts, drift, legacy patterns,
architectural violations) must be surfaced — never silently ignored. Protocol:
continue on the canonical convention, no automatic refactors, no renames
without approval, document + recommend in the "Architecture Observations"
format (Issue / Impact / Recommendation / Priority), and log durable findings
to the worklog so Maor sees them. Consistency over perfection — but obvious
better directions get surfaced, not buried. No action needed beyond updating
the plugin.

## Core Technology Stack — Iron Law (2026-06-11, Maor) — READ stack.md

`knowledge/stack.md` now carries the full canonical stack rules: Expo Router
(file-based, never React Navigation directly), TanStack Query (centralized
query keys), RHF+Zod for every form, the backend flow (minimum acceptable:
Screen → Service → Repository; Screen → Supabase forbidden), the PostHog
wrapper (`analytics.track` ✅ / `posthog.capture` ❌), **expo-secure-store**,
Reanimated + Gesture Handler, Gorhom sheets, **Zustand (restricted list:
session/locale/direction/feature-flags/temp-onboarding/global-UI — never
server data)**, **date-fns**, **expo-location**, and the Dependency
Governance 4 questions (no approval = STOP).

- [ ] **Sivan: migrate the Supabase session storage to expo-secure-store.**
      The current client (`apps/mobile/src/lib/supabase.ts`) persists the
      session in AsyncStorage — now forbidden for tokens. Use a SecureStore
      storage adapter for the Supabase client. (Note: SecureStore values are
      capped at 2KB each — Supabase sessions can exceed it; the standard
      pattern is an AES-key-in-SecureStore + encrypted-payload-in-AsyncStorage
      adapter, e.g. Supabase's documented Expo recipe. Pick with Maor if
      unsure.)
- [ ] **Sivan: add `expo-secure-store`, `date-fns`, `expo-location`, `zustand`**
      as each becomes needed (all stack-approved now; Expo modules are Expo
      Go safe).

## State Management Guard + Dependency Budget (2026-06-11) — NEW, in momlee-react-native

**State has ONE home** (Maor's decision; TanStack Query + React Hook Form are
now stack-approved): UI state = local `useState`; server state = **TanStack
Query** in feature hooks (queryFn → service → repository); form state =
**React Hook Form** + Zod resolver; global client state = ONLY if nothing else
fits, per-store Maor approval (no Zustand-by-default). Never copy server data
into a client store.

**Dependency Budget:** don't add a dependency if the feature can be built in
under 100 LOC with the existing stack; every dependency prints a
`DEPENDENCY GATE` block (under-100-LOC test, justification, cost) and needs
Maor's approval + a `stack.md` row in the same change.

- [ ] **Sivan: add `@tanstack/react-query` + `react-hook-form` (+ Zod resolver)**
      when the first data-fetching/form screen needs them — both JS-only,
      Expo Go safe. Until then nothing to install.

## Analytics: PostHog via wrapper (2026-06-11, Maor's decision) + Analytics Gate

**Tool decided: PostHog** (supersedes the first-party-only plan from
2026-06-10). Everything goes through ONE abstraction —
`@momlee/core/analytics` (`analytics.track('otp_requested', {...})`) — with
`providers/posthog.provider.ts` as the only file importing the SDK. **Never
call PostHog (or any analytics SDK) directly from screens/components.** The
event taxonomy is STABLE (seed list in `knowledge/analytics.md`); payloads are
PII-free (`baby_age_range` ✅, `baby_birth_date` ❌).

Also live: skill **momlee-analytics** — every feature prints an
`ANALYTICS GATE` block at plan time (events / verification / success KPI
traced to WSMA) and isn't "done" until events are SEEN landing.

- [ ] **Sivan: scaffold the wrapper** — `packages/core/analytics/`
      (`analytics.ts`, `analytics.types.ts` typed taxonomy,
      `providers/posthog.provider.ts`), fail-soft without env key; migrate the
      existing `logEvent` console stub into it (same event names).
- [ ] **Sivan: add `posthog-react-native`** (now stack-approved — verify Expo
      Go compatibility on install; if it needs native code, defer the SDK to
      the EAS dev-client stage and keep the fail-soft console provider until then).
- [ ] **Maor: provide the PostHog project key + host** (`EXPO_PUBLIC_POSTHOG_KEY`,
      EU/US cloud choice) and update the privacy policy / store labels to name
      PostHog as a processor (data-inventory row already updated).

## Migration Gate (2026-06-11) — NEW, hard gate before any DB change

New skill **momlee-migration**: every database change (table/column, RLS
policy, enum, view, bucket, schema-touching Edge Function) must print a
`MIGRATION GATE` block BEFORE any SQL: migration file + rollback plan + RLS
impact + affected tables + affected APIs. New tables ship RLS in the same
migration; destructive changes need a backup step + Maor's approval; the live
DB stays Maor-coordinated; `database.types.ts` is regenerated before and after.
No action needed beyond updating the plugin.

## Component Reuse Audit (2026-06-11) — NEW, hard gate before any new component

"Reuse before create" is now a **proven audit**, not a guideline (upgraded in
**momlee-design-system**; figma-first rule #2 and /momlee-screen step 6 point at
it). Before creating ANY component, Claude must search `components.md` (both
tables) + the Figma inventory + the code, by name AND synonyms
(Sheet/Modal/Drawer, Badge/Chip/Tag…), print a `REUSE AUDIT` proof block, and
verdict REUSE → EXTEND → CREATE. Base primitives (Button, Input, Card, Avatar,
Badge, Sheet) are presumed to exist — a second one is a duplicate. A CREATE
verdict requires the component in Figma first + a row in `components.md` in the
same change. No action needed beyond updating the plugin.

## AI Prompt Guard (2026-06-11) — NEW, applies to everything

New skill **momlee-prompt-guard**: if information is not in an official source
(Figma, annotations, design-system/, knowledge/, planning/, or an explicit
instruction), it does not exist — STOP and ask; never invent. A missing
component (e.g. no Time Picker in the design system) means that part is
**blocked until Maor designs it** — build the rest, log the blocker. This is
also iron rule #4 in **momlee-figma-first**. No action needed beyond updating
the plugin — just know the rule is now binding for every Claude session.

## Architecture Gate (2026-06-11) — NEW, applies to all data wiring

The layered call chain **Screen → Hook → Service → Repository → Supabase** is now
a hard rule (see `../knowledge/architecture.md` → "The layered call chain" and the
updated **momlee-react-native** skill). Screens never import Supabase; business
logic lives only in services; only the repository layer touches the client.

- [x] **DONE (2026-06-11, commit `ce1147a` on momlee-native): machine
      enforcement is LIVE in the app repo.** eslint gates (Architecture, RTL
      physical classes/keys, tokens-only hex, Permission wrapper, Analytics
      wrapper, Dates) in `apps/mobile/eslint.config.js` — CI runs lint, so a
      violation fails the build. Plus `scripts/momlee/check-deps.mjs`
      (Dependency Governance vs `stack-allowlist.json` — adding a dep requires
      editing the allowlist = explicit approval) and `check-migrations.mjs`
      (post-2026-06-12 migrations: RLS with CREATE TABLE + `-- retention:`
      declaration), wired as a CI step. Verified: current code passes clean;
      a violating test file trips all five gate families.
- [x] No existing screen calls Supabase directly (verified by the lint run —
      0 errors).

## Dev-environment notes (already in effect)

- The app is pinned to **Expo SDK 54** so the App Store **Expo Go** runs it on a
  real iPhone via QR (SDK 56 was rejected by Expo Go). Don't bump the SDK
  without checking Expo Go support. See `../knowledge/dev-environment.md`.
- pnpm monorepo with `node-linker=hoisted` (required for Metro). After pulling
  dep changes: `pnpm install` from the repo root.

## /momlee-audit round 2 — fixes landed (2026-06-11, commit a5a024a)

DONE (Maor approved "fix everything"): H1 session→expo-secure-store ·
H2 not-configured bypass now __DEV__-only · H3 analytics wrapper in
@momlee/core (typed taxonomy + providers; screens use analytics.track) ·
G1 lint gates now cover packages/ui+core (caught & fixed physical offsets in
OtpInput) · R1 CTA Loading state wired · L2 fontByWeight token · import dups.

STILL OPEN:
- [ ] PostHog: Maor's project key + dependency gate → providers/posthog.provider.ts
- [ ] Taxonomy decision (Maor): adopt `onboarding_step_viewed` or migrate to seed events
- [ ] Component FILE naming: conventions say kebab-case; component files are
      PascalCase (RN standard) — Maor to rule
- [ ] R3 offline handling (with the shared error/empty states work)
- [ ] M4 regenerate database.types.ts after the live migration applies
- [ ] Live security deploy (see the security section above)

## Mechanical-enforcement additions (2026-06-12)

Per Maor ("mechanical checks over memory text"): the asset-color lesson is
now machine-enforced — `check-token-provenance.mjs` in CI (every token value
must cite its Figma variable on the same line; caught touchTarget on run #1)
+ an eslint rule banning hex literals in color-like JSX props. Also live:
the TextInput writingDirection rule in check-rtl.mjs (caught OtpInput run #1).

## Still open after the live deploy (2026-06-12)
- [ ] Delete the 6 legacy Edge Functions from the Supabase dashboard (still deployed)
- [ ] Rotate Mapbox tokens (old exposure) + ROTATE THE TWILIO AUTH TOKEN (pasted in chat)
- [ ] Maor: delete the temporary Supabase access token (used for db push, no longer needed)
- [ ] Configure Test Phone Numbers (972528547424=123456 format) to test without SMS costs

