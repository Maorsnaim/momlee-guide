# Momlee OS — The Hub: Feature & Story

> Feature is type ① (a product capability). Story is type ② — the **hub**: a
> small author core PLUS wiring Claude proposes and the human approves. The Story
> is the build unit that compiles into the YAML. Keys per `00-foundations.md`.
> Status: LOCKED.

---

## Feature  ·  `epic.feature` (`meetups.details`)

Purpose: a product capability inside an Epic; the middle layer between a broad
Epic and small Stories.

- **Author core (~6):** `Name`, `Key` (`epic.feature`), `Epic` (parent),
  `Feature Type` (Core Flow / Access Flow / Profile Flow / Admin Flow /
  Monetization / Trust&Safety / Notification / Analytics / Infrastructure),
  `User Value`, `Priority`, **`Scope`** (see `00-foundations.md`).
  Optional: `Description`.
- **Derived (not authored):** `Product Segment` ← rollup from the Epic.
- **Back-relation (never typed):** `Related Stories / Automations / Decisions`.
- **Workflow:** `Status`.

---

## Story  ·  `epic.feature.story` (`meetups.details.mom_views_full_details`)  — THE HUB (type ②)

Purpose: a specific behavior performed by an actor inside a Feature. The unit
that becomes the compiled YAML and goes to development.

### Three buckets

**① Author core (~6) — the human writes the behavior:**
- `Name`
- `Feature` (the anchor; `Epic` is a rollup from it, never re-entered)
- `Story Type` (User Action / System Behavior / Admin Action / Access Behavior /
  Notification Behavior / Error State / Empty State)
- `User Story` ("As a <role>, I want… so that…")
- **`Acceptance Criteria`** — THE KING. The precise, unambiguous behavior
  contract. Everything in bucket ② is compiled from this (+ Figma + code). The
  sharper the AC, the less interpretive the YAML.
- `Priority`
- **`Scope`** (MVP / Post-MVP / Backlog / Cut) — inherits the Feature's default;
  the Story is authoritative for what actually ships.

**② Claude proposes → human approves — the wiring:**
- `Permissions`, `Roles`, `Eligibility`, `Product Rules`
- `Events`, `Automations`
- `Components`, `Design Screens`, `Schema Registry`

Claude reads the Acceptance Criteria + the Figma screens + the code + the
existing registries, and **proposes** which of these apply. The human ticks
"yes." This approval step is exactly where the unambiguous YAML is produced — the
human does NOT hand-pick these from scratch.

**③ Derived / workflow:**
- `Key` ← derived (`epic.feature.story`)
- `Epic` ← rollup from Feature
- `Status` / `Design Status` (incl. `Claude Ready`) / `Dev Status`
- `Tasks` ← Claude splits the Story into tasks
- `Description` (short, optional)

### Why this keeps authoring light

A Story has many fields, but the human still authors only ~6. The richness comes
from **compilation + approval**, not hand-entry. The "Claude Ready" design status
marks a Story whose AC + screens are complete enough to compile.

### The compile flow (Story → YAML)

1. Human authors the ① core (esp. Acceptance Criteria).
2. Claude proposes bucket ② wiring from AC + Figma + code + registries.
3. Human approves ②.
4. Claude compiles ① + ② + ③(code/Figma reality) into the unambiguous YAML.
5. Human reviews the YAML.
6. Claude takes it to development (web / Vercel).

## Scope — MVP membership (LOCKED)

`Scope` (select: `MVP` / `Fast-follow` / `Post-MVP` / `Backlog` / `Cut`; see
`00-foundations.md`) is **orthogonal to
`Status`** (lifecycle). It answers one question: *does this ship in the lean Web
MVP?* (per the 2026-06-28 Native→Web pivot).

- Set on the **Feature** as the planned default for the whole capability.
- Set on the **Story** as the authoritative per-build decision — it inherits the
  Feature's default but can override (ship a thin MVP slice of an otherwise
  Post-MVP feature).
- Drives the roadmap views ("what's in MVP") and keeps MVP tight. A Story can be
  `Status: Draft` **and** `Scope: MVP` at the same time — the two are independent.
