# Momlee OS — Task & Automation

> Task = the execution unit under a Story. Automation = a behavior **registry +
> spec**, NOT an automation builder. Keys per `00-foundations.md`. Status: LOCKED.

---

## Task  ·  execution unit under a Story (type ②, mostly Claude-generated)

Purpose: one implementation step of a Story.

- **Author core (when drafted by hand) — ~3:** `Name`, `Story` (parent),
  `Task Type` (design / backend / frontend / migration / test).
- **Claude generates:** most Tasks are produced when Claude splits a Story; Maor
  & Sivan may also draft tasks and Claude polishes.
- **Tracking fields:** `Status` (Not Started / In Progress / In Review / Done /
  Blocked), `Owner` (person), `Priority` (Low / Medium / High / Critical),
  `Description`. Scope is inherited from the Story (not a field here).
- Cleanup: the current Tasks DB has duplicate relations `Story` + `Stories` →
  collapse to ONE relation to Stories.

---

## Automation  ·  `epic.trigger.outcome` — Registry + Spec (NOT a builder)

Purpose: document a system behavior (trigger → logic → outcome). It is a
**registry entry + spec**, not a 30-field form. It follows the **same pattern as
a Story**: author light summaries → Claude resolves them into canonical relations
→ you approve.

**① Author core (~5, free-text — fast, works before registries are full):**
- `Name`
- `Serves Story` (anchor; Epic/Feature inherit through it)
- `Trigger Summary` — plain language: when does it fire?
- `Logic Summary` — plain language: conditions / timing / logic.
- `Outcome Summary` — plain language: what happens.
- `Timing` — the concrete delay/schedule value (`Immediate` / `3 days after event`
  / `daily 0900`); `Trigger Type` is the category, `Timing` is the value, and the
  real cron lives in code.
- `Status`
- (`Key` derived `epic.trigger.outcome`.)

**② Claude resolves → you approve — the canonical wiring:**
- `Trigger Event` (→ Events registry), `Automation Action`,
  `Conditions / Eligibility`, `Communication Channels`, `Message Templates`,
  `Events Created`.
- Claude **maps the summaries to canonical registry rows**. If no canonical
  match exists (e.g. the trigger event isn't in the registry yet), Claude
  **STOPS and asks "create a new event/action?"** — it never invents silently.
  This is where the unambiguous spec is produced.

**③ Set when built / synced:**
- `Implementation Location` (Backend / Supabase / Make / OneSignal / Resend …),
  `Execution Reference` (path/URL), `Privacy Notes` (from data inventory).

Why this shape: light to author (5 summaries + an anchor), yet unambiguous at
compile (relations resolved + approved). One mental model across the whole OS —
Automation behaves exactly like a Story.

**Field model (LOCKED 2026-06-29, after the efficiency pass — DONE in live Notion).**
The Automations DB is now lean and complete:
- **`Feature` = rollup from `Related Stories`** (the Story carries Feature) — never a
  manual field. `Related Stories` is the single anchor. **`Epic` stays manual-light**
  (rollup-through-rollup is unreliable in Notion).
- **Dropped as redundant:** `Condition Notes` (overlaps `Logic Summary` + the
  `Conditions` → Eligibility relation) and `Target Roles` (audience = `Target User
  Types`; authorization is a Permissions concern, not an automation's).
- **Renamed** `Events` → `Trigger Event` (vs `Events Created`).
- **`Timing`** added: `Trigger Type` = the timing *category* (Event-based / Delayed
  After Event / Scheduled / Recurring / Webhook …); `Timing` = the concrete *value*;
  the real cron/scheduler config lives in code.
- `Events Created` / `Privacy Notes` are **legitimately empty** for many message
  automations — fill only where the automation truly emits a canonical event or
  touches PII (reminders → email/phone; a status webhook → identity data).
- The `Conditions` relation reuses **Eligibility** rows as the automation's gating
  conditions (e.g. `mom.has_completed_onboarding` for the incomplete-profile
  reminder); the inverse/exception nuance goes in `Logic Summary`.
