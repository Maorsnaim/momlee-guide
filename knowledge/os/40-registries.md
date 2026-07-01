# Momlee OS — Reality-Mirror Registries (type ③)

> These DBs are **mirrors of an external source of truth** — never hand-authored
> content. Notion holds a thin pointer row so a Story can link to them; the real
> content lives in Figma or in the code. Status: LOCKED.

## The DBs and their source of truth

| DB | Source of truth |
|---|---|
| Components | Figma (+ code components) |
| Design Screens | Figma |
| Design Tokens | Figma |
| Database Tables | Code / Supabase |
| Schema Registry | Code / Supabase |
| APIs | Code |
| Events | Code (analytics registry files) |

## The rule

- **Author core ≈ minimal** — NOT a literal zero. A human may set a few pointer
  fields, but never the full content (no props, schema, API body, or token value
  typed by hand).
- **Mirror row fields:** `Name`, `Key`, `Source`, `External Link`, `Status`,
  **`Sync Status`**.
- **Source of truth = external.** Claude syncs the mirror from Figma / code.
- **`Sync Status` is mandatory** — `In Sync` / `Stale` / `Missing`. It tells you
  whether the Notion mirror matches the external source. A `Stale`/`Missing` row
  is a signal, not data.

## Why they exist at all

The compile step **reads** these registries so that when Claude wires a Story it
**links to a real component / event / table / token — and never invents one.**
If a Story needs something not in the registry (no such component/event yet),
Claude STOPS and asks rather than fabricating it (the `momlee-prompt-guard` rule).

## What is NOT here

The actual content — a component's props, a table's columns, an API's body, a
token's value — stays in Figma / code. Do not copy it into Notion. The mirror is
an index for linking and a freshness signal, nothing more.

## Sync Status — standard + current gap (light pass, 2026-06-28)

Standard `Sync Status` enum (adopt the Schema Registry's): `Unknown` / `Aligned`
/ `Needs Check` / `Mismatch` / `Deprecated`.

Live-Notion gap: **Schema Registry already has it; most other ③ DBs (Events,
Components, Design Screens, Design Tokens, Database Tables, APIs, Templates) do
NOT.** Notion-alignment action: add `Sync Status` to every ③ DB that lacks it.

**Schema Registry is the exemplar + the privacy backbone.** It already carries,
per field: `Contains PII` / `Data Sensitivity` / `Exposure Level` / `Privacy
Notes`. This is the source that `Entity.Privacy Class` rolls up from and that the
Data Inventory compiles from (see Entity in `10-intent-dbs.md` + `90-dispositions.md`).
Components is a clean Figma-mirror (Figma ID + link + code pointer + light spec)
— just add `Sync Status`. Events is under-built — see `50-behavior-messaging.md`.
