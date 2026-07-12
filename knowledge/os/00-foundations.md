# Momlee OS — Foundations (Working Method)

> The "book": how product intent in Notion becomes unambiguous, buildable specs.
> Source of truth for the method. Skills enforce it; the plugin ships it to Sivan.
> Status: **draft, built DB-by-DB.** Decisions locked here are marked LOCKED.

## The problem this solves

Notion was being asked to do two opposite jobs at once: (1) be the **human
authoring tool** (fast, easy to fill) and (2) be the **complete, unambiguous
machine spec** Claude consumes. Pushing job #2 into Notion produced 30–50 field
tables that are impossible to maintain. The fix is to split the jobs across
layers.

## Principle 1 — Source of truth is split by layer (LOCKED)

- **Notion = product *intent* and decisions.** Human-authored, light. What/why,
  rules, decisions, identity, access. Small per item.
- **Code + Figma = technical *reality*.** Components, tokens, DB schema, events,
  APIs. This already exists — it is **never duplicated into Notion**.
- **The YAML = a compiled *artifact*, not a source.** Claude assembles it from
  Notion **+** code/Figma; Maor & Sivan review it. **Unambiguity is produced at
  compile + review time — it is never hand-authored.**

So "who is the source of truth?" → both, for different layers, reconciled by the
compiled YAML.

## Principle 2 — Three DB types (LOCKED)

- **① Intent DB** — human authors a small core; the rest is inherited / derived.
  (Epics, Entities, User Types, Roles, Eligibility, Permissions, Product Rules,
  Decision Log, Features.)
- **② Hub** — the **Story**. Small author core **+** wiring Claude *proposes and
  the human approves* (Permissions, Roles, Events, Components, Screens…).
- **③ Reality-mirror registry** — author core ≈ 0; **synced from code/Figma**,
  not hand-authored. (Components, Design Screens, Design Tokens, Database Tables,
  Schema Registry, APIs, Events.)

Consequence: roughly half the DBs are **not hand-maintained at all** — they
mirror code/Figma. Most of the old "50-field" pain lived there.

## Principle 3 — Author-core (LOCKED)

For every DB, define the **Author core**: the minimal fields a human must type to
create a valid row. Every other field is one of:

- **Inherited / back-relation** — attaches itself through a relation (e.g. an
  Epic's `Related Stories` fills when a Story picks its Feature). Never typed.
- **Claude proposes → human approves** (bucket ②) — Claude reads the Acceptance
  Criteria + Figma + code + registries and proposes the wiring.
- **Derived / workflow / synced** — Key derived from Name; Status is workflow;
  ③ fields synced from code.

The Author-core table per DB *is* the contract: it tells Maor how light Notion
stays, and tells Claude exactly what to compile/sync.

## The hierarchy (LOCKED)

**Epic → Feature → Story → Task.** ("Epic" replaces the old "Module" — there is
no "Module" anywhere: use `Epic`, `Relation → Epics`, `epic.…` keys.)

- **Epic** = a top-level product area / system domain.
- **Feature** = a product capability inside an Epic.
- **Story** = a specific behavior inside a Feature (the build unit → YAML).
- **Task** = an implementation step of a Story.

## Epic decomposition policy (LOCKED 2026-06-28)

Epics are decomposed on **one axis only: capability domain** — a coherent area of
*what the product does* (the conventional agile sense of an Epic). **Never
decompose Epics by actor / user type.** Test for a valid Epic: "is this a coherent
domain of capability?" If the answer names a user type, it is not an Epic.

The **actor** dimension (Mom / Professional / Organization) is modeled on four
other axes, never as an Epic:

- **User Type** (identity): `mom`, `professional`, `admin`.
- **Entity** (tenant + data objects): `organization`, `organization_member`,
  `provider_profile`, `mom_profile`, `subscription`.
- **Role** (capability within a tenant): `organizations.owner` / `admin` /
  `member`, …
- **Eligibility** (conditions): verified, subscribed, …

**Multi-tenant frame (locked):** Organization = the tenant. Every Professional
belongs to an Organization (a solo Pro = a *Solo Organization* of one).
Subscription sits on the Organization. Team membership = `organization_members`
(M:N) governed by Org Roles. So "add a team member" is an **Organization & Team**
capability, not a "Professionals" one. Organization is an **Entity**, not a User
Type.

Rules:

- **Actor-specific flows = Features inside the relevant capability Epic.**
  Onboarding → `Mom Onboarding` / `Pro Onboarding` / `Org Onboarding` Features;
  shared fields (name, phone, DOB) live in shared Stories/Components, divergent
  steps in Feature-specific Stories.
- **Cross-actor shared capability = one Epic.** Meetups is one Epic; mom/pro/org
  differences = `meetup_type` + Eligibility + variant Stories, never separate Epics.
- **The two former actor-Epics are refocused as capability-Epics** (renamed, not
  deleted): `Professionals` → **Provider Profile & Services** (provider profile,
  services catalog, leads, dashboard); `Organizations` → **Organization & Team**
  (org entity, members, Org Roles, invitations, settings). Onboarding /
  subscription / pro-meetups do **not** live here — each goes to its own
  capability Epic.
- **MVP:** Mom is the primary actor; all Provider/Org capability is
  `Scope = Fast-follow`; pro-meetups are admin-curated. Scope changes the timing,
  never the structure.
- **Variation within a capability = data + rules, never structure (LOCKED).**
  The many shapes a capability takes are modeled as data + access rules inside the
  one Epic, never as new Epics and never as a Feature-per-variant. Four layers:
  - **Behavior axis** — a `*_type` field for *how* it behaves (e.g.
    `conversation.conversation_type` = `direct` / `group` / `support`).
  - **Business-context axis** — `context_type` + `context_id` for *which world it
    belongs to* (e.g. `meetup` / `provider_profile` / `organization` /
    `support_ticket` / `none`). This is a **separate axis** from behavior: a meetup
    group chat and a (future) community group chat share `conversation_type=group`
    but differ by `context_type`.
  - **Access matrix** — Permissions + Eligibility decide *who may do what*
    (e.g. `communication.conversation.initiate`); Product Rules carry the business
    logic (e.g. "a professional cannot initiate to a mom unless she contacted
    first" — ties to D4, lead = contact-intent). A who-can-do-what matrix is the
    access layer, not structure.
  - **Genuinely different flows** (e.g. per-actor onboarding) = separate
    **Features** inside the one Epic, sharing Stories/Components where identical.

  **The meta-rule: one `type` field never carries two axes** (same lesson as
  Scope vs Status). If a value mixes "how it behaves" with "what world it is in,"
  split it. Concretely: `pro` vs `free` is **not** a context — it is the meetup's
  own `meetup_type`; `context_type` stays `meetup`. Re-conflating axes into one
  field is exactly the trap this principle prevents.

Current Epic set. Capability Epics: App Shell & Navigation · Browse & Discovery ·
Auth & Access · Onboarding · Profile & Settings · Meetups · Provider Profile &
Services · Organization & Team · Subscriptions · Communication · Notifications ·
Trust & Safety · Business Insights · Admin. Plus one enabler Epic: Platform /
Infrastructure.
("Communication" = in-app conversations/messaging, distinct from
Notifications, which is outbound system delivery.)

**Analytics split (LOCKED 2026-06-29).** The former "Analytics" Epic was two
things under one name; they are split. (1) **Business Insights** = a capability
Epic - business analytics/insights dashboards for Professionals and
Organizations (leads, profile/service views, trends); Post-MVP, bordering
Backlog. (2) **Product instrumentation** (PostHog telemetry) is NOT a capability
and NOT this Epic - it is a cross-cutting gate: the Events registry holds the
taxonomy, every Story links the Events it emits (a DoD gate), and the setup work
lives in the **Platform / Infrastructure** enabler Epic. Platform /
Infrastructure is the home for cross-cutting technical foundations (Features:
Product Analytics, Error Monitoring, Logging, Performance Monitoring, Feature
Flags, Observability Dashboard); it passes the Epic test as a coherent technical
domain and is team-facing, not an actor (precedent: Notifications/Admin are also
non-mom-facing). Error tracking provider = Sentry (under evaluation, not final).

**Epic boundary rules (LOCKED 2026-06-30).** To prevent capabilities drifting
between Epics:
1. **One capability = one Owning Epic.** A capability is owned by the domain it
   operates on - not by the actor performing it, and not by the screen where it
   appears.
2. **Surface is not ownership.** Surface Epics (App Shell, Admin) may render or
   operate capabilities from other domains, but do not own those capabilities.
3. **Actor does not equal ownership.** An admin reviewing reports or verification
   items is operating a domain capability through an admin surface; the domain
   Epic (e.g. Trust & Safety) still owns the lifecycle.
4. **Lifecycle boundary.** Onboarding owns only first-time activation/setup;
   ongoing management of the same information belongs to a separate domain Epic
   (e.g. Profile & Settings).

**Profile & Settings (Epic, MVP, refined 2026-06-30).** Two layers, never "Mom
only". **(1) Profile = domain-owned.** This Epic owns the *mom-facing* profile:
completion after onboarding, editing children, profile photo, bio,
address/location after onboarding, and profile preferences. Mom is the primary
MVP user. **(2) Settings = cross-actor account layer.** A shared account/settings
surface (language, notifications, privacy preferences, settings navigation and
entry points) that Professionals and Organizations may also use later (Post-MVP).
**Boundaries:** (a) the **professional profile is NOT here** - provider profile,
services, business info, credentials, verification documents, service areas and
availability belong to **Provider Profile & Services** (a separate domain:
`provider_profiles` vs `parent_profiles`); (b) authentication/security (change
password, change email, account recovery, connected login providers, session
management) stays in **Auth & Access** - Profile & Settings only provides the
surface / nav entry point to it. This is boundary rule 1 applied: profile is
owned by its domain, not by the screen or the shared settings layer.

**Provider Profile & Services (Epic, Post-MVP; key `provider_profile`).**
The refocused former `Professionals` actor-Epic (one page — a 2026-06-30 stub
duplicate was merged back into it on 2026-07-12 after the external audit caught
it; dedup lesson: scan the live DB, not the book, before creating). It owns the
professional domain and reserves its boundary so provider-profile work is not
misplaced under Profile & Settings, Admin, or Onboarding. Reserved scope:
`provider_profiles`, professional services, business information,
credentials/certifications, verification documents, availability / service
areas, leads/dashboard, and provider-facing profile management. Admin *actions*
on professionals (verification approval etc.) remain a Trust & Safety capability
via the Admin console (boundary rule 3); account-level auth stays in Auth &
Access.

**Browse & Discovery (Epic, MVP, confirmed 2026-06-30; was "Home / Discover").**
The cross-domain browse/discovery layer - how a user finds and browses content.
App Shell does NOT own Home content; this Epic does. It owns: the Home screen, the
meetup index/list, a provider directory (Post-MVP), cross-domain search, recommended/
personalized sections, ranking/feed composition, and discovery empty states.
**Boundary vs the entity domains (rule 1):** Meetups owns the meetup entity,
creation, the detail page and registration; Provider Profile & Services owns the
professional profile/catalog. Browse & Discovery only composes *how they are
browsed/surfaced* - "how you find/browse it" = here; "what the entity is and what
you do with it" = its domain. **MVP location scope (per Sivan):** proximity-based
ranking and a "near you" section are **Post-MVP**, pending a single-neighborhood
experiment. MVP discovery = a simple meetup index scoped to the experiment
neighborhood, with **no distance sorting**. Location is still captured at onboarding
(`default_location_radius_meters = 500`) but is **not used for ranking in MVP** -
captured now, used later.

**App Shell & Navigation (locked 2026-06-29; refined 2026-06-30)** = the global
frame ONLY: the `bottom_nav_bar` and tab routing, navigation structure, route
guards, global layout/states (global empty / coming-soon), and **routing to the
default post-onboarding destination**. It does **NOT** own the Home/landing
*content* - that belongs to the **Browse & Discovery** Epic (boundary rule 2). It is
the home for the navigation frame that recurs on *every* screen; it passes the
Epic test and is not an actor.

This Epic is the canonical worked example of the layer-split + the variation
principle. A globally-recurring component is **not** an exception to the model —
it decomposes:

- **The artifact = one Component** (③ reality-mirror): `bottom_nav_bar`, synced
  from Figma/code, referenced by every Screen. You never write a Story for "the
  nav appears on screen X" — that is just the Component attached to the Screen.
- **The behaviors = Stories**: tab navigation, "route to the default destination
  after onboarding" (App Shell *routes*; the Home *content* is owned by Home /
  Discover), active-tab state, and "nav hides / shows coming-soon for sections not
  yet live".
- **Per-actor variation = data + access, never a new component**: Mom vs Pro vs
  Org see different tab sets from the **same** `bottom_nav_bar`, driven by
  Role / Eligibility — not three components and not a Story-per-actor.
- **MVP "section exists but isn't ready yet" = Scope + a Product Rule**: a tab
  whose Feature is `Scope = Post-MVP` is hidden or shows a coming-soon state via
  one Product Rule ("tabs whose feature is not yet live are hidden / coming-soon")
  + one Story. When the Feature reaches MVP, the tab lights up — a timing change,
  never a structural one.

## Naming & Key conventions (LOCKED)

All technical keys: English, lowercase, `snake_case` within a part, dots between
parts. Names (Title) are human-readable; Keys are stable and machine-facing.

| Concept | Key format | Examples |
|---|---|---|
| Epic | `epic` (area, snake_case) | `meetups`, `auth_access`, `organizations` |
| Feature | `epic.feature` | `meetups.details`, `meetups.registration` |
| Story | `epic.feature.story` | `meetups.details.mom_views_full_details` |
| Entity | `entity` (singular) | `meetup`, `mom_profile` (DB table = plural `meetups`) |
| User Type | bare identity | `mom`, `professional`, `organization`, `admin` |
| Role | `domain.role` | `users.mom`, `organizations.owner`, `admin.reviewer`, `system.automation` |
| Eligibility | `actor.condition` | `mom.is_identity_verified`, `pro.has_active_subscription` |
| Permission | `epic.entity.action` | `meetups.meetup.view_full_details`, `organizations.members.invite` |
| Automation | `epic.trigger.outcome` | `onboarding.location_skipped.send_reminder` |
| Event | `entity.action_past_tense` | `meetup.created`, `verification.submitted` |

Open naming note: Role key prefix is the domain (`users.`) while Eligibility
prefix is the actor (`mom.`). Intentional; revisit only if it causes confusion.

### Name vs Key, completeness, dedup (LOCKED 2026-06-29)

**Name vs Key (every DB).** The title field (`Name` / `Data Type` / `Channel` …)
is **human-readable prose** ("Mom is identity verified", "Create a meetup", "View
full meetup details"). The `Key` is the stable machine identifier in its DB's
format (table above). Never type the key format into the Name, and never type prose
into the Key. (This fixed a real drift where Eligibility and Permission names had
been typed as keys.)

**Record completeness (the quality bar).** Every record must have ALL its
author-core fields filled - no blanks - and a consistent format across rows of the
same DB. Bilingual `he / en` everywhere, with ONE exception: Technology Stack
`Purpose` is English-only. A genuinely-unknown value is captured as an Open Question
or marked "pending Sivan approval" - never left blank and never invented.

**Dedup + deprecation.** One concept = one row. Always discover existing rows
before creating (lookup/registry DBs are pre-seeded). The Notion API **cannot delete
or archive a single database row** (only a whole data source). So to retire a
duplicate: (1) migrate its relations/links/data to the surviving canonical row
FIRST, (2) rename it `"<name> (duplicate - delete)"`, (3) set a Deprecated status if
the DB has one, (4) flag it for Maor to delete in the Notion UI.

**Verify Source-of-Truth against the live DB.** When a field points at real data
(e.g. Eligibility `Source of Truth`), confirm the `table.field` exists in the live
Supabase DB (read-only `supabase-momlee`). If it does not exist yet, write
"currently Notion, requires Sivan approval before implementing" with the proposed
column - never assert a column that is not there (the live `database.types.ts` is
stale).

## Cross-cutting fields: Scope & Status (LOCKED)

Two **orthogonal** fields appear on most authored (①/②) DBs:

- **`Scope`** = release bucket: `MVP` / `Fast-follow` / `Post-MVP` / `Backlog` /
  `Cut`. `Fast-follow` = ships within days of the initial launch (e.g. the
  professional / organization / subscription cluster — built in parallel now,
  launched right after MVP).
- **`Status`** = lifecycle: `Draft` / `Approved` / `In Use` / `Needs Review` /
  `Deprecated` (exact set varies per DB).

Independent: a row can be `Scope: Fast-follow` **and** `Status: Draft`. Never
encode scope inside Status.

**Where the two axes live (LOCKED 2026-06-28).** Scope & Status are the two
*planning axes* and they travel together: a level carries them **as manual
fields only where a human actually makes that call**; below that they are
inherited or derived.

| Level | Scope | Status |
|---|---|---|
| **Epic** | manual, light (editorial headline, e.g. `organizations` = Fast-follow); rollup later | manual, light; rollup later |
| **Feature** | **manual** — the primary scoping unit (MVP vs Post-MVP is decided here) | **manual** |
| **Story** | **inherited** from its Feature (override only for a genuine exception) | manual **work** status (it already carries Dev Status / Design Status) |
| **Task** | none — inherits from the Story | manual **work** status |

So Feature is the home of both axes; Story inherits Scope and tracks its own
build status; Epic keeps a light editorial pair now (rollups are a later, smarter
upgrade — see the Status-rollup note); Task tracks status only.

**Story Scope override (impl, LOCKED 2026-06-29).** Story `Scope` is a read-only
rollup from the Feature; the genuine-exception override the level-map allows is a
separate manual `Scope Override` select on the Story (empty = inherit). Effective
scope = `Scope Override` if set, else the inherited `Scope`. Use sparingly - e.g.
a Post-MVP story inside an MVP feature (session replay, React Native crashes
inside the otherwise-MVP Platform features).

## The workflow (target)

1. Maor + Sivan build a **Story** in Notion with its small author core (above
   all: a precise **Acceptance Criteria**). The Story's authoring surface is a
   7-section bilingual **row template** in the page body (not the generic 6 DB
   tabs): `Summary · User Story · Preconditions · Main Flow · Success State ·
   Error/Blocked States · Acceptance Criteria` (locked 2026-06-29). The 6 DB
   tabs document concept/registry DBs; authoring DBs like Story and structured
   inventories like Privacy Notes live in their row template + fields instead.
2. Claude proposes the Story's wiring (bucket ②); they approve.
3. Claude compiles the Story + code/Figma reality into an **unambiguous YAML**.
4. Maor + Sivan review the YAML.
5. Claude takes the YAML to development (web / Vercel).

Mechanical gates still apply (reuse-before-create, tokens-only from Figma,
RTL, security/privacy). See the rest of the plugin's skills + knowledge.
