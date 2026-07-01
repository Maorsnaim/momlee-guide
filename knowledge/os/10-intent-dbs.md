# Momlee OS — Intent DBs (type ①) — Author Cores

> For each type-① DB: **Author core** = the only fields a human types. Everything
> else is inherited (back-relation), Claude-proposed (②), or derived/synced.
> Keys per `00-foundations.md`. Status: LOCKED unless noted.

---

## Epic  ·  `epic`

Purpose: a top-level product area / system domain (`meetups`, `auth_access`).

- **Author core (~4):** `Name`, `Epic Type` (Core Product / Marketplace / Social
  / Platform / Admin / Compliance / Analytics / Infrastructure), `Product
  Segment`, `Owner`. Optional: short `Description`.
- **Derived:** `Key` ← from Name (snake_case).
- **Back-relation (never typed):** `Related Features / Entities / Automations /
  Decisions`.
- **Rollup (derived, not a relation):** `Related Stories` ← via Features.
- **Workflow:** `Status`.
- **Dropped (LOCKED 2026-06-28):** `Epic Description` (duplicate of
  `Description`); the direct `Related Eligibility` relation (eligibility attaches
  to Permissions/Stories — surface per-Epic via rollup only if ever needed).

## Entity  ·  `entity` (singular; DB table = plural)

Purpose: a core business object (`meetup`, `mom_profile`, `subscription`).

- **Author core (~5):** `Name`, `Key` (singular snake_case), `Epic`, `Entity
  Type` (Core / Profile / Join / Event / System / External / Reference),
  `Description`. Optional: `Business Owner`, `Lifecycle States` (fill once clear).
- **Add (③ link):** `Database Tables` relation (entity→table map; `meetup` ↔
  `meetups`) — needed by the compile step.
- **Privacy:** NO manual privacy field. `Privacy Class` is a **rollup** from the
  field-level PII tags in the Schema Registry; the detailed inventory is compiled
  into the Data Inventory (see `90-dispositions.md`).
- **Back-relation (never typed):** `Related Automations`, `Related to Eligibility`
  (Eligibility.Target Entity), `Related to Permissions` (Permission.Target
  Entities), `Related to User Types` (authored side = `User Type.Primary Entity`).
- **Dropped (LOCKED 2026-06-28):** `Related to Roles` (no clear purpose).
- **Workflow:** `Status`.
- Cleanup: rename the auto-generated `Related to … (…)` relation labels.

## User Type  ·  `mom` | `professional` | `admin`   (Organization is NOT a user type — it's a tenant Entity)

Purpose: WHO the user is at the product level — drives which app surface they get.
Few, stable. **MVP user types = `mom` + `admin`; `professional` is Post-MVP**
(per the admin-curated pro-meetups decision — professionals are not self-serve in
the MVP, so the professional/organization/multi-tenant cluster is deferred).

- **Author core (~6):** `Name`, `Key` (bare identity), `Description`,
  `Primary Entity` (→ Entities; e.g. mom → `mom_profile`),
  `Primary Role` (→ Roles; default role, e.g. mom → `users.mom`),
  `Product Surface` (MVP: Mom App / Admin Console · Post-MVP: Professional
  Dashboard / Organization Dashboard).
- **Back-relation (never typed):** `Related Roles` (authored on `Role.User Type`),
  `Targeted Automations` (renamed from `Related Automations`).
- **Workflow:** `Status`.

## Role  ·  `domain.role` (`users.mom`, `organizations.owner`, `admin.reviewer`, `system.automation`)

Purpose: the authorization role a user holds. Belongs to a User Type. Carries
**no runtime state** (state lives in Eligibility).

- **Author core (~5):** `Name`, `Key` (`domain.role`), `Domain` (Mom /
  Professional / Organization / Admin / System — namespace), `Description`,
  `User Type` (parent; renamed from the auto `Related to User Types (…)`).
  Plus `Scope` + `Status`.
- **Claude proposes / back-relation (never typed):** `Granted Permissions` (②,
  derived from Product Rules), `Related Stories` (Story.Roles),
  `Targeted Automations`.
- **Dropped (LOCKED 2026-06-28):** `Role Type` (overlaps Domain); `Required
  Eligibility` (eligibility gates Permissions/actions, not the role identity);
  `Related Entities` (mirror of the dropped Entity→Roles); the duplicate
  `Related Roles` relation (it pointed at Stories).
- **Rows held now (keys + namespaces only; do NOT overfill permissions/eligibility yet):**
  - `Scope: MVP` — `users.mom`, `admin.admin`, `admin.reviewer`, `system.automation`.
  - `Scope: Fast-follow` (minimal core only) — `professionals.professional`,
    `organizations.owner`, `organizations.member`.

## Eligibility  ·  `actor.condition` (`mom.is_identity_verified`, `pro.has_active_subscription`)

Purpose: a runtime condition that must be TRUE *now* to use a permission. This is
the single home for all "verified / subscribed / onboarded" state.

- **Author core (~7):** `Name`, `Key`, `Eligibility Type` (Verification /
  Subscription / Onboarding / Profile Completion / Org Status / Account Status /
  Context), `Condition Statement`, `Failure Behavior` (product decision),
  `Epic` (owning area), `Target Entity` (what it checks). Optional: `Check
  Timing`. Plus `Scope` + `Status`.
- **③ from code (Claude fills):** `Source of Truth` (table/API/service),
  `Enforcement Layer` (Frontend/API/Backend/RLS/QA…).
- **Back-relation:** `Related Permissions`, `Related Product Rules`,
  `Related Stories`, `Related Automations`.
- **Dropped (LOCKED 2026-06-28):** duplicate `Related Stories 1`; the `Roles`
  relation (mirror of the dropped Role→Required Eligibility — eligibility links
  to Permissions, not Roles directly).
- **Workflow:** `Status`.
- **Conventions (LOCKED 2026-06-29):** `Name` = prose ("Mom is identity verified"),
  `Key` = `actor.condition`. `Enforcement Layer` always includes at least one
  server layer (RLS / Backend) plus Frontend for UX — security is server-decided.
  `Source of Truth` = the authoritative `table.field` (or service), verified against
  the live DB; if it does not exist yet, write "currently Notion, requires Sivan
  approval" with the proposed column. `Condition Statement` + `Failure Behavior` are
  bilingual. Note: the live mom table is `parent_profiles` (parent = mom), and
  subscriptions / organizations / parent-verification tables do not exist yet.

## Permission  ·  `epic.entity.action` (`meetups.meetup.view_full_details`)

Purpose: the ACTION that exists — not who gets it, not when.

- **Author core (~6):** `Name`, `Key`, `Permission Type` (Entity Action /
  Business Capability / System), `Target Entities`, `Action` (create / view /
  join / publish / invite / review …), `Description`. Plus `Scope` + `Status`.
- **Claude proposes → approve (②):** `Granted Roles`, `Required Eligibility` —
  derived from the relevant Product Rule, never typed raw per permission.
- **Back-relation:** `Related Stories`, `Related Product Rules`.
- **Dropped (LOCKED 2026-06-28):** duplicate `Related Stories 1`. (Created/edited
  metadata = Notion system fields — ignore.)
- **Workflow:** `Status`.
- **Conventions (LOCKED 2026-06-29):** `Name` = prose ("Create a meetup"), `Key` =
  `epic.entity.action`. Fill `Granted Roles` + `Required Eligibility` +
  `Target Entities` + bilingual `Description` on every permission. The permission set
  must be COMPLETE per the stories — e.g. a "mom leaves meetup" story requires
  `meetups.registration.leave`. Two naming styles drifted (formal "Epic — Entity —
  Action" vs prose); **prose is canonical**, dedup duplicates by `Key`.

## Product Rule  ·  `epic.subject.constraint`

Purpose: the human, enforceable product statement ("Only verified moms can view
full meetup details").

- **Author core (~6):** `Name`, `Key`, `Epic` (owning area — add; missing in
  live), `Rule Type` (Access / Eligibility / Business Logic / Privacy /
  Trust&Safety / Notification / Subscription), `Rule Statement` (one binding
  sentence), `Source Decision` (→ Decision Log). Plus `Scope` + `Status`.
- **Claude proposes → approve (②):** `Related Permissions`, `Related Eligibility`,
  `Enforcement Layer` (Frontend/API/Backend/RLS/QA…).
- **Back-relation:** `Related Stories`.
- **Dropped (LOCKED 2026-06-28):** the mislabeled duplicate `Related Rules`
  relation (it points at Stories).
- **Workflow:** `Status` — live enum has only `Draft`; extend to the standard
  lifecycle (Draft / Approved / In Use / Needs Review / Deprecated).

## Decision Log  ·  (no technical key — human title)

Purpose: open questions, decisions, assumptions, risks. The most human DB —
almost everything is author-entered. **No `Scope`** (not a release-bucketed build
unit; `Priority`, incl. `Blocking`, carries urgency).

- **Author core:** `Name` (phrase as a question while open), `Type` (Open
  Question / Decision / Assumption / Risk), `Status` (Open / Needs Input /
  Proposed / Decided / Revisit Later / Superseded), `Current Answer`,
  `Final Decision`, `Decision Date`, `Creates Product Rule?` (Yes/No/Maybe),
  `Owner`, `Priority`, `Related Epics`.
- **Back-relation:** `Related Features`, `Related Stories`, `Related Product
  Rules` (the rule created from it).
- `Create Draft Rule` button — keep. **FIX (live Notion): the current automation
  does not properly create/link the Product Rule — the Decision→Rule transition
  is broken.** Track in the Notion-alignment task.
- **Dropped (LOCKED 2026-06-28):** duplicate `Related Stories 1`.
