# Target Data Model — MomLee

> **ADOPTED as the working reference for Plan C (Maor, 2026-07-16).** This is
> the per-table target spec for the schema re-baseline. Precedence when
> sources disagree: (1) the Momlee OS registries (Database Tables / Entities /
> Schema Registry — they are newer and carry the honest live Sync Status),
> (2) this document, (3) ask Maor — never invent. Some sections predate later
> decisions (e.g. D1 organizations = IN MVP, pro-meetups admin-curated in MVP,
> 2026-06 security fixes already live); the OS reflects those.

**Date:** 2026-06-03
**Status:** Proposal for team review (audit + planning; no code/migrations written)
**Audited against:** `MomLee-web-ref` @ `improved_momlee_by_claude` — 82 migrations in `supabase/migrations`, `packages/supabase/src/database.types.ts`, `FEATURES_SPEC.md`, `MIGRATION_PLAN.md`, plus governance docs (`docs/06-data-model.md`, `docs/12-existing-build-reference.md`) and the authoritative schema digest (`docs/audit/CURRENT_BACKEND_STATE.md`).

---

## 0. How to read this document

This is the **target** normalized data model covering all 35 product areas. For each table the marker is:

- **[EXISTS]** — present today and broadly correct; keep as-is or with cosmetic change.
- **[EXTEND]** — present today but needs new columns / FKs / constraints.
- **[NEW]** — does not exist today; net-new table.
- **[CONFLICTING]** — present today but in a shape the target rejects; refactor or deprecate.

Each area uses the audit template (Existing → Status → Risk → Recommendation → Notes/open questions). Tables are listed with key columns, relationships, and enums. Everything is **RLS-aware**: every table is owner-or-admin by default, with explicit public-read carve-outs noted.

### Global conventions for the target
- **PK:** `id uuid default gen_random_uuid()` unless it mirrors another row's id (e.g. `provider_profiles.id == user_id`, an invariant enforced by `20260513130000`).
- **Timestamps:** `created_at timestamptz default now()`, `updated_at timestamptz` with the existing `update_updated_at_column` trigger where mutable.
- **Identity FK:** all user references → `public.users(id)` (note `profiles` was RENAMEd to `users`).
- **Enums:** the current build favors `text + CHECK` over PG enums for new tables (see `favorites.entity_type`, `children.gender`). The target **standardizes on real PG enums** for closed vocabularies where the value set is stable, and keeps `text + CHECK` only where values churn (e.g. master-list `status`). This is a deliberate hardening over today's mix. `user_roles.role` is migrated from `TEXT` to the existing `app_role` enum (currently a top risk).
- **Money:** integer ILS (agorot-free, whole shekels) per existing `provider_offerings.cost_min/max` and spec (`price int`), `currency text default 'ILS'`.
- **Geo:** `latitude/longitude numeric` validated by `validate_israel_coordinates()` (existing). PostGIS `geography(Point)` is an open question (see §35).

---

## AREA 1 — Users / Profiles / Roles

### `users` [EXISTS / EXTEND]
- **Existing:** `users` (renamed from `profiles`): `id` (PK, FK `auth.users`), `email`, `display_name`, `avatar_url`, `phone`, `is_active`. Populated by `handle_new_user` trigger (`on_auth_user_created`). Evidence: digest §Identity, `handle_new_user` is OAuth-aware and creates `users` + parent role + `parent_profiles`.
- **Extend (target):** add `suspended_until timestamptz null`, `banned_at timestamptz null` (FEATURES_SPEC §4.6 / M09), `chat_vendor_user_id text null` (M12), `locale text default 'he'`, `deleted_at timestamptz null` (soft-delete for GDPR-style account deletion in Settings §4.4).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Extend.
- **Notes:** `phone` is admin-only in all user-facing projections (FEATURES_SPEC §2.3). Enforce at the query/view layer, not just RLS, because authenticated mothers can read each other's `parent_profiles`.

### `user_roles` [EXTEND / CONFLICTING]
- **Existing:** `user_roles`: `id`, `user_id`, `role` **TEXT**, `UNIQUE(user_id, role)`. The `app_role` enum exists (`admin|moderator|user|provider|parent`) but the column is still `TEXT` — flagged as a top risk (two sources of truth for role; `useAuth.tsx` derives role from *both* profile-table existence *and* `user_roles`).
- **Target:** migrate `role` → `app_role` enum. Keep multi-role (a mother upgrades to provider — Airbnb model, Schema Philosophy). `useAuth` should read role from `user_roles` as the single source of truth; profile-table existence becomes a *consequence*, not a *signal*.
- **Status:** Conflicting (text vs enum + dual source). **Risk:** Medium. **Recommendation:** Refactor.
- **Open Q:** do we keep `moderator` distinct from `admin`, or collapse? Spec only ever mentions admin.

### `admin_profiles` [EXISTS]
- **Existing:** `admin_profiles`: `id`, `user_id` (1:1), `access_level`, `department`, `notes`. `update_admin_profiles_updated_at` trigger.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep.

### `mom_profiles` (today `parent_profiles`) [EXISTS / EXTEND]
- **Existing:** `parent_profiles`: `id` (= user id), `user_id`, `bio`, `city`, `neighborhood`. Legacy `children_ages[]` / `number_of_children` already DROPPED (M02). No `CREATE TABLE` in migrations (Lovable-era; only ALTERed) — see §35 rebuild risk.
- **Target:** keep table; the product term is "mom" but the physical name `parent_profiles` can stay to avoid a churny rename (or alias via a view `mom_profiles`). Add: `interests text[]` (Discover Mothers filter, §2.3), `region_key text` FK→`regions` (home-region prefill), `latitude/longitude numeric` (for distance-based mom-to-mom discovery — today only providers/meetups are geocoded). **Multiple-birth tag is derived** from focus-baby count, NOT stored (§2.6).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Extend.
- **Open Q:** is mom location needed at point precision (privacy) or only region/city? Spec §2.3 filters on location but never displays raw lat/lng for moms.

---

## AREA 2 — Onboarding progress + events

### `onboarding_progress` [NEW]
- **Existing:** nothing. Today onboarding completion is implicit (profile rows exist). `audit_action` enum *does* carry `start_provider_onboarding` / `complete_provider_onboarding` / `discard_provider_onboarding` (database.types.ts L1640-1664) but there is no progress table.
- **Target columns:** `id`, `user_id` (unique), `flow text` (`mom|provider`), `current_step text`, `completed_steps text[]`, `is_complete bool default false`, `started_at`, `completed_at`, `updated_at`.
- **Status:** Missing. **Risk:** Medium (onboarding resumption + funnel analytics depend on it). **Recommendation:** Create New.

### `onboarding_events` [NEW]
- **Existing:** nothing (only the three audit_action enum values above).
- **Target columns:** `id`, `user_id`, `flow text`, `step text`, `event text` (`step_viewed|step_completed|step_skipped|abandoned`), `metadata jsonb`, `created_at`. Append-only, feeds the professional-funnel analytics (§24).
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New.
- **Open Q:** keep onboarding events separate, or fold into the generic `funnel_events` (§24)? Recommendation: separate, because onboarding has a fixed step vocabulary while funnel events are open.

---

## AREA 3 — Children (birth_group, due_date, primary baby)

### `children` [EXISTS / EXTEND]
- **Existing:** `children`: `id`, `parent_id` FK→users, `gender` (`female|male`, **nullable** for legacy), `birth_date`, `name`, `is_focus_baby` bool. The single-focus partial unique index was **dropped** (`20260427110000`) to support twins/triplets (multiple focus babies allowed). Owner-only RLS.
- **Target — add the product fields not yet present:**
  - `birth_group uuid null` — groups siblings born together (twins/triplets share one `birth_group`); enables the multiple-birth tag and "find other moms of multiples" without recomputing.
  - `due_date date null` — for pregnant moms (a mom can onboard before birth; `birth_date` then null, `due_date` set).
  - `is_primary bool` (a.k.a. "primary baby") — distinct from `is_focus_baby`: the *single* baby the app centers the experience on. Today multiple focus babies are allowed but there is no "the one" pointer. **Open modeling tension** (see below).
  - `pregnancy_status text` enum (`expecting|born`) — derived-capable but worth storing for query simplicity.
- **Status:** Exists (core), Missing (birth_group / due_date / primary). **Risk:** Medium. **Recommendation:** Extend.
- **Open Q (important):** the spec (§2.1/§2.6) says "multiple focus babies" and the single-focus index was *deliberately removed*. The target now introduces a `is_primary` ("primary baby") concept. **Decision needed:** is "primary baby" the same as "focus baby" (then re-add a single-true constraint and drop the multiples allowance), or a separate pointer layered on top of multiple focus babies? Recommendation: keep `is_focus_baby` (set of maternity-leave babies, multiples allowed) AND add nullable `is_primary` with a partial-unique `(parent_id) WHERE is_primary` for the one baby that drives the home feed.

---

## AREA 4 — Professionals (provider core)

### `professionals` (today `provider_profiles`) [EXISTS / EXTEND]
- **Existing:** `provider_profiles`: `id` (PK, **invariant `id == user_id`**, `20260513130000`), `user_id`, `verification_status` (`draft|pending|verified|rejected`), `is_active`, `is_premium`, `bio`, `education`, `clinic_*`, `address_*`, `latitude`, `longitude`, `service_areas[]`, `languages[]`, `active_hours` (jsonb), `comes_to_home`, social URLs, `academic_title` (`dr|prof`) + `_status`/`_document_url`/`_submitted_at`, `subscription_tier_pref` (`free|pro_trial`), `pro_trial_started_at`/`_expires_at`, `pending_changes` (jsonb), `background_image`, `years_experience` (database.types.ts L336). No `CREATE TABLE` in migrations (Lovable-era). Public-read RLS (`USING true`).
- **Target:** keep as the provider hub. **Deprecate** these columns into dedicated tables/derivations:
  - `is_premium` → derived from `subscriptions.plan='pro'` (M11; drop in M14).
  - `verification_status` → derived "publicly visible" state (identity verified AND ≥1 approved cert; M03/M10; drop in M14).
  - `subscription_tier_pref` / `pro_trial_*` → move to `subscriptions` / `trials` (§21–22).
  - `education` (free text) → `education` table (§11).
- **Status:** Exists. **Risk:** Medium (overloaded table; trial/premium/verification state should not live here long-term). **Recommendation:** Extend now, slim later.
- **Note:** the DEAD `providers` table (Lovable mock: name/address/services[]) must be **dropped** — not part of target. **Recommendation: Deprecate.**

---

## AREA 5 — Profession taxonomy (domains / professions / services / specializations / requests)

This is the biggest **gap vs target**. Today there are flat master vocabularies but no hierarchy (domain → profession → service → specialization).

### `domains` [NEW]
- **Existing:** none. Today `professions`, `services` are flat.
- **Target columns:** `id`, `name_he` (unique), `slug`, `display_order`, `status` (`active|pending_approval|rejected`). Top of the taxonomy (e.g. "שינה", "הנקה", "התפתחות").
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New.

### `professions` [EXISTS / EXTEND]
- **Existing:** `professions`: `id`, `name_he` (unique), `status` (`active|pending_approval|rejected`), `requested_by_user_id`, `reviewed_by/_at`. Master-vocab pattern. RLS own-or-verified-or-admin.
- **Target:** add `domain_id uuid null` FK→`domains` to slot professions under domains.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Extend.

### `services` [EXISTS / EXTEND / CONFLICTING]
- **Existing:** TWO overlapping systems: (a) master `services` (`id`, `name_he`, `status`, …) used by `provider_services`; (b) LEGACY `service_types` + `provider_service_types` (Lovable-era, used by `useFavoriteServiceTypes`). The digest flags the overlap explicitly.
- **Target:** consolidate on master `services`; add `profession_id uuid null` FK→`professions` for hierarchy. **Deprecate** `service_types` / `provider_service_types` after migrating tags into `services` / `provider_services`.
- **Status:** Conflicting (two service systems). **Risk:** High (search filters read different tables). **Recommendation:** Refactor + Deprecate legacy.

### `specializations` [NEW]
- **Existing:** none as a table. `education_type` enum has a `specialization` value but that is education granularity, not a taxonomy node.
- **Target columns:** `id`, `name_he`, `service_id uuid null` FK→`services` (or `profession_id`), `status`, master-vocab columns. The finest taxonomy grain.
- **Status:** Missing. **Risk:** Low/Medium. **Recommendation:** Create New (confirm with product that this grain is needed for v1 filtering).

### `taxonomy_requests` (today inline `requested_by_user_id` + `status`) [EXISTS pattern]
- **Existing:** the "request a new entry" workflow is modeled *inline* on each master table (`status='pending_approval'`, `requested_by_user_id`) rather than a separate requests table. Works today for `professions/certifications/institutes/services/license_authorities`.
- **Target:** **keep the inline pattern** — it is simpler and already wired into the admin queue (`usePendingMasterEntries`). A separate `taxonomy_requests` table is NOT recommended; it would duplicate state.
- **Status:** Exists (as inline pattern). **Risk:** Low. **Recommendation:** Keep.
- **Open Q:** the target adds `domains` + `specializations` to the master family — they must follow the same inline `status/requested_by_user_id/reviewed_*` pattern and join the admin approval queue.

---

## AREA 6 — professional_professions (join)

### `provider_professions` (= professional_professions) [EXISTS]
- **Existing:** `provider_professions`: `provider_id`, `profession_id`, `license_number`, `license_authority_id` FK→`license_authorities`, `license_country`, `license_document_url`, `license_expires_at`, `verification_status`, `verified_by/_at`. Credentials-v2 (`20260511*`). Master-dependency triggers (`trg_enforce_*_claim_approval_dependencies`).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep. (This is the modern replacement for the dropped `provider_certifications`.)
- **Note:** `certification-docs` and license document storage bucket is **PUBLIC-read** — license/diploma URLs are world-readable. **Top HIGH/privacy risk** in the digest. Target: move credential docs to a **private bucket** with signed-URL access. (Storage, not a table, but model-adjacent and security-critical.)

---

## AREA 7 — Verification (levels / documents / reviews / badges)

The target splits **identity verification** (all users, Veriff) from **credential verification** (providers, admin manual). Today only credential verification exists (inline `verification_status` on credential tables); identity is a no-op (FEATURES_SPEC §1.1).

### `verifications` (identity) [NEW]
- **Existing:** none — identity gate is a no-op today.
- **Target columns:** `id`, `user_id`, `provider text` (`veriff|persona`, enum), `status` (`pending|approved|rejected|expired`), `provider_session_id`, `attempted_at`, `completed_at`, `raw_payload jsonb`. Latest row per user is canonical (index `(user_id, attempted_at desc)`). Service-role-only write (webhook). M10 (deferred — Veriff).
- **Status:** Missing. **Risk:** Medium (it is THE trust gate; deferred but structurally central). **Recommendation:** Create New.

### `verification_documents` [NEW]
- **Existing:** documents are scattered as URL columns (`license_document_url`, `diploma_url`, `academic_title_document_url`). No unified document table.
- **Target columns:** `id`, `owner_user_id`, `doc_type text` (`diploma|license|id_front|id_back|academic_title|other`), `storage_path`, `bucket`, `is_private bool default true`, `uploaded_at`, plus a polymorphic `linked_entity_type/linked_entity_id` (links to a credential row or a verification). Centralizing documents enables consistent private-bucket handling (fixes the world-readable-docs risk).
- **Status:** Missing (today inline URLs). **Risk:** Medium. **Recommendation:** Create New (or keep inline for v1 and only fix the bucket privacy — see Open Q).
- **Open Q:** is a unified `verification_documents` table worth it now, or just fix the bucket and keep inline URLs? The inline approach is wired everywhere; a central table is cleaner but is a migration. Recommendation: fix bucket privacy now; defer the central table.

### `verification_reviews` [NEW]
- **Existing:** review *actions* are captured in `audit_log` (`audit_action` ~65 values incl. cert approve/reject) + inline `reviewed_by/_at` on each credential row. No standalone review table.
- **Target columns:** `id`, `subject_type text` (`identity|profession|education|service|master_entry`), `subject_id`, `reviewer_id`, `decision text` (`approved|rejected`), `reason text`, `created_at`. Optional — `audit_log` already covers this; a dedicated table only helps if we need a queryable review history per subject.
- **Status:** Missing (covered by audit_log + inline). **Risk:** Low. **Recommendation:** Keep current (audit_log) unless product needs per-subject review threads.

### `badges` / `provider_badges` [NEW]
- **Existing:** none. "Verified" badge is derived (count + types of approved certs, §3.6).
- **Target:** the "verified" badge is **derived**, not stored. If we later add achievement badges ("founding member", "top rated"), model `badges` (catalog) + `provider_badges` (awards). For v1, **derive** — no table.
- **Status:** Missing (intentionally). **Risk:** Low. **Recommendation:** Keep derived; create only when non-derivable badges appear (e.g. founding member, §22).

---

## AREA 8 — Verification levels

### `verification_levels` [NEW — optional]
- **Existing:** none. Trust is binary-ish today (verification_status enum on provider).
- **Target:** the spec models trust as two independent boolean-ish streams (identity verified, ≥1 cert approved) that combine into "publicly visible." A formal `verification_levels` ladder (e.g. L0 unverified → L1 identity → L2 credentialed → L3 license-checked) is **not in the spec** but is a natural future axis.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Defer / Create New only if product wants tiered trust display. For v1, derive the two-stream state.

---

## AREA 9 — Education

### `provider_education_records` (= education) [EXISTS]
- **Existing:** `provider_education_records`: `provider_id`, `education_type` (enum: `bachelor|master|doctorate|certificate|specialization`), `certification_id` FK→`certifications`, `institute_id` FK→`institutes`, `year_started/_ended`, `diploma_url`, `license_number`, `license_expires_at`, `verification_status`. Credentials-v2. Master-dependency trigger `trg_enforce_education_master_deps`.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep.
- **Note:** `diploma_url` → private bucket (see Area 6 security note).

---

## AREA 10 — Experience

### `provider_experience` (= experience) [NEW]
- **Existing:** only `provider_profiles.years_experience` (a single int, database.types.ts L336). No structured work-history table.
- **Target columns:** `id`, `provider_id`, `title`, `organization_name text` (free text or FK→`organizations` §13), `description`, `start_date`, `end_date null`, `is_current bool`, `display_order`. Lets providers list roles/positions, not just a number.
- **Status:** Missing (only the scalar). **Risk:** Low. **Recommendation:** Create New (keep `years_experience` as a derived/cached convenience, or drop it).
- **Open Q:** is structured experience in v1 scope, or is `years_experience` enough? Spec §3.1 lists business info + certifications but not work history. Recommendation: defer the table; keep the scalar for v1.

---

## AREA 11 — Credentials (master vocabularies)

### `certifications`, `institutes`, `license_authorities` [EXISTS]
- **Existing:** master-vocab pattern (`id`, `name_he` unique, `status` `active|pending_approval|rejected`, `requested_by_user_id`, `reviewed_by/_at`). `certification_institutes` join (admin-curated, `source='admin'|'derived_from_claim'`, hard-delete cascade per "buildings vs blocks" model, M03). Dropped legacy: `provider_certifications` + `certification_institutes` predecessors.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep.
- **Note:** target adds `domains` + `specializations` to this family (§5) following the identical inline-approval pattern.

---

## AREA 12 — Services (provider ↔ service)

### `provider_services` [EXISTS]
- **Existing:** `provider_services`: `provider_id`, `service_id` FK→`services`, `verification_status`, `UNIQUE(provider_id, service_id)`. Trigger `trg_enforce_service_master_dep`.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep.
- **Note:** add a `is_primary bool` so a provider can mark one primary service (FEATURES_SPEC §3.2 "one can be marked primary"). Today no primary flag. **Recommendation: Extend** (add `is_primary`, partial-unique per provider).

### `provider_offerings` (= packages / `service_packages` in spec) [EXISTS / CONFLICTING naming]
- **Existing:** `provider_offerings`: `provider_id`, `title`, `description`, `cost_min/_max`, `duration_value/_unit`, `is_active`, `display_order`. Free-form, no admin. Public-read.
- **Target (spec §3.3 calls it `service_packages` with `name/description/price`):** the EXISTING `provider_offerings` already satisfies the Packages requirement. **Decision:** reuse `provider_offerings` as the Packages table (it is richer — has min/max cost + duration) rather than creating `service_packages`. Reconcile the spec naming.
- **Status:** Exists (under a different name than the spec's M04). **Risk:** Medium (spec/code naming mismatch could cause a duplicate table). **Recommendation:** Keep `provider_offerings`; **do NOT create `service_packages`** — map M04 onto it. Add `currency text default 'ILS'` if not present.
- **Open Q:** confirm with the team that M04 ("Service Packages") is satisfied by `provider_offerings` rather than a new table. This avoids two parallel "what the provider sells" tables.

---

## AREA 13 — Organizations (+ admins / members / services)

Entirely **NEW** — no organization concept exists today; providers are individuals.

### `organizations` [NEW]
- **Target columns:** `id`, `name_he`, `slug`, `bio`, `logo_url`, `address_*`, `latitude/longitude`, `contact_phone`, `contact_email`, `website`, `verification_status`, `created_by user_id`, `created_at`.
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New (confirm v1 scope — see Open Q).

### `organization_members` [NEW]
- **Target columns:** `id`, `organization_id`, `user_id` (or `provider_id`), `role text` (`owner|admin|member`), `status text` (`invited|active|removed`), `joined_at`. UNIQUE(org, user).

### `organization_admins` [NEW]
- **Note:** can be a subset of `organization_members` with `role IN ('owner','admin')` rather than a separate table. **Recommendation:** model as a role on `organization_members`, not a separate table.

### `organization_services` [NEW]
- **Target columns:** `id`, `organization_id`, `service_id` FK→`services`, `UNIQUE`. Mirrors `provider_services` at the org level.
- **Status (all org tables):** Missing. **Risk:** Medium. **Recommendation:** Create New.
- **Open Q:** organizations are NOT in `FEATURES_SPEC.md` (the spec is individual-provider centric). Is multi-provider/clinic support a v1 target or a later phase? If later, mark these four tables as **Phase-2** and do not build now. **This needs an explicit product decision before modeling further.**

---

## AREA 14 — Physical / business details

### `provider_profiles` business columns [EXISTS]
- **Existing:** `clinic_*`, `address_*`, `latitude/longitude`, `service_areas[]`, `comes_to_home`, `active_hours` (jsonb), social URLs all live on `provider_profiles`. Auto-geocode on edit (`updateProviderProfileAction` / `profile.ts`).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep.
- **Open Q:** `active_hours` is jsonb — fine for v1. If we need bookable availability later, normalize into a `provider_availability` table. Defer.

---

## AREA 15 — Plans / Subscriptions / Entitlements

All **NEW** (today only the `subscription_tier_pref` `free|pro_trial` hint + `is_premium` bool on `provider_profiles`). No real subscription/billing.

### `plans` [NEW]
- **Target columns:** `id`, `code text` (`free|pro`), `name`, `billing_cycle text` (`monthly|annual`), `price int` (99/990 ILS per §3.7), `currency`, `is_active`. A small catalog table so pricing is data, not code.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New.

### `subscriptions` [NEW]
- **Existing:** none (only `provider_profiles.is_premium` + tier hint).
- **Target columns:** `id`, `provider_id` FK→`provider_profiles`, `plan text` (`free|pro` enum), `status text` (`trial|active|past_due|cancelled|expired`), `billing_cycle text` (`monthly|annual`), `trial_ends_at`, `current_period_start/_end`, `payment_provider`, `payment_provider_subscription_id`, `created_at`. Service-role write (webhooks). M11 (deferred — payment provider TBD).
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New. Migrate `is_premium=true` → active PRO row; drop `is_premium` in M14.

### `subscription_invoices` [NEW]
- **Target columns:** `id`, `subscription_id`, `period_start/_end`, `base_amount`, `extra_meetup_count`, `extra_meetup_amount` (50 ILS/over-limit Pro Meetup, §3.4/§3.7), `total`, `status` (`open|paid|failed`), `payment_provider_invoice_id`, `created_at`.
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New.

### `entitlements` [NEW — recommended]
- **Existing:** none. Today feature-gating (gallery, pro-meetup creation) would be derived ad-hoc from `is_premium`.
- **Target columns:** `id`, `provider_id`, `feature text` (`photo_gallery|create_pro_meetup|pro_meetup_history|…`), `source text` (`plan|trial|founding_member|admin_override`), `granted_at`, `expires_at null`. An entitlements table decouples "what you can do" from "what plan you're on" and cleanly models trial + founding-member + admin-override grants (admin pro override exists today: `20260520100000_admin_pro_override.sql`).
- **Status:** Missing. **Risk:** Medium (without it, gating logic is scattered). **Recommendation:** Create New.
- **Open Q:** entitlements as a table vs. computed-on-read from `subscriptions`+`trials`+overrides. A table is auditable and cache-friendly but needs sync triggers. Recommendation: derive in v1 via a SQL function `has_entitlement(provider, feature)`; promote to a table if performance/audit demands.

---

## AREA 16 — Trials / Founding members

### `trials` [NEW]
- **Existing:** `provider_profiles.pro_trial_started_at/_expires_at` + `subscription_tier_pref='pro_trial'`. Inline, not a table.
- **Target columns:** `id`, `provider_id`, `kind text` (`pro_3mo`), `started_at`, `expires_at`, `started_reason text` (`first_cert_approved`), `converted_to_subscription_id null`, `status text` (`active|expired|converted`). Trial starts when first cert approved (§3.7/M11). Folding this into `subscriptions` (status='trial') is also valid.
- **Status:** Partial (inline columns). **Risk:** Low. **Recommendation:** Refactor into `subscriptions.status='trial'` (preferred — one lifecycle table) OR a dedicated `trials` table. Decide with M11.

### `founding_members` [NEW]
- **Existing:** none. Concept not in current schema or spec.
- **Target columns:** `id`, `provider_id` (or `user_id`), `granted_at`, `perks jsonb`, `cohort text`, `granted_by`. Likely drives a permanent entitlement (free PRO, badge).
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New only if the founding-member program is real for v1. **Open Q:** is this a launch program? Not in FEATURES_SPEC — needs product confirmation. If yes, it pairs with an `entitlements` row (`source='founding_member'`) and a derived badge.

---

## AREA 17 — Meetups (+ pro / QA types, participants, waitlist, attendance)

### `meetups` (today `baby_meetups`) [EXISTS / EXTEND]
- **Existing:** `baby_meetups`: `creator_id`, `title`, `description`, `latitude/longitude`, `location_address`, `meetup_date`, `meetup_time`. **No `meetup_type` column today** (flagged risk — favorites already encode `pro_meetup` but the meetup table can't distinguish). Public-read. `cleanup_old_meetups` function + cron.
- **Target — extend (M05):** add `meetup_type` enum (`free|pro|qa`), `price int null`, `payment_link text null`, `capacity int null` (host-defined; governance doc §06 wants `CHECK (capacity BETWEEN 2 AND 100)`), `status text` (`open|full|completed|cancelled`). Server validation: `meetup_type='pro'` ⇒ `price ∈ [25,120]` AND `payment_link` present (§3.4).
- **Status:** Exists (free), Missing (type/price/capacity/status). **Risk:** Medium. **Recommendation:** Extend.
- **Open Q:** the task brief mentions a **QA meetup type** (Q&A sessions) beyond free/pro. The spec only has free/pro. Confirm whether `qa` is a third `meetup_type` (e.g. a free or paid Q&A/AMA with a provider) and whether it has distinct columns. Model as a third enum value; reuse the same table.

### `meetup_participants` (today `meetup_attendees`) [EXISTS / EXTEND]
- **Existing:** `meetup_attendees`: `meetup_id`, `parent_id`, `status`. No `CREATE TABLE` in migrations (Lovable-era).
- **Target — extend (M05):** add `attendance_status` enum (`interested|pending_payment|paid_pending_provider_confirm|paid|free_added|cancelled|attended`) for the double-confirm payment lifecycle (§3.4). `UNIQUE(meetup_id, parent_id)`. Capacity enforced by trigger (joined count ≤ capacity, governance §06).
- **Status:** Exists (basic), Missing (attendance lifecycle). **Risk:** Medium. **Recommendation:** Extend.
- **Open Q (from MIGRATION_PLAN M05):** keep legacy `status` alongside new `attendance_status` during transition, or replace? Recommendation: replace once backfilled.

### `meetup_waitlist` [NEW]
- **Existing:** none. No waitlist concept today.
- **Target columns:** `id`, `meetup_id`, `user_id`, `position int`, `status text` (`waiting|promoted|expired`), `created_at`. Only needed once capacity caps exist and a full meetup should queue.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New only if product wants waitlists in v1. **Open Q:** not in spec — confirm. If deferred, capacity simply blocks join.

### `meetup_attendance` (check-in record) [NEW — optional]
- **Existing:** `attended` is a value in the proposed `attendance_status` enum, so attendance is a *state*, not a separate row.
- **Target:** **do NOT create a separate attendance table** — `attendance_status='attended'` on `meetup_participants` covers it. A separate table only helps if we record multiple check-in events. Recommendation: Keep on the participant row.

---

## AREA 18 — Meetup recommendations (relevance)

### `meetup_recommendations` [NEW]
- **Existing:** none. No recommendation/relevance engine today (governance §06 lists "matching/recommendation engine — later stage").
- **Target columns:** `id`, `user_id`, `meetup_id`, `relevance_score numeric`, `reason jsonb` (e.g. `{distance_km, baby_age_match, interest_overlap}`), `computed_at`, `surfaced_at null`, `dismissed_at null`. Populated by a scheduled job; read by the discover feed.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New as a **Phase-2** materialized recommendations cache. For v1, discovery is filter+distance (no scored relevance). **Open Q:** is scored relevance v1 or later? Spec §2.2 is filter-based; recommend deferring the table.

---

## AREA 19 — Mom connections

### `mom_connections` [CONFLICTING vs spec — modeled as favorites today]
- **Existing:** there is **no `mom_connections` table** by design. Per Schema Philosophy + FEATURES_SPEC §2.3: "the favorite IS the connection." A mom favoriting another mom (`favorites` with `entity_type='mother'`) is the one-way Instagram-style connection. M08 wires this.
- **Target decision:** **keep the favorites-as-connection model** — do NOT introduce a separate `mom_connections` table. One-way, no mutual gating, `entity_id` = favorited mom's `users.id`.
- **Status:** Exists (via `favorites`). **Risk:** Low. **Recommendation:** Keep (no new table).
- **Open Q:** if product later wants *mutual* friendship / accept-request semantics, THEN a `mom_connections` table with `(requester, addressee, status)` becomes necessary. v1 spec explicitly says one-way, so defer.

---

## AREA 20 — Chat (conversations / messages / blocks / reports)

The target **adopts an external chat vendor** (Sendbird/Stream/Twilio — TBD, M12). MomLee does NOT maintain its own message store long-term.

### `chat_messages` [EXISTS / DEPRECATE]
- **Existing:** `chat_messages`: `meetup_id`, `sender_id`, `message_text`. Supabase Realtime per-meetup channels. RLS = participants only (`is_meetup_participant`).
- **Target:** retain **read-only as historical archive**; retire once vendor group chat is live (M12/M14).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Deprecate (archive).

### `conversations` / `conversation_participants` / `messages` [NEW — but likely VENDOR-OWNED]
- **Existing:** none beyond `chat_messages`. Governance §06 sketched `conversations`+`messages`, but the spec **supersedes** that with an external vendor.
- **Target:** if the vendor owns messages, we store only a thin mapping: `chat_threads { id, vendor_thread_id, kind (dm|meetup|application), ref_id, created_at }` and `users.chat_vendor_user_id`. We do **NOT** replicate messages.
- **Status:** Missing (intentionally vendor-side). **Risk:** Medium (vendor not yet chosen). **Recommendation:** Create only the thin mapping table; **do not** build full conversations/messages tables.

### `chat_blocks` [NEW]
- **Existing:** none.
- **Target columns:** `id`, `blocker_id`, `blocked_id`, `created_at`, `UNIQUE(blocker_id, blocked_id)`. Governance §06 explicitly flags blocks/reports as **critical for a product for mothers**. Block may also be enforced vendor-side, but we keep the canonical record.
- **Status:** Missing. **Risk:** Medium (safety). **Recommendation:** Create New.

### chat reports → see Area 32 (unified `reports`).

---

## AREA 21 — Professional funnel events

### `funnel_events` [NEW]
- **Existing:** partial signal via `audit_log` (`audit_action` has provider-onboarding start/complete/discard) but not a true funnel store.
- **Target columns:** `id`, `user_id`, `funnel text` (`provider_signup|provider_onboarding|subscription`), `step text`, `event text`, `metadata jsonb`, `occurred_at`. Append-only. Powers conversion analytics for the professional acquisition funnel.
- **Status:** Missing (audit_log is a weak proxy). **Risk:** Low. **Recommendation:** Create New (or fold onboarding-specific events from §2 here; keep onboarding_progress separate as the state table).

---

## AREA 22 — Analytics aggregates

### `analytics_daily` / rollup tables [NEW]
- **Existing:** none. Only live views `provider_rating_stats`, `provider_favorite_stats` (per-provider aggregates, not time-series).
- **Target:** materialized rollups, e.g. `provider_metrics_daily { provider_id, date, profile_views, favorites_count, requests_received, applications_sent, pro_meetup_attendees }` and `platform_metrics_daily { date, new_users, new_providers, meetups_created, … }`. Populated by a daily cron.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New as needed (Phase-2 analytics). Keep the two existing stat views.
- **Open Q:** build in-DB rollups vs. ship raw events to an external analytics tool (the env shows Amplitude/HubSpot MCPs available). Recommendation: events table (§21/§23) + external analytics first; in-DB rollups only for provider-facing dashboard numbers.

---

## AREA 23 — Visit tracking

### `entity_visits` / `profile_views` [NEW]
- **Existing:** none. No view/visit tracking today.
- **Target columns:** `id`, `viewer_user_id null` (null = anonymous visitor), `entity_type text` (`provider|meetup|mother`), `entity_id`, `source text`, `created_at`. Append-only, high-volume → partition or TTL.
- **Status:** Missing. **Risk:** Low (but volume/privacy sensitive). **Recommendation:** Create New, but **respect the anti-stalking rule** (§2.3): a provider may see aggregate view counts; a mom must NEVER see "who viewed me." Store viewer id for analytics only; never expose it user-to-user.
- **Open Q:** GDPR/retention — set a TTL (e.g. 90 days) on raw visits; keep only aggregates long-term.

---

## AREA 24 — Notifications (events / preferences / tokens / templates / logs)

All **NEW** (M07). Today only transactional emails (React Email) + cron reminders exist; no in-app notification center.

### `notifications` (in-app feed) [NEW]
- **Target columns:** `id`, `user_id`, `type text`, `title`, `body`, `link text null`, `payload jsonb null`, `read_at timestamptz null`, `created_at`. Index `(user_id, read_at, created_at desc)`. Realtime channel `notifications-{user_id}`. Owner-only RLS.
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New.

### `notification_events` [NEW]
- **Target:** the *producer* log — what fired, for whom, dedup key. Columns: `id`, `event_type`, `target_user_id`, `dedup_key`, `payload jsonb`, `created_at`. Lets producers (triggers, cron, webhooks) be idempotent (mirrors existing `reminder_sent` idempotency pattern for meetup reminders).
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New (or reuse per-channel idempotency tables like `reminder_sent`).

### `notification_preferences` [NEW]
- **Target columns:** `id`, `user_id`, `category text` (`meetup_reminder|new_message|connection|new_provider|pro_meetup|billing|admin`), `channel_email bool`, `channel_push bool`, `channel_inapp bool`. Per §2.8/§4.4. Owner-only.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New.

### `push_tokens` (device tokens) [NEW]
- **Target columns:** `id`, `user_id`, `platform text` (`ios|android|web`), `expo_push_token text` (Expo Push, mobile), `device_id`, `created_at`, `last_seen_at`, `revoked_at null`. UNIQUE(token).
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New (needed for mobile push — the iOS-first priority).

### `notification_templates` [NEW]
- **Existing:** templates live in code today (React Email components), not DB.
- **Target columns:** `id`, `key`, `channel text` (`email|push|inapp`), `locale text`, `subject`, `body`, `variables jsonb`, `is_active`. DB templates let non-engineers edit copy. Optional — could keep in code for v1.
- **Status:** Missing (code-side today). **Risk:** Low. **Recommendation:** Defer (keep React Email in code for v1); create when marketing needs self-serve copy.

### `notification_logs` [NEW]
- **Target columns:** `id`, `user_id`, `notification_id null`, `channel text`, `provider text` (`resend|expo`), `status text` (`sent|delivered|failed|bounced`), `provider_message_id`, `error`, `created_at`. Deliverability audit.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New (light version: log sends + failures).

---

## AREA 25 — Reviews

The target **splits** reviews into provider-level and meetup-level (Schema Philosophy). Note governance §12 says recent commits **removed** the old `provider_ratings`; reconcile.

### `provider_ratings` [EXISTS / CONFLICTING]
- **Existing:** `provider_ratings`: `parent_id`, `provider_id`, `rating`, `review_text`. View `provider_rating_stats`. BUT governance §12 / digest flags ratings as being **removed** in recent commits ("Ratings/reviews … recent commits remove it; replaced by split").
- **Target:** keep a **general provider rating** table (`provider_ratings`) for overall quality. Reconcile the removal: if it was dropped, re-introduce per the split model.
- **Status:** Conflicting (exists in types but flagged removed). **Risk:** Medium (source-of-truth unclear). **Recommendation:** Confirm current state, then Keep/Re-create as the general rating table.
- **Open Q:** verify whether `provider_ratings` still exists in the live DB or was dropped. The digest lists it under tables AND notes it as being removed — resolve before building.

### `meetup_reviews` [NEW]
- **Existing:** none.
- **Target columns (M05):** `id`, `meetup_id` FK→`baby_meetups`, `attendee_id` FK→users, `location_rating int 1..5`, `professionality_rating int 1..5`, `overall_rating int 1..5`, `review_text null`, `created_at`. `UNIQUE(meetup_id, attendee_id)`. Only eligible attendees (`attendance_status ∈ paid|free_added|attended`) may post. Aggregates roll up to the provider's profile.
- **Status:** Missing. **Risk:** Low. **Recommendation:** Create New.

---

## AREA 26 — Q&A

### Forum (`forum_questions` / `forum_answers`) [EXISTS / DEPRECATE]
- **Existing:** `forum_questions` (`creator_id`, `title`, `content`, `provider_types[]`, `is_answered`), `forum_answers` (`question_id`, `provider_id`, `content`). Trigger `on_answer_created`, `mark_question_as_answered`. Public-read.
- **Target:** **REMOVED entirely** (FEATURES_SPEC Deprecated section — dropped to avoid "empty app" feel). Archive + drop in M14.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Deprecate (do not reintroduce).
- **Note:** the task brief's "Q&A" likely refers to **QA meetups** (§17 `meetup_type='qa'`), NOT the forum. The forum stays dead.

---

## AREA 27 — Moderation / Reports

### `reports` [NEW]
- **Existing:** none (governance §06 + §12 flag reports/blocks as critical-missing).
- **Target columns (M09):** `id`, `reporter_id` FK→users, `reported_entity_type text` (`user|meetup|message|provider`), `reported_entity_id uuid`, `reason_category text` (`inappropriate_content|harassment|fake_profile|scam|off_platform_pressure|other`), `description text null`, `status text` (`pending|reviewed|dismissed|actioned`), `action_taken text null` (`warned|suspended|banned`), `reviewed_by null`, `reviewed_at null`, `created_at`. Reporter inserts + reads own; admin reads/updates all.
- **Status:** Missing. **Risk:** Medium (safety-critical for a mothers' product). **Recommendation:** Create New.
- **Linked:** suspension/ban lives on `users.suspended_until` / `users.banned_at` (§1); auth middleware blocks login.

---

## AREA 28 — Admin

### `admin_profiles`, `admin_settings`, `regions` [EXISTS]
- **Existing:** `admin_profiles` (§1). `admin_settings`: `setting_key`, `setting_value` jsonb, `description` (public keys else admin RLS). `regions`: `region_key`, `region_name_he`, `keywords[]` (public-read; powers region filters + geocoding).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep all.
- **Note:** admin review queues (cert/master/identity/reports) are **query surfaces over existing tables**, not new tables. The only new admin-driven table is `reports` (§27).

---

## AREA 29 — Audit log

### `audit_log` [EXISTS]
- **Existing:** `audit_log`: `actor_id`, `action` (`audit_action` enum ~65 values), `target_table`, `target_id`, `target_name`, `reason`, `metadata jsonb`. Append-only. Provider-readable subset (`20260512120000`).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep / Extend the enum as new actions arise (reports, subscriptions, notifications). The enum is already the canonical action vocabulary — every new admin/lifecycle action adds a value here.

---

## AREA 30 — Favorites (cross-cutting; underpins Saved + mom connections)

### `favorites` [EXISTS]
- **Existing:** `favorites`: `user_id`, `entity_type` (`provider|mother|baby_meetup|pro_meetup`), `entity_id` (polymorphic, **trigger-enforced FK** via `favorites_validate_entity`), `notes`, `position`, `UNIQUE(user_id, entity_type, entity_id)`. Cleanup triggers per target table. Computed relationship `favorites_provider`. M01 done; M05/M08 wire the other entity types. Owner-only RLS (anti-stalking, §2.3).
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep. This single table serves Saved/Interested AND mom connections.
- **Note:** legacy `parent_favorite_providers` is DEPRECATED (drop in M14). `validate_entity` trigger must additionally enforce `meetup_type` ↔ `entity_type` consistency (M05).

---

## AREA 31 — Requests / Applications (reverse marketplace)

### `requests` [NEW]
- **Target columns (M06):** `id`, `user_id` FK→users, `title`, `description`, `service_type_id` FK→`services`, `latitude/longitude numeric`, `location_label`, `date_start date`, `date_end date null`, `budget int null`, `status text` (`open|matched|closed|cancelled`), `created_at`. Visible only to eligible providers (matching service + area).
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New.

### `applications` [NEW]
- **Target columns (M06):** `id`, `request_id` FK→`requests`, `provider_id` FK→`provider_profiles`, `message`, `proposed_package_id null` FK→`provider_offerings` (§12 decision), `proposed_price int null`, `status text` (`pending|accepted|rejected|withdrawn`), `created_at`. `UNIQUE(request_id, provider_id)` active. Accept → request `matched` + opens chat (M12).
- **Status:** Missing. **Risk:** Medium. **Recommendation:** Create New.
- **Open Q (M06):** "in area" matching = intersection with provider `service_areas[]` else fallback radius? Confirm.

---

## AREA 32 — Provider CRM (existing, out of v1 scope)

### `provider_customers` / `provider_reminder_templates` [EXISTS / DEPRECATE]
- **Existing:** `provider_customers` (`provider_id`, `parent_id`, `service_name`, `status active|completed`, reminder fields), `provider_reminder_templates` (`type email|whatsapp`, `content`). Owner+admin RLS. License-expiry cron uses provider license fields, not these.
- **Target:** **out of v1 scope** (FEATURES_SPEC Deprecated). Retain for now; archive + drop in M14. The Requests/Applications loop replaces in-app booking.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Deprecate.

---

## AREA 33 — Config / reference / cron idempotency [EXISTS]
- **Existing:** `regions` (§28), `admin_settings` (§28), `reminder_sent` (`meetup_id`, `reminder_window`, `sent_at` — cron idempotency for meetup reminders). Vercel crons: `meetup-reminders`, `license-expiry`.
- **Status:** Exists. **Risk:** Low. **Recommendation:** Keep. The notification producer log (§24) generalizes `reminder_sent`'s idempotency pattern.

---

## AREA 34 — Storage (model-adjacent, security-critical)
- **Existing:** 4 buckets, all **public-read**, owner-by-path-prefix write. **`certification-docs` is PUBLIC-read → diploma/license documents are world-readable** (digest TOP RISK #2, HIGH/privacy).
- **Target:** credential/ID/verification documents → **private bucket** + signed-URL access in server actions. Avatars / provider gallery / meetup images can stay public. This pairs with the optional `verification_documents` table (§7).
- **Status:** Conflicting (privacy leak). **Risk:** **High.** **Recommendation:** Refactor bucket privacy ASAP (independent of the data-model work).

---

## AREA 35 — Cross-cutting modeling decisions & open questions

### Enum strategy
- Migrate `user_roles.role` TEXT → `app_role` enum (top risk).
- New closed vocabularies (`attendance_status`, `meetup_type`, `report.reason_category`, `subscription.status`, `verification.status`) → real PG enums in the target (hardening over today's `text+CHECK` habit). Keep master-list `status` as `text+CHECK` (values are stable but the column doubles as a workflow state).

### Reproducible-rebuild risk (HIGH, infra)
- Lovable-era tables have **NO `CREATE TABLE`** in migrations (`provider_profiles`, `parent_profiles`, `meetup_attendees`, `parent_favorite_providers`, `provider_service_types`, `service_types`, `provider_ratings`). A rebuild-from-migrations **fails**. **Recommendation:** add baseline `CREATE TABLE IF NOT EXISTS` migrations capturing current shape BEFORE further evolution. Not a target-model table, but a precondition to safely shipping any of the above.

### Stale types
- `database.types.ts` lists dropped tables (`parent_favorite_providers`, `service_types`) and is ~90% accurate. **Regenerate** after each module (`supabase gen types … > packages/supabase/src/database.types.ts`).

### Geo
- Today `latitude/longitude numeric` + `validate_israel_coordinates()`. Distance/viewport works without PostGIS. **Open Q:** adopt PostGIS `geography(Point)` + GiST index for proper radius queries (mom-to-mom distance, request matching) or keep numeric + bounding-box math? Governance §06 assumed PostGIS; current build does not use it. Recommendation: introduce PostGIS when mom-discovery distance and request-radius matching land (M06/M08).

### Identity & role single-source-of-truth
- Resolve the dual role signal (profile-table existence vs `user_roles`) in `useAuth`. `user_roles` is canonical.

### Naming reconciliations (avoid duplicate tables)
1. **Packages:** reuse `provider_offerings`; do NOT create `service_packages` (§12).
2. **Mom connections:** use `favorites(entity_type='mother')`; do NOT create `mom_connections` (§19) unless mutual semantics are added.
3. **Mom profiles:** physical name `parent_profiles` stays; optionally expose a `mom_profiles` view.
4. **Provider:** `provider_profiles` is canonical; **drop the dead `providers` mock**.

### Scope decisions needed before modeling (product calls)
- **Organizations** (§13): v1 or Phase-2? Not in FEATURES_SPEC.
- **QA meetup type** (§17): is `qa` a real third type and what columns?
- **Founding members** (§16): real launch program?
- **Waitlist** (§17), **meetup_recommendations / relevance** (§18), **analytics rollups** (§22), **structured experience** (§10): all Phase-2 candidates not in the v1 spec.
- **provider_ratings** (§25): confirm whether it currently exists or was dropped — reconcile types vs commits.

---

## Appendix A — Target table roster (status at a glance)

| # | Target table | Today | Status |
|---|---|---|---|
| 1 | users | users | EXTEND (suspend/ban/chat_vendor/locale/deleted_at) |
| 2 | user_roles | user_roles (text) | REFACTOR (→ app_role enum) |
| 3 | admin_profiles | admin_profiles | EXISTS |
| 4 | mom_profiles | parent_profiles | EXTEND (interests, region, geo) |
| 5 | onboarding_progress | — | NEW |
| 6 | onboarding_events | — | NEW |
| 7 | children | children | EXTEND (birth_group, due_date, is_primary, pregnancy_status) |
| 8 | provider_profiles | provider_profiles | EXTEND/slim |
| 9 | domains | — | NEW |
| 10 | professions | professions | EXTEND (domain_id) |
| 11 | services | services | EXTEND (profession_id); legacy service_types DEPRECATE |
| 12 | specializations | — | NEW |
| 13 | provider_professions | provider_professions | EXISTS |
| 14 | verifications (identity) | — | NEW (M10) |
| 15 | verification_documents | inline URLs | NEW (optional; fix bucket first) |
| 16 | provider_education_records | provider_education_records | EXISTS |
| 17 | provider_experience | years_experience scalar | NEW (defer) |
| 18 | certifications/institutes/license_authorities | same | EXISTS |
| 19 | certification_institutes | same | EXISTS |
| 20 | provider_services | provider_services | EXTEND (is_primary) |
| 21 | provider_offerings (=packages) | provider_offerings | EXISTS (map M04) |
| 22 | organizations (+members/services) | — | NEW (scope TBD) |
| 23 | plans | — | NEW |
| 24 | subscriptions | is_premium/tier hint | NEW (M11) |
| 25 | subscription_invoices | — | NEW (M11) |
| 26 | entitlements | — | NEW (or derive) |
| 27 | trials | inline pro_trial_* | REFACTOR (into subscriptions) |
| 28 | founding_members | — | NEW (program TBD) |
| 29 | baby_meetups (=meetups) | baby_meetups | EXTEND (type/price/capacity/status) |
| 30 | meetup_attendees (=participants) | meetup_attendees | EXTEND (attendance_status) |
| 31 | meetup_waitlist | — | NEW (defer) |
| 32 | meetup_reviews | — | NEW (M05) |
| 33 | meetup_recommendations | — | NEW (Phase-2) |
| 34 | favorites (=Saved + mom connections) | favorites | EXISTS |
| 35 | chat_threads (vendor mapping) | chat_messages | NEW thin map; chat_messages DEPRECATE |
| 36 | chat_blocks | — | NEW (safety) |
| 37 | funnel_events | audit_log proxy | NEW |
| 38 | analytics rollups | rating/favorite views | NEW (Phase-2) |
| 39 | entity_visits | — | NEW (privacy-bound) |
| 40 | notifications | — | NEW (M07) |
| 41 | notification_events | reminder_sent | NEW |
| 42 | notification_preferences | — | NEW (M07) |
| 43 | push_tokens | — | NEW (mobile) |
| 44 | notification_templates | code (React Email) | DEFER |
| 45 | notification_logs | — | NEW (light) |
| 46 | provider_ratings | provider_ratings (?) | RECONCILE (exists vs dropped) |
| 47 | reports | — | NEW (M09, safety) |
| 48 | requests | — | NEW (M06) |
| 49 | applications | — | NEW (M06) |
| 50 | audit_log | audit_log | EXISTS |
| 51 | admin_settings / regions / reminder_sent | same | EXISTS |
| — | forum_questions/answers | same | DEPRECATE |
| — | provider_customers / reminder_templates | same | DEPRECATE |
| — | providers (dead mock) | providers | DROP |
| — | parent_favorite_providers | same | DROP (M14) |
| — | service_types / provider_service_types | same | DEPRECATE |

---

## Appendix B — Enum roster (target)

Existing keep/extend: `app_role` (admin|moderator|user|provider|parent — now used by `user_roles`), `education_type` (bachelor|master|doctorate|certificate|specialization), `audit_action` (~65, extend per new actions). Existing to deprecate: `customer_status`, `reminder_type` (with provider CRM removal).

New target enums: `meetup_type` (free|pro|qa?), `attendance_status` (interested|pending_payment|paid_pending_provider_confirm|paid|free_added|cancelled|attended), `meetup_status` (open|full|completed|cancelled), `subscription_status` (trial|active|past_due|cancelled|expired), `billing_cycle` (monthly|annual), `verification_provider` (veriff|persona), `verification_status_identity` (pending|approved|rejected|expired), `report_reason` (inappropriate_content|harassment|fake_profile|scam|off_platform_pressure|other), `report_status` (pending|reviewed|dismissed|actioned), `report_action` (warned|suspended|banned), `request_status` (open|matched|closed|cancelled), `application_status` (pending|accepted|rejected|withdrawn). Master-list `status` stays `text+CHECK` (active|pending_approval|rejected).
