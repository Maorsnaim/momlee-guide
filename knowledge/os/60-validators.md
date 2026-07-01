# Momlee OS — Validators (mechanical enforcement)

> The bridge from rules to enforcement: every enforceable Product Rule / ADR /
> Principle / Design rule maps to a concrete check, so the AI can't silently do
> the wrong thing. **Source of truth for the checks = code/plugin; Notion holds
> the map + the gaps.** Mined from the old Notion "11 - Validators" (2026-06-24).
> Status: LOCKED.

## Why it's critical

This is the layer that operationalizes the whole OS thesis ("unambiguous, no
inventing"). A rule that isn't mapped to a validator is **not enforced** — it's a
visible `[GAP]`. A validator turns "we agreed X" into "CI/Claude blocks not-X."

## Where the checks actually live (NOT Notion)

- **ESLint rules** — `apps/*/eslint.config.js` (architecture, tokens-only, RTL,
  permissions wrapper, analytics wrapper, dates, …)
- **CI scripts** — `scripts/momlee/*.mjs` (+ `stack-allowlist.json`)
- **Claude gates** — the `momlee-guide` plugin skills (print-the-block gates:
  REUSE AUDIT / MIGRATION GATE / ANALYTICS GATE / UPLOAD GATE …)
- **Hooks** — `momlee-guide/hooks/` (harness worklog Stop hook)

Notion's Validators DB is the **human-readable index** of all of the above + a
gap tracker. (It overlaps with the enforcement already shipping in the plugin.)

## Validators DB — fields

- **Author core (the curated map):** `Name`, `Validator Type` (ESLint Rule /
  Static Code Check / AST Check / Unit / Integration / E2E / Schema Validation /
  RLS Test / Design System Check / Architecture Check / Privacy Check /
  Performance Check), `Run Mode` (Manual / Claude Preflight / Claude Postflight /
  CI / Local Script), `Failure Behavior` (Warning / Block Task / Block Merge /
  Needs Review), `CI Required`, `Enforces` (what it enforces, in words).
- **③ code pointer:** `Source File` (where the check lives) + sync status implied
  by `Status: Missing CI`.
- **Relations (what it enforces):** `Related Product Rules`, `Related ADRs`,
  `Related Principles`, `Related Epics`, `Related Features`, `Related Design
  System Area`.
- **Workflow / gap:** `Status` (Active / Draft / Deprecated / **Missing CI**).
  `Missing CI` and `[GAP]`-prefixed rows = rules not yet mechanically enforced.

## Failure-behavior legend

- `Block Merge` — CI fails the PR.
- `Block Task` — Claude must NOT finish the task.
- `Needs Review` — surfaced to Maor for a decision.
- `Warning` — noted, non-blocking.

## The Claude enforcement workflow (every implementation task)

1. **Load context** — figma-first pulls design + the relevant ADRs / Principles /
   Product Rules.
2. **Identify validators** — before coding, list which validators apply and run
   the Claude-preflight gates (print the REUSE/MIGRATION/ANALYTICS/UPLOAD blocks).
3. **Implement** — inside the boundaries the gates declared.
4. **Run validators** — `pnpm lint` + `scripts/momlee/*` + `pnpm type-check` +
   `pnpm test`; the worklog hook blocks ending the turn until the commit is logged.
5. **Report pass/fail** — explicitly list which validators ran and their result.
   NEVER mark a task complete without listing the validators run.
6. **Fix before complete** — any `Block Merge` / `Block Task` failure is fixed (or
   explicitly waived by Maor) before the task is done.

## How it ties the OS together

Product Rule → Validator(s) → code check. A Product Rule with no validator is a
governance gap (surfaced, not hidden). This is the mechanism behind the compiled
YAML being trustworthy: the Story's rules don't just exist on paper — each is
backed by a validator that blocks violations.
