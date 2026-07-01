# Momlee OS — Behavior & Messaging

> The Behavior section DBs. Keys per `00-foundations.md`. Status: LOCKED.
> (Automations live in `30-task-automation.md`.)

---

## Events  ·  `entity.action_past_tense` (`meetup.created`)  — ③ mirror from code

Purpose: a canonical thing that happened in the system. **Events and "Analytics
Events" are MERGED into one registry** — a single `Events` DB with an
`Event Kind` to tell them apart. (Two tables for the same concept = drift.)

- **Author core ≈ minimal:** `Name`, `Key` (`entity.action_past_tense`),
  `Entity`, `Event Kind` (Domain / Analytics / System), `Description`. Plus
  `Source` (code registry), `Sync Status`, `Status`.
- **Source of truth:** code (the analytics/event registry files). Claude syncs
  the mirror; `Sync Status` mandatory.
- **Relations:** `Triggers Automations` + `Created By Automations` (two roles —
  matches Automation's `Trigger Event` / `Events Created`), `Used in Stories`,
  `Used in Features`, **`Privacy Notes`** (→ Data Inventory — enforces *no PII in
  analytics event payloads*). Optional: `Resulting States` (event → entity state
  change; tie to a state-machine DB later — Post-MVP).
- The compile step **reads** Events so Claude links a real event — never invents
  one. No canonical event for a needed trigger → STOP and ask (create it).
- Buildout mined from the old Notion Events DB: `Event Kind`, the two automation
  roles, and the `Privacy Notes` link were the smart parts worth keeping.

## Communication Channels  ·  transport-only lookup DB (LOCKED 2026-06-29)

Purpose: the ONE canonical list of **transport channels** — how a message is
delivered. **Channel = transport only.** It is NOT the message category and NOT a
delivery-surface variant.

- **Canonical set (5):** `in_app`, `push`, `email`, `sms`, `whatsapp` — plus one
  **sentinel** `system` (explicitly internal / no user-facing delivery, for
  server-only automations). Nothing else is a channel.
- **Fields:** `Channel` (prose name), `Key`, `Description` (bilingual), and the
  channel **capabilities**: `Requires Template`, `Supports Scheduling`,
  `Supports User Opt-out`, `Delivery Surface`, `Default Technology` (→ Tech Stack),
  `Privacy Notes` (the PII it carries — sms/whatsapp → phone, email → email,
  push → device token), `Message Categories` (→ which categories are allowed on it),
  `Automations`, `Status`.
- **Capability vs policy (the rule):** the channel states what is technically
  possible (sms CAN be scheduled, CAN be opted out of). The *policy* (auth SMS must
  NOT be opt-out-able; marketing requires consent) lives on the **Message Category**,
  not the channel. So transactional vs marketing vs authentication are **categories,
  never channels**.
- **Delivery surface is a field, not a row:** in-app notification feed vs modal vs
  banner are one `In-App` channel with a Delivery Surface description — not separate
  channel rows.

## Message Categories  ·  the message-policy DB (LOCKED 2026-06-29)

Purpose: the behavioral/legal **class** of a message — carries the consent policy
(transactional vs marketing) and which channels may carry it.

- **Fields:** `Name` (prose), `Key` (snake_case), `Consent Class`
  (Transactional = always sent / Marketing = requires consent + allows opt-out),
  `Description` (bilingual), `Communication Channels` (→ allowed channels),
  `Templates`, `Status`.
- **Seeded set (2026-06-29):** Authentication · Account & Security · Verification ·
  Meetup Activity · Meetup Reminders · Messages & Chat (Transactional) · Marketing &
  Promotions · Product Updates (Marketing).
- A Template belongs to one Channel + one Message Category: the **category** decides
  consent / opt-out, the **channel** decides transport.

## Message / Notification Templates  ·  ③ mirror from code

Purpose: the actual notification/email templates. Source of truth = code (React
Email + the messaging layer); Notion mirrors a pointer.

- **Author core ≈ minimal:** `Name`, `Key`, `Channel` (→ Communication
  Channels), `Message Category` (→ Message Categories), `External Link`,
  `Sync Status`, `Status`.
- Do not paste template bodies into Notion — they live in code.
