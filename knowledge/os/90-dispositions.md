# Momlee OS — Dispositions (what stays, what merges, what leaves Notion)

> The triage decisions for the whole DB map. Status: LOCKED (2026-06-28).

## Merged

- **Events + Analytics Events → one `Events` DB** with an `Event Kind`
  (domain / analytics / both). Avoids two tables for the same concept.

## Demoted / re-shaped

- **Communication Channels → tiny lookup DB** (Reference Entity) — one canonical
  list, picked by relation. (Not a standalone heavy DB, not a per-field select.)
- **Message Categories → tiny lookup DB** — carries the `Consent Class`
  (transactional / marketing) for compliance.
- **Notification Templates → ③ mirror** from code (React Email). Pointer only.

## Source of truth = repo / plugin (NOT a hand-maintained Notion DB)

- **Engineering section** — refined (2026-06-28, because Validators links to
  `Related ADRs` / `Related Principles`):
  - **ADRs** — a **full authored Notion DB** (architecture decisions: Context /
    Decision / Consequences / Alternatives / Status / Implementation Status); the
    engineering parallel to Decision Log. Validators links to it. NOT thin. See
    `70-engineering.md`.
  - **Engineering Principles** — full authored Notion DB (see `70-engineering.md`).
  - **Technology Stack** — REVISED (2026-06-28, after field review): it IS a
    useful **OS DB** (structured vendor registry: Layer, Categories, Purpose,
    Pricing, Monthly Cost, Free Tier, Required Env Vars, SDK Package, Common
    Mistakes, Decision/Implementation Status) and a link target (Communication
    Channels → Default Technology). **Keep it as a Notion DB.** The plugin's
    `knowledge/stack.md` stays the narrative + Dependency-Governance rules; this
    DB is the structured vendor data. Cross-workspace fixes: `Communication
    Channels`→old `384450ad` (dup of current `Communication Channels 1`→
    `91c450ad`); `Privacy Notes`→old `4ae83d0e` — re-point/dedup.
- **Data Inventory / Privacy Notes** — canonical = the repo
  `docs/compliance/DATA_INVENTORY.md` (already a per-feature Definition-of-Done
  artifact). In Notion, privacy is a **light field on Entity / Story** (PII type,
  sensitivity) that feeds the canonical doc — not a separate authored inventory.

## Net DB map (after triage)

Hierarchy: Epic · Feature · Story · Task.
Identity/Access: User Types · Roles · Permissions · Eligibility.
Decisions/Rules: Decision Log · Product Rules.
Data: Entities · Database Tables (③) · Schema Registry (③).
Design: Components (③) · Design Screens (③) · Design Tokens (③).
Behavior: Events (③) · Automations · Communication Channels (lookup) ·
Message Categories (lookup) · Notification Templates (③).
Compliance: privacy field on Entity/Story → repo Data Inventory.

Everything ③ is synced from code/Figma with a mandatory `Sync Status`; nothing in
③ is hand-authored content.

## Known alignment fixes (apply during live-Notion alignment)

A running list of concrete fixes the live Notion needs (the design is in the book;
these are execution items):

> **Progress (2026-06-29 → 07-01).** RESOLVED: Events + Analytics merged
> (`06 - Analytics Events` is now a filtered VIEW of Events; the stale
> "wrong embedded DB" note was removed); Message Categories rebuilt + seeded;
> Communication Channels rebuilt as the 5-transport + `system` lookup;
> Automations author-summaries + Timing added; Permissions + Privacy Notes
> verified **dedup-clean**; Decision Log format locked (English title, bilingual
> body) + Decision Date / Creates-Rule; new Epics (Profile & Settings, Browse &
> Discovery, Provider Profile & Services) + the 4 boundary rules; Stories deep
> pass (Status enum, `Area` colour field, `Validators` relation, real bodies on
> all 65). STILL OPEN (UI-only, no API — the Notion MCP cannot delete rows or a
> secondary data source): delete the 4 deprecated Communication Channels rows,
> remove the stray empty data source inside Stories, remove the `.is` auto-links
> in Eligibility. STILL OPEN (design): Notification Templates build-out,
> `Sync Status` on the ③ DBs, the APIs-registry decision, and a
> **State-Machine / Lifecycle registry** (flagged load-bearing for the `attended`
> North Star). See `planning/open-tasks.md` → "Notion OS" for the Plan-C
> architecture decision.

- **GLOBAL RULE — no relation may point outside Momlee OS.** On EVERY page
  without exception, every relation must target a **current** Momlee OS
  collection. A relation pointing at an old-workspace collection is broken →
  find the current in-OS DB and re-point. (Should become a Validator: "no
  cross-workspace relation".) Old → current re-point map:
  - old Modules `382450ad` → **Epics** `38c450ad…8083`
  - old Features `383450ad` → **Features** `af4450ad…22de`
  - old Stories `b320e186` → **Stories** `31b450ad…cbaf`
  - old Entities `a744c2a5` → **Entities** `6b9498d7`
  - old Product Rules / Schema `386450ad…` → **Product Rules** `109450ad` /
    **Schema Registry** `111fdc18`
  - old Privacy Notes `4ae83d0e` → **Privacy Notes** `95e450ad`
  - old Behavior `384450ad…` (Events/Automations/Channels/Processors) → current
    **Events** `051450ad` / **Automations** `4c4450ad` / **Communication
    Channels** `91c450ad`
  - old States `6a38ad1d` → no current States DB (decide if needed)
  Known offenders so far: Validators, ADRs, Principles, Privacy Notes,
  Communication Channels. **Must scan ALL pages.**
- **Communication Channels + Privacy Notes** — stale copies from the old Notion;
  relations dangle to old pages. Re-point or rebuild.
- **Decision Log `Create Draft Rule` button** — the Decision→Product-Rule
  automation is broken; fix it.
- **`Sync Status` missing** on most ③ DBs (Events, Components, Design Screens,
  Design Tokens, Database Tables, APIs, Templates) — add it.
- **`Module`→`Epic` rename** on every relation/field still labelled "Module".
- **Duplicate relation cleanup** — `Related Stories 1` (Permissions, Eligibility,
  Decision Log), `Related Rules`→Stories (Product Rules), `Story`+`Stories`
  (Tasks), `Epic Description` (Epics), `Related Roles`/`Required Eligibility`
  cross-links (Entity/Role/Eligibility).
- **Automations** — add author summaries (`Trigger/Logic/Outcome Summary`),
  `Status`, `Scope`, `Events Created`; existing relations are the ② bucket.
- **Broken pages** — `04 - Message Categories` and `06 - Analytics Events` each
  embed the WRONG database (an "Automations" DB). Rebuild: Message Categories as a
  tiny lookup (Name/Key/Consent Class); Analytics Events folds into `Events`
  (`Event Kind = Analytics`).
- **Notification / Email Templates** — live DB is nearly empty (only `Name`).
  Build out as a ③ mirror with **two pointers** (like Components): `Name`, `Key`,
  `Channel`, `Message Category`, **`Figma Link`** (design), `External Link`
  (code / React Email), `Sync Status`, `Status`. Body stays in code/Resend.
- **Privacy Notes / Data Inventory** — rich + correct shape (Legal Basis,
  Retention, App Store Notes, Data Subject, Sensitive, Publicly Visible) — KEEP as
  authored governance DB. BUT ~all its relations point at OLD collections
  (Automation/Channel/Event `384450ad`, Entity `a744c2a5`, Feature `383450ad`,
  Product Rules / Schema Fields `386450ad`) → re-point to current OS.
- **Communication Channels** — `Privacy Notes` relation points at old `4ae83d0e`
  → re-point. (Otherwise a good lookup: Default Technology, Requires Template,
  Supports Scheduling/Opt-out.)
- **Design Screens** — add `Sync Status`; the `Related Screens` relation is
  mislabeled (points at Stories) → rename `Related Stories`.
- **Database Tables** — add `Sync Status`; add a relation to **Entity** (it is
  standalone today — Entity↔table is the entity→implementation link). Has Source
  of Truth + PII/Sensitivity/RLS/Soft-Delete already.
- **Design Tokens** — **DROPPED from Notion (2026-06-28, Maor).** Source of truth
  = Figma Variables → `@momlee/tokens` (code) → plugin `design-system/tokens.md`
  (usage rules/naming). Nobody consumes tokens from Notion (Claude pulls them via
  Figma MCP + the Tailwind preset), and per-token rows with literal `Value` are
  content+drift, not a pointer. Unlike Components/Schema Registry, tokens carry no
  Notion-only value.
- **User Flows** — considered; **defer** (not an MVP DB). A flow = a Feature's
  Stories in order + the Figma prototype; a dedicated DB overlaps Feature/Story/
  Figma. Only justified later as **cross-feature journeys**, and even those are
  better tracked via Events/funnel. (The old link was from workspace `37f450ad`.)
- **APIs** — no APIs DB exists in the current OS. Decide: add an APIs registry, or
  fold API mapping into Database Tables.
- **Task** — live is minimal (`Name`, `Story`, dup `Stories`); add `Task Type`,
  `Status`, `Scope`, collapse the dup relation (per `30-task-automation.md`).
