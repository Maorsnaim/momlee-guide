---
name: momlee-os-intake
description: Use whenever Maor or Sivan share a conversation, meeting notes, a spec, or any raw product text that needs to land in the Momlee OS Notion. Distills the text into the right artifacts (Decision Log / Product Rules / Epics / Features / Stories / Tasks / Eligibility / Permissions / Roles / User Types / Events / Automations / Communication Channels / Message Categories / Privacy Notes / Technology Stack / Schema Registry / Entities / ADRs / Principles / Validators / Design Screens / Components), each created from its DB template with EVERY field filled (esp. Key), fully cross-linked, bilingual (he / en). MANDATORY before any write: scan the OS to guarantee no duplicate and no contradiction. Trigger on "put this in Notion", "סדר את זה ב-Notion", pasted transcripts, or any product decision / flow text.
---

# Momlee OS Intake — conversation in, perfect OS out

A conversation becomes structured, buildable Momlee OS records. The bar is **perfect**: right artifact type, every field filled, real `Key`, full template body, full cross-linking, bilingual, **zero duplicates, zero contradictions, zero stray links**. This file is the operational procedure; the conceptual model lives in `../../knowledge/os/` — read `00-foundations.md` first, then `10-intent-dbs.md`, `20-feature-story.md`, `30-task-automation.md`, `40-registries.md`, `50-behavior-messaging.md`, `60-validators.md`, `70-engineering.md`. Tooling = the `claude_ai_Notion` MCP (needs the operator's Notion connected AND access to the Momlee OS workspace; if absent, this skill cannot write — say so). Notion writes are authorized; **Supabase / live code are READ-ONLY**.

## THE IRON RULE — scan before you write (never skip)

**No `notion-create-pages` / `notion-update-page` call happens before a scan of the OS confirms two things:**

1. **No duplicate** — the concept does not already exist (in any naming variant / language). If it exists → **enrich that row, do not create a second**.
2. **No contradiction** — the new item does not conflict with an existing **Decision Log** entry, **Product Rule**, **Eligibility** condition, **Permission**, or another record. If it conflicts → **STOP. Surface both sides to the human. Do not write until resolved** (usually: mark the old Decision `Superseded` + log the new one, or reconcile the rule). Never let two live records disagree.

A write with no prior scan is a defect, even if the content is correct. When in doubt, scan wider.

## The pipeline (phases, always in order)

- **P0 Extract & classify** — split the text into ATOMIC items (one sentence often holds a decision + an open question + a screen + a rule; never merge). Map each atom to one or more artifact types (table below). Decided facts, open questions, and "check X" tasks are DIFFERENT artifacts.
- **P1 Scan** (the IRON RULE) — for each atom, run the **Scan matrix** below: query the target DB **and every related DB** for duplicates and contradictions, and to find the canonical rows you will link to. Space queries to avoid 429.
- **P2 Draft from template** — fetch the target collection to get its current `default_page_template` + schema (options can change). Draft the row: author-core fields, real `Key`, and the full bilingual template body. Never assume a field/option — read the live schema.
- **P3 Wire** (the Wiring matrix) — resolve every relation to a **canonical existing row**. If a needed target does not exist (e.g. the trigger Event isn't in the registry), **STOP and ask "create it?"** — never invent a link silently, never point at the wrong row.
- **P4 Verify completeness** — run the Completeness checklist. Every author-core field + relevant relation filled; bilingual; Key correct; no trailing period; `מיטאפ` not `מיטאף`; no stray auto-link.
- **P5 Approval gate** — present the drafted row(s) + the chain + anything left Open, and get the human's OK (respect Maor's deep-review pace: per-record, explicit approval, surface contradictions, don't "close corners"). Only genuine forks need a question; otherwise log faithfully and confirm.
- **P6 Write + post-write verify** — write; then **re-fetch** and confirm the body, the `Key`, and that each relation landed **dual** (both sides). Confirm you did not create a duplicate. Report what was created / linked / left Open.

## Classification guide (signal → artifact)

| Signal in the text | Artifact | Key format |
|---|---|---|
| A settled choice ("we decided", "no Y for now") | **Decision Log** `Decision` / `Decided`, fill `Final Decision`; if it dictates ongoing behavior ALSO make a **Product Rule** | title = EN statement |
| An unresolved choice ("should we…?") | **Decision Log** `Open Question` / `Open`, fill `Current Answer` | title = EN question |
| "check / investigate / task for Sivan" | **Decision Log** `Open Question` (or `Assumption` if believed-true), task in `Current Answer` | title = EN |
| An enforceable ongoing behavior | **Product Rule** | `epic.descriptor` |
| A whole capability domain | **Epic** (capability, never an actor) | snake_case |
| A capability made of several stories | **Feature** (under an Epic) | `epic.feature` |
| One concrete user/system behavior / screen-step | **Story** (under a Feature) — the HUB | `epic.feature.story` |
| A condition that gates access ("18+", "verified") | **Eligibility** | `actor.condition` |
| A who-can-do-what | **Permission** | `epic.entity.action` |
| A stable actor / authorization role | **Role** | `domain.role` |
| The account-level actor type (mom / professional / admin) | **User Type** | snake_case |
| Something happened worth firing / measuring | **Event** (`Event Kind` = Domain/Analytics/Both, ③) | `entity.action_past` |
| "the system then sends / refreshes / does X" | **Automation** | `epic.trigger.outcome` |
| An outbound transport (in-app/push/email/sms/whatsapp) | **Communication Channel** (transport only; ③-ish lookup) | `channel` |
| A consent class for messages (transactional / marketing) | **Message Category** | `snake_case` |
| Personal data collected / used | **Privacy Note** | `data_type` (snake_case) |
| A vendor / library ("Mapbox", "Persona") | **Technology Stack** (CHECK FIRST — most exist) | vendor slug |
| A data field implied ("email verified status", "radius") | **Schema Registry** (③ mirror of code) | `table.field` |
| A domain object (Meetup, Mom Profile, Organization) | **Entity** | snake_case |
| An engineering decision / an architectural principle | **ADR** / **Principle** (authored engineering DBs) | per DB |
| A rule→check mapping (CI / ESLint / Claude-gate) | **Validator** (the enforcement bridge) | per DB |
| A screen / state with a Figma frame | **Design Screen** (③, Platform field) | per DB |
| A reusable UI piece | **Component** (③) | per DB |

A single atom usually produces a **chain**: decided behavior → Decision (Decided) + Product Rule (`Source Decision` = that decision) + Story (`Product Rules`) + Permission/Eligibility + Event + Privacy. Build the whole chain and link it.

## SCAN matrix — what to query before writing each artifact

For every artifact: query for a **duplicate** (Name + Key, all languages/variants) and for a **contradiction**. Also query the DBs whose rows you will **link to**, so you link the canonical row, not a guess. Minimum scan per artifact:

- **Decision Log** → Decision Log (dup + any decision it supersedes/contradicts), Product Rules (does a live rule already say the opposite?).
- **Product Rule** → Product Rules (dup/conflict), Decision Log (its `Source Decision`), Eligibility + Permissions (the access it references).
- **Epic** → Epics (dup + the boundary rules in `00-foundations.md`: is this really a new capability domain, or does it belong to an existing Epic?).
- **Feature** → Features + its Epic.
- **Story** → Stories (dup), its Feature, and every hub relation below (so you can wire it).
- **Eligibility** → Eligibility (dup/conflict), Roles, Permissions, Entities, live `supabase-momlee` for `Source of Truth`.
- **Permission** → Permissions (dup — note the historic prose-vs-formal split), Roles, Eligibility, Entities.
- **Role / User Type** → Roles + User Types (dup + the Role↔User-Type boundary), live `user_roles.role` (canonical values = `admin` / `parent` / `provider`).
- **Event** → Events (dup), Stories that fire it, Automations it triggers, Privacy Notes.
- **Automation** → Automations (dup), Events (trigger + created), Eligibility (conditions), Communication Channels, Message Categories, Stories, User Types.
- **Communication Channel / Message Category** → both DBs (transport vs consent-class — never conflate), Technology Stack, Privacy Notes.
- **Privacy Note** → Privacy Notes (dup — snake_case canonical), Entities, Schema Registry, Events, Automations.
- **Technology Stack** → Technology Stack (most vendors exist — enrich, don't add).
- **Schema Registry** → Schema Registry (dup), Entity, live `supabase-momlee` for the real column.
- **Validator** → Validators (dup), the Product Rule / ADR / Principle it enforces.
- **Design Screen / Component** → the same ③ DB (dup) + the Stories it serves.

## WIRING matrix — the relations to ensure (never miss one)

Fetch the target DB's schema to get the exact property names, then wire every relation that applies. The **Story is the hub** — its live relations (verified) are: `Feature`, `Permissions`, `Roles`, `Eligibility`, `Events`, `Automations`, `Product Rules`, `Privacy Notes`, `Schema Registry`, `Components`, `Design Screens`, `Validators`, `Decision Log`, `Tasks`. Wire all that apply; a Story that touches data with no `Schema Registry`/`Privacy Notes`, or an action with no `Permissions`, is probably under-wired.

- **Permission** → `Granted Roles`, `Required Eligibility`, `Target Entities`, `Related Stories`, Epic.
- **Eligibility** → `Roles`, `Target Entity`, `Related Permissions`, `Related Stories`, Epic (+ `Source of Truth` vs live DB).
- **Role** → `Domain`, `Role Type`, `Scope`, `Granted Permissions`, `Required Eligibility`, `Related Entities`, `Related Stories`, `Related to User Types`.
- **User Type** → `Primary Entity`, `Related Roles`, its Stories.
- **Event** → `Event Kind`, `Used In Story`, `Triggers Automations`, `Created By Automations`, `Privacy Notes`, `Sync Status`.
- **Automation** → `Trigger Event`, `Events Created`, `Conditions`(Eligibility), `Communication Channels`, `Message Templates`, `Related Stories`, `Target User Types`, `Timing`.
- **Product Rule** → `Source Decision`, `Epic`, `Related Stories`, `Related Permissions`, `Related Eligibility`, `Enforcement Layer`.
- **Decision Log** → `Type`, `Status`, `Decision Date`, `Creates Product Rule?`, `Related Epics/Features/Stories`, `Related Product Rules`.
- **Communication Channel** → `Message Categories`, `Default Technology`(Tech Stack), `Privacy Notes`, capability fields (`Requires Template` / `Supports Scheduling` / `Supports User Opt-out` = channel capability, not policy).
- **Message Category** → `Consent Class`, `Communication Channels`.
- **Privacy Note** → `Related Entity`, `Schema Fields`, `Communication Channels`, `Events`, `Related Automations`, `Related Stories` + legal fields (`Legal Basis`, `Sensitive Data`, `Data Subject`, `Publicly Visible`).
- **Entity** → `Entity Type`, `Database Tables`, `Privacy Notes`, `Related Features`, `Related to Eligibility/User Types`, `Status`.
- **Schema Registry** → `Data Type`, `Source of Truth`, PII/Sensitivity/Exposure, Entity link.
- **Validator** → the `Related ADRs` / `Related Principles` / Product Rule it enforces + the concrete check (CI / ESLint / Claude-gate). An unmapped rule = a visible `[GAP]`.
- **Technology Stack** → `Layer`, `Categories`, `Purpose` (EN), `Decision Status`, `Implementation Status`; link target for Channels' `Default Technology`.
- **Design Screen** → `Platform`, Figma link, `Related Stories`, `Sync Status`.

## Per-DB templates (how each row is written)

Fetch the collection to get the current `default_page_template`; create with `template_id`, or `apply_template` on an existing row. Bodies to FILL with real bilingual content (not the example text):

- **Story** (7 sections): Summary / User Story / Preconditions / Main Flow / Success State / Error & Blocked States / Acceptance Criteria. (System Story variant: Summary / Goal / Scope / Implementation Notes / Dependencies / Observability & Validation / Out of Scope / Acceptance Criteria.)
- **Feature** (12): Summary / Feature Goal / Problem / Scope / Out of Scope / Main Feature Flow / Key States / Success / Error & Blocked / System Impact / Open Questions / Acceptance Criteria.
- **Epic** (7): Summary / Epic Goal / Scope / Out of Scope / Core Features / Key User Types / Acceptance Criteria (+ an "Ownership Boundary" callout where the boundary rules apply).
- Each section = a `### n. עברית / English` heading + a `<callout color="gray_bg">` holding `Hebrew` / `---` / `English`.
- **Decision Log**: title = concise **English** statement; `Final Decision` / `Current Answer` = bilingual `he / en`. (The one DB whose title is English-first.)
- **Automation / Privacy / Permission / Eligibility / Event / Role / User Type / Tech Stack / Schema / Channel / Message Category / Validator / Design Screen** = property-only (blank body) — fill every field.

## Conventions (LOCKED)

- **Key formats:** Epic `snake_case`; Feature `epic.feature`; Story `epic.feature.story`; Permission `epic.entity.action`; Event `entity.action_past`; Eligibility `actor.condition`; Automation `epic.trigger.outcome`; Rule `epic.descriptor`; Schema `table.field`; Role `domain.role`. Never leave the template placeholder. Epic set incl. `profile_settings`, `browse_discovery`, `provider_profile`, `business_insights`, `platform`.
- **Name vs Key:** Name = human prose ("Create a meetup"); `Key` = machine id. Never swap them.
- **Live code terms in keys:** `parent` (mom) and `provider` (professional), matching live `user_roles.role`; product term (Mom / Professional) stays in Names/UI.
- **Every field full:** all author-core + relevant relations, consistent format, bilingual `he / en` (only exception: Tech Stack `Purpose` = EN). Unknown → Open Question or "pending Sivan approval", never blank, never invented.
- **Hebrew writing (Maor):** (1) never start a line with an English word — lead with Hebrew (`ה-Epic`, not `Epic`); (2) `מיטאפ`, never `מיטאף` (fix on touch); (3) English term → Hebrew then `(English)`; (4) field values (non-body) do NOT end with a period; (5) no em-dashes, no emojis — plain hyphens.
- **Channels = transport only** (`in_app`/`push`/`email`/`sms`/`whatsapp` + `system` sentinel); consent class lives in Message Categories.

## Contradiction protocol (when P1 finds a conflict)

1. Do NOT write.
2. State the conflict: the existing row (Name + Key + the conflicting field) vs the new intent.
3. Recommend the resolution: usually (a) mark the old Decision `Superseded` and log the new one, or (b) split "two decisions in one row" into two, or (c) enrich the existing row. If a decision genuinely reverses a prior one, the prior row must be flipped to `Superseded` — never leave both live.
4. Get the human's call, then execute the resolution, then proceed.

## For Sivan — adding a record through Claude

Describe what you want to add in plain language (he or en). Claude will scan the OS for duplicates/contradictions, pick the right DB, create the row from its template, fill every field (prose Name + real Key + bilingual body), cross-link the whole chain, and report what it did + any Open Questions for you. You don't need the field rules — just the product intent; Claude applies the conventions and STOPS to ask on a genuine fork or a conflict. (Requires your Notion connected + access to the Momlee OS workspace; otherwise Claude will tell you and fall back.)

## MCP execution gotchas (hard-won)

- **Relations are finicky.** `notion-create-pages`: a relation property takes **ONE page-URL string** (a JSON array is rejected "expected string"; a comma-joined multi-URL string is rejected "Invalid page URL"). So set ONE target at create, add the rest via `update_properties` afterward or from the dual side — then **re-fetch to confirm**. Dates: use the expanded key `date:<Field>:start` = `YYYY-MM-DD`. Select single value = bare option name.
- **`notion-update-page` content** needs REAL newlines, not `\n` escapes (it renders `\n` literally). `content_updates` = array of `{old_str,new_str}`; `replace_content` param is `new_str`.
- **The API cannot delete/archive a row**, and `in_trash` fails (404) on a SECONDARY data source inside a multi-source DB. So dedup/removal is **UI-only**: migrate relations to the canonical row, rename the loser `"<name> (duplicate - delete)"` + set Deprecated, and flag Maor to delete in the UI.
- **Schema DDL** (`notion-update-data-source`): `COMMENT 'text'` for field docs — never put `;`, `(`, `)`, `:`, or the geresh `׳` inside a COMMENT. `ALTER COLUMN SET SELECT(...)` cannot recolor an EXISTING option — restate existing options by NAME ONLY (no color), give colors only to NEW options; a full restate drops any option you omit, so list them all.
- **Stray auto-links:** a dotted key whose segment is a TLD (`mom.is` → `.is`, also `.co`/`.it`/`.io`/`.app`) gets auto-linked in a property; the reliable fix is removing the link in the UI — flag it.
- **`fetch` can return a stale cached page** right after a write — trust the write result or re-fetch a little later.
- **Rate limits (429)** on bursts of queries (`collection_router_upstream_429`) — space them; updates are less limited than queries.

## Live collection IDs (ALWAYS `fetch` the collection before writing — schemas/options/templates change)

Epics `38c450ad-0ae6-8083-8b24-000bb72162be` · Features `af4450ad-0ae6-83a8-870c-071f7d3422de` · Stories `31b450ad-0ae6-82e3-abfb-0708511ccbaf` · Tasks `e8d450ad-0ae6-8225-a4f8-87288039ecd2` · Decision Log `c6d450ad-0ae6-83d0-a8c3-874956dcef8a` · Product Rules `109450ad-0ae6-8368-b946-87a19af52cb5` · Permissions `38c450ad-0ae6-805b-8ef7-000b98298c33` · Eligibility `38c450ad-0ae6-80b2-9b71-000ba769e406` · Roles `0fc450ad-0ae6-83f8-bdba-075612f7109d` · User Types `38c450ad-0ae6-801a-8075-000b963a54fa` · Events `051450ad-0ae6-8227-be0a-0703e3aca4d0` · Automations `4c4450ad-0ae6-837f-99d2-078e3e5c75f6` · Communication Channels `91c450ad-0ae6-8288-a490-077c0de5eba9` · Message Categories `b9a48566-0d22-4a8e-9c5d-6d0fca4e0c51` · Privacy Notes `95e450ad-0ae6-8340-a650-07933d7dbc4b` · Technology Stack `eb7450ad-0ae6-835b-92c2-871abf9d263f` · Entities `6b9498d7-f228-4008-bfb7-1e853b4530d7` · Schema Registry `111fdc18-78d4-4830-b0b7-760351059f2f` · Validators `299450ad-0ae6-82b9-b2e7-0761f4a35aae` · Components `4a2b09f4-87ec-4035-94ea-a2b3660826ca` · Design Screens `0d0c8989-d0c0-4b69-8a93-7701e4a7a9c6`. Not cached (search / fetch to locate): **ADRs, Principles, Notification Templates, Database Tables, APIs**. (Dev Changelog `ee6d4bbb-1444-479c-b818-36f7e3951988` is handled by `momlee-worklog`, not intake.)

## Definition of done

Every atom is represented; each record was **scanned first (no duplicate, no contradiction)**, created from its template with a full bilingual body + real Key + every relevant relation wired (and verified dual); conventions hold (bilingual, no emojis/em-dashes, Hebrew rules, no trailing period, no stray links); conflicts were surfaced and resolved (old decisions flipped to Superseded, not left live); open items captured as Open Questions; approval taken; a short report lists what was created, linked, resolved, and left open.
