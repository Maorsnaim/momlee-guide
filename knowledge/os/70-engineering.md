# Momlee OS — Engineering (ADRs, Principles)

> The engineering-decision layer. ADRs and Principles are **full authored Notion
> DBs** (type ① — the engineering parallel to Decision Log / Product Rules), and
> they are the link targets that **Validators** maps checks onto. Technology Stack
> is NOT a Notion DB — it lives in the plugin (`knowledge/stack.md`). Status: LOCKED.

---

## ADR  ·  Architecture Decision Record  (full authored DB, type ①)

Purpose: a recorded architecture decision — the engineering parallel to the
Decision Log. Authored in Notion (not a mirror, not thin).

- **Author core:** `Name`, `Context`, `Decision`, `Consequences`, `Alternatives
  Considered`, `Date`, `Status` (Accepted / Proposed / Superseded / Deprecated).
  Optional: `Key` / number (`ADR-001`) for stable reference.
- **Tracking:** `Implementation Status` (Implemented / Partial / Not Implemented /
  **Diverged**) — the "decision vs reality" tracker; very useful.
- **Relations:** `Validators` (back — the checks enforcing it), `Related Epics`
  (renamed from `Related Modules`), `Related Entities`, `Related Architecture
  Components`.

## Principle  ·  Engineering Principle  (full authored DB, type ①)

Purpose: a standing engineering rule (e.g. "screens never import Supabase").
Cross-cutting; usually global.

- **Author core:** `Name` (the title — rename live `Rule` → `Name`/`Principle` to
  avoid confusion with Product Rules), `Principle Statement`, `Why`, `Category`
  (Architecture / Security / API / Tests / Authorization / Trust&Safety /
  Analytics / Frontend / Performance / Product Architecture), `Enforcement Level`
  (consider a select: Must / Should / Recommended).
- **Relations:** `Validators` (back — the checks enforcing it). Optional: `Key`.

## How they tie in

`ADR` / `Principle` → `Validators` → code check. A principle or ADR with no
validator is an enforcement `[GAP]` (visible via Validators). This is the
engineering half of the rules→enforcement bridge; the product half is
`Product Rule` → `Validators` (see `60-validators.md`).

## Technology Stack — NOT a Notion DB

The stack lives in the plugin: `knowledge/stack.md` (the canonical stack + the
Dependency Governance gate). The old Notion "Technology Stack" page is a stale
copy with broken relations — do not maintain it; point to the plugin instead.
