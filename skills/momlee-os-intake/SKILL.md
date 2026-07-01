---
name: momlee-os-intake
description: Use whenever Maor or Sivan share a conversation, meeting notes, a spec, or any raw product text that needs to land in the Momlee OS Notion. Distills the text into the right artifacts (Decision Log / Product Rules / Stories / Features / Epics / Eligibility / Permissions / Events / Automations / Privacy Notes / Tech Stack / Schema Registry / Channels / Roles / Entities), each created from its DB default template with EVERY field filled (esp. Key), fully cross-linked, bilingual (he / en), no duplicates, no stray auto-links. Trigger on "put this in Notion", "סדר את זה ב-Notion", pasted transcripts, or any product decision/flow text.
---

# Momlee OS Intake — conversation in, perfect OS out

A conversation between Maor and Sivan becomes structured, buildable Momlee OS records. The bar is **perfect**: right artifact type, every field filled, real Key, full template body, full cross-linking, bilingual, zero duplicates, zero stray links. This skill is the operational procedure; the conceptual model lives in `../../knowledge/os/` (read `00-foundations.md`, `10-intent-dbs.md`, `20-feature-story.md`, `30-task-automation.md`, `40-registries.md`, `50-behavior-messaging.md`). Tooling = the `claude_ai_Notion` MCP. The live DB is read-only-safe to write (Notion writes are authorized; Supabase/code are NOT).

## The pipeline (always in this order)

1. **Extract** — split the text into atomic items. One sentence often holds several (a decision + an open question + a screen + a rule). Do not merge them.
2. **Classify** — map each atom to one or more artifact types (table below). Decided facts, open questions, and "check X" tasks are DIFFERENT artifacts — never file an open question as a closed Story.
3. **Discover existing** — BEFORE creating anything, query each target DB (`notion-query-data-sources`, `SELECT "Name","Key",url ...`) for rows that already exist. Lookup/registry DBs (Technology Stack, Roles, Communication Channels, Entities, User Types, Schema Registry) are **pre-seeded** — reuse, never recreate. Enrich an existing row instead of duplicating it.
4. **Create from template** — create each row with `notion-create-pages` passing `template_id` (the DB's `default_page_template`, from a `fetch` of the collection). The template carries the page-body sections + the page icon. For rows that already exist, use `notion-update-page` command `apply_template`.
5. **Fill everything** — every author-core field, above all **`Key`** (conventions below). Fill the **template body sections** with real bilingual content (not the example text). Clear the template's stray default `Assign to` people (`"Assign to": null`).
6. **Cross-link** — set the relations. They are DUAL: setting one side fills the other. Wire the Story (the hub) to its Permissions / Events / Product Rules / Decisions / Eligibility / Automations / Privacy Notes / Schema Registry; then the non-Story-derivable links (Permission→Required Eligibility + Granted Roles + Target Entities; Rule→Source Decision + Epic; Eligibility→Roles + Target Entity + Epic; Automation→Events + Channels + Target User Types; Privacy→Related Entity; Feature→Epic; Story→Feature).
7. **Verify + report** — fetch a sample, confirm body + Key + links, report what was created/linked and what was left as Open Questions.

## Classification guide (signal → artifact)

| Signal in the text | Artifact | Notes |
|---|---|---|
| A settled choice ("we decided", "we want X", "no Y for now") | **Decision Log** Type=`Decision`, Status=`Decided`, fill `Final Decision` | If it dictates ongoing behavior, ALSO create a **Product Rule** (set its `Source Decision`). |
| An unresolved choice ("should we...?", "is it possible...?") | **Decision Log** Type=`Open Question`, Status=`Open`, fill `Current Answer` | Do NOT make it a Story yet. |
| "check / investigate / task for Sivan" | **Decision Log** Type=`Open Question` (or `Assumption` if it's a believed-true finding), put the task in `Current Answer` | These are the Sivan tasks. |
| A belief stated as likely-true ("per the docs, X can...") | **Decision Log** Type=`Assumption`, Status=`Needs Input` | |
| An enforceable ongoing behavior ("unverified mom sees limited meetups") | **Product Rule** (Rule Type: Access/Eligibility/Business Logic/Privacy/Trust&Safety/Notification/Subscription) | Link `Related Stories`, `Related Permissions`, `Related Eligibility`, `Source Decision`, `Epic`. |
| A concrete user/system behavior, one screen-step or flow ("mom enters email") | **Story** (under a Feature) | Full 7-section body. |
| A product capability made of several stories ("Identity Verification") | **Feature** (under an Epic) | Full 12-section body. |
| A whole capability domain ("App Shell & Navigation") | **Epic** | Full 7-section body. Capability-only, never an actor — see `00-foundations.md`. |
| A condition that gates access ("must be 18+", "verified") | **Eligibility** `actor.condition` | e.g. `mom.is_adult`. Link Roles, Target Entity, Related Permissions/Stories. |
| A who-can-do-what ("verified mom can create a meetup") | **Permission** `epic.entity.action` | Link Granted Roles, Required Eligibility, Target Entities. |
| Something happened worth firing/measuring ("phone verified") | **Event** `entity.action_past` | Event Kind = Domain/Analytics/Both. ③ — Sync Status. |
| "the system then sends / refreshes / does X" | **Automation** `epic.trigger.outcome` | Fill Trigger/Logic/Outcome summaries + Trigger Type + Automation Type; link Events, Channels, Target User Types/Roles. |
| Personal data is collected/used (phone, email, location, child, ID) | **Privacy Note** (Data Type) | Legal Basis, Sensitive, Data Subject (Child = extra care), Publicly Visible, Related Entity. Feeds repo `docs/compliance/DATA_INVENTORY.md`. |
| A vendor/library ("Mapbox", "Twilio Verify", "Persona") | **Technology Stack** | CHECK FIRST — most exist. Layer, Categories, Decision/Implementation Status. |
| A data field implied ("email has a verified status", "radius") | **Schema Registry** `table.field` | Data Type, Source of Truth, Sensitivity, Exposure, Contains PII. ③ mirror of code. |
| A screen / state mentioned | **Design Screen** (③, from Figma) — usually only register if a frame exists | Platform field (iOS/Android/Web/Admin Web). |
| A reusable UI piece | **Component** (③, from code/Figma) | Don't story "component appears on screen X". |

A single atom commonly produces a **chain**: a decided behavior → Decision (Decided) + Product Rule (Source Decision = that decision) + Story (Related Rule) + Permission/Eligibility + Event + Privacy. Build the chain and link it.

## Key conventions (LOCKED)

`Epic` = snake_case area (`auth_access`, `onboarding`, `meetups`, `trust_safety`, `app_shell`, `professionals`, `organizations`, `subscriptions`, `notifications`, `communication`, `business_insights`, `platform`, `admin`). `business_insights` = Pro/Org analytics dashboards (was `analytics`); `platform` = Platform / Infrastructure enabler Epic (PostHog/Sentry/logging/flags). Product telemetry is NOT an Epic - it is the Events registry + a per-Story DoD gate. `Feature` = `epic.feature`. `Story` = `epic.feature.story`. `Permission` = `epic.entity.action`. `Event` = `entity.action_past_tense`. `Eligibility` = `actor.condition`. `Automation` = `epic.trigger.outcome`. `Rule` = `epic.descriptor`. `Schema` = `table.field`. Every created row MUST have a real Key (never leave the template placeholder `epic.feature_slug`).

## Naming, completeness & dedup (LOCKED — read before editing any DB)

- **Name vs Key:** the title field is **human-readable prose** ("Create a meetup", "Mom is identity verified"); the `Key` is the machine identifier in its DB format. Never put the key in the Name or prose in the Key.
- **Every field full:** fill ALL author-core fields + the relevant relations on every row, consistent format across rows, bilingual `he / en` (the ONE exception: Technology Stack `Purpose` is English-only). Unknown value → Open Question or "pending Sivan approval", never blank, never invented. See `../../knowledge/os/00-foundations.md` (Name vs Key, completeness, dedup).
- **One concept = one row:** discover existing rows first. The API **cannot delete/archive a row** — to retire a duplicate: migrate its relations to the canonical row FIRST, then rename it `"<name> (duplicate - delete)"` + set Deprecated status (if the DB has one) + flag Maor to delete in the UI.
- **Channels = transport only** (`in_app`/`push`/`email`/`sms`/`whatsapp` + `system` sentinel); categories live in **Message Categories** (consent class), delivery surface is a field. **Eligibility/Permission `Source of Truth`/columns** are verified against the live `supabase-momlee` DB; if absent, mark "pending Sivan approval". See `../../knowledge/os/50-behavior-messaging.md` + `10-intent-dbs.md`.

## For Sivan — adding a record through Claude

Tell Claude what you want to add in plain language (he or en). Claude will: pick the right DB, create the row **from that DB's template**, fill every field (prose Name + real Key + bilingual content), cross-link it, and report what it created + any open questions for you to decide. You do not need to know the field rules — just describe the product intent; Claude applies the conventions above. If something is genuinely undecided, Claude leaves it as an Open Question for you rather than guessing.

## Template bodies to FILL (do not leave the example text)

- **Story** (7 sections): Summary / User Story / Preconditions / Main Flow / Success State / Error & Blocked States / Acceptance Criteria. (Stories DB no longer has `User Story` / `Acceptance Criteria` as *properties* — they live in the body.)
- **Feature** (12 sections): Summary / Feature Goal / Problem / Scope / Out of Scope / Main Feature Flow / Key States / Success State / Error & Blocked / System Impact / Open Questions / Acceptance Criteria. (Features DB has no `User Value` property anymore.)
- **Epic** (7 sections): Summary / Epic Goal / Scope / Out of Scope / Core Features / Key User Types / Acceptance Criteria.
- Each section is a `### n. עברית / English` heading + a `<callout color="gray_bg">` with `Hebrew \n --- \n English`.
- **Automation / Privacy / Permission / Eligibility / Event / Tech / Schema** = property-only (their templates have a blank body); fill all fields.

## Quality gates (every record)

- **Bilingual**: Hebrew, then `---`, then English, in descriptions and body callouts.
- **No emojis, no em-dashes** anywhere (Maor). Plain hyphens. (The OS tab format already uses ✓ / ✗ marks in When-To-Use/When-Not — those are fine, they match the established format.)
- **Hebrew writing rules (Maor, 2026-06-30) — apply to every Hebrew string:** (1) never start a line/sentence with an English word and then continue — lead with Hebrew so the line stays RTL-aligned (write `ה-Epic`, not `Epic`); (2) it is `מיטאפ` (Meetup, slang), NEVER `מיטאף` (fix on touch); (3) an English term goes Hebrew-first with the English in parentheses, e.g. `פיצ'ר (Feature)`, not a bare `Feature`; (4) in field values (`Description` and any non-body property) do NOT end with a period — periods are only for page-body Content; (5) no em-dashes, plain hyphens.
- **RTL-first** in any UI guidance.
- **No stray auto-links**: Notion auto-links text that looks like a URL. A dotted key whose segment is a TLD (`mom.is_adult` → `.is`, also `.co`, `.it`, `.io`, `.app`) gets `mom.is` turned into a hyperlink in the property. Re-set the value as plain text; if Notion re-links it, the only reliable fix is removing the link in the UI (the API auto-links). Flag any such key for a UI link-removal.
- **No duplicates**: discovery step is mandatory. If you already created a duplicate, you cannot trash a row via the API — rename it `"<name> (duplicate - delete)"` + set a Deprecated status and flag Maor to delete it in the UI.
- **Open ≠ closed**: investigations and undecided questions go to Decision Log as Open Questions; never fabricate a decision or a closed Story.
- **Ask only on genuine forks**: when Maor explicitly asks for a recommendation (e.g. "should email be verified in onboarding?") give a recommendation and confirm; otherwise log faithfully and proceed. Respect his deep-review pace — surface contradictions, do not "close corners".

## MCP execution gotchas (hard-won)

- `notion-update-page` content needs **REAL newlines**, NOT `\n` escapes (it renders `\n` as literal "n" and escapes tags). (`notion-create-pages` content tolerates `\n`, but prefer real newlines everywhere.)
- Relations are set as a **JSON-array string of page URLs**: `"Epic": "[\"https://app.notion.com/<id>\"]"`. Multi-select single value = the bare option name.
- `notion-create-view` builds a configured linked view (`FILTER "P" = "v"; GROUP BY "P"; SHOW ...`) but **only appends to the end** of a page; a `<database data-source-url=...>` block in markdown does NOT resolve.
- `apply_template` injects the template's default `Assign to` + placeholder `Key` — override Key, clear Assign to.
- Schema DDL (`notion-update-data-source`): `COMMENT 'text'` for field descriptions; **never put `;` or the Hebrew geresh `׳` inside a COMMENT**; `ALTER COLUMN SET SELECT(...)` cannot recolor an EXISTING option (restate existing colors, only new options pick colors); `in_trash:true` trashes a data source (there is no row-trash).
- Notion `fetch` can return a **stale cached** page right after a write; verify with a fresh fetch a bit later or trust the write result.
- Watch for **rate limits (429)** on bursts of queries — space them or batch.

## Live collection IDs (verify with `fetch` if unsure)

Epics `38c450ad-0ae6-8083-8b24-000bb72162be` · Features `af4450ad-0ae6-83a8-870c-071f7d3422de` · Stories `31b450ad-0ae6-82e3-abfb-0708511ccbaf` · Tasks `e8d450ad-0ae6-8225-a4f8-87288039ecd2` · Decision Log `c6d450ad-0ae6-83d0-a8c3-874956dcef8a` · Product Rules `109450ad-0ae6-8368-b946-87a19af52cb5` · Permissions `38c450ad-0ae6-805b-8ef7-000b98298c33` · Eligibility `38c450ad-0ae6-80b2-9b71-000ba769e406` · Roles `0fc450ad-0ae6-83f8-bdba-075612f7109d` · User Types `38c450ad-0ae6-801a-8075-000b963a54fa` · Events `051450ad-0ae6-8227-be0a-0703e3aca4d0` · Automations `4c4450ad-0ae6-837f-99d2-078e3e5c75f6` · Communication Channels `91c450ad-0ae6-8288-a490-077c0de5eba9` · Privacy Notes `95e450ad-0ae6-8340-a650-07933d7dbc4b` · Technology Stack `eb7450ad-0ae6-835b-92c2-871abf9d263f` · Entities `6b9498d7-f228-4008-bfb7-1e853b4530d7` · Schema Registry `111fdc18-78d4-4830-b0b7-760351059f2f` · Components `4a2b09f4-87ec-4035-94ea-a2b3660826ca` · Design Screens `0d0c8989-d0c0-4b69-8a93-7701e4a7a9c6`.

Template ids live on each collection's `default_page_template` (fetch the collection to get the current one).

## Definition of done

Every atom from the text is represented; each record is created from its template with full body + real Key + all fields; the chain is cross-linked; bilingual; no emojis/em-dashes; no duplicates; no stray links; open items captured as Open Questions; a short report lists what was created, linked, and left open for Maor/Sivan.
