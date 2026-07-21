> Canonical, Maor-maintained. Edit here; the skills reference this. Flows to Sivan via git.

# Canonical Glossary — one entity, one name

**The law: no synonyms, ever.** Every domain entity has exactly ONE canonical
term. New code uses it; UI uses the product term; the DB keeps its legacy
names (never renamed — see `data-model.md`). A new entity term enters the
language ONLY by Maor adding it here first.

| Canonical term | DB / legacy code (do NOT rename) | Product / UI term (he) | FORBIDDEN synonyms in new code |
|---|---|---|---|
| **mom** | `parent`, `parent_profiles` | אמא / Mom | mother, mum, user, member, customer |
| **provider** | `providers` (thin MVP record, admin-managed) + `provider_profiles` (frozen full user profile, Post-MVP) | בעל/ת מקצוע / Pro / Professional | professional (in code), pro (in identifiers), expert, vendor, business |
| **child** | `baby_*` columns (e.g. `baby_birth_date`), children records | ילד/ה | baby (in NEW identifiers — legacy `baby_*` stays), kid, infant |
| **meetup** | `meetups` (renamed from `baby_meetups` 2026-07-20, ratified), `meetup_type` (mom\|pro) | מפגש | event, gathering, session, activity |
| **organization** | `organizations`, `organization_members` | ארגון | org (in identifiers), company, business, group |
| **subscription** | subscription/billing columns | מנוי | plan, membership, package |
| **verification** | `verifications` | אימות | KYC (in identifiers — fine in prose), validation (reserved for input validation), approval |

## Derived rules

- **DB names are frozen — EXCEPT during the Plan C re-baseline.** `parent` and
  `provider_*` stay as they are; `baby_meetups` was renamed to `meetups`
  (2026-07-20, pipeline-deployed pre-launch, ratified by Maor 2026-07-21) —
  the one window where renames cost nothing. New renames still require a
  flagged decision, not a silent migration.
- **`provider` is ONE concept at two fidelities, not two entities:** the thin
  admin-managed `providers` record (MVP — no provider users exist; saves
  re-typing host info per PRO meetup, gives moms a clickable host card) and
  the frozen rich `provider_profiles` (Post-MVP provider-as-user). When the
  provider epic revives, the thin record gains a `user_id` / merges into the
  rich profile. The dupes/naming gates must not treat them as a collision.
- **UI copy** uses the product term (from Figma/i18n), never the code term —
  a mom never sees the word "parent", a Pro never sees "provider".
- **Two names for one concept = a bug.** If you find an existing synonym in
  the codebase, that's a **momlee-architecture-review** finding — report it,
  don't silently add a third name.
- **A concept not in this table** (e.g. a new entity) = STOP — Maor names it
  and adds the row here first (**momlee-prompt-guard**).

## Format conventions (summary — full: `conventions.md`)

- Files/folders: `kebab-case` · Components: `PascalCase` (Figma component
  names) · Hooks: `useX` · Packages: `@momlee/<name>`
- Routes (expo-router): file-based, `kebab-case` segments
- Tables/columns/enums: `snake_case` (exact names from `data-model.md`)
- Analytics events: `object_action`, `snake_case`, past tense (`analytics.md`)
- i18n keys: `namespace.context.key`
