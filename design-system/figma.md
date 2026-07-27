# Figma — Momlee Design System

> Maor-maintained. Flows to Sivan via git. Clear English (dev docs).

This file is the map to the Momlee Figma design file: where it lives, how to
read its structure, and how to access it through the Figma MCP.

## The file

- **Figma file:** Momlee Design System
- **URL:** https://www.figma.com/design/zDwy1JIV8htDxYluYTPbfq/Momlee-Design-System
- **fileKey:** `zDwy1JIV8htDxYluYTPbfq`

## Node map

| nodeId | Page | Contents |
|--------|------|----------|
| `16819:38317` | Flows/Screens | The **Mom onboarding flow** — ~30 iPhone screens at 402px width. |

> **Only the Mom onboarding flow has been designed so far, and it is NOT final
> yet.** No Pro surface, no other flows exist in Figma at this point. Treat
> everything as a moving target until Maor marks it final.

## URL → nodeId parsing

Figma URLs follow this shape:

```
figma.com/design/:fileKey/...?node-id=:nodeId
```

To turn a URL's `node-id` into the nodeId the MCP expects, **convert `-` to `:`**.

Example: `node-id=16819-38317` → nodeId `16819:38317`.

## MCP access notes (IMPORTANT)

- `get_design_context` and `get_metadata` work **REMOTELY** with just the
  **fileKey + nodeId** — no Figma Desktop app required.
- `get_design_context` output **contains the token variable references**, e.g.
  `--colors/background/bg-brand-solid: #b05f64`. This is how you read tokens
  without the desktop app.
- `get_variable_defs` does **NOT** work remotely — it errors with **"nothing
  selected"** unless the Figma Desktop app is open with a layer selected.
- **So: prefer `get_design_context` to read tokens.**

## Momlee 2.0 — the NEW Design System file (fileKey `bPV6lWPTjZPN6pPC4S7J3j`)

Maor is building the next-generation DS file. It is **NOT canonical yet** —
code twins and `// figma:` refs still point at the old file above; a mapped
switchover will be announced. When working IN the 2.0 file, these placement
laws are binding (Maor, 2026-07-27):

1. **`❖` pages are section headers only** — zero children, ever. A component
   sitting on a `❖` page is a filing bug.
2. **Components live only in `↳` sub-pages**, assigned by the knowledge test:
   knows nothing about Momlee (button, field, divider) → a sub-page under
   `❖ Base Components` (Buttons / Inputs / Navigation / Dividers /
   Controls) · knows a domain concept (meetup, notification) → a sub-page
   under `❖ App Components` (Meetups / List Items / Callouts) · exists to
   serve a page template → `↳ Page Templates` · presentation prop (e.g. iOS
   Keyboard) → `↳ Design annotations`.
3. **`❖ Screens` pages hold INSTANCES only** — never a master.
4. **Behavioral annotations live on the FIRST frame of a flow** (the
   `/Placeholder` frame); sibling state frames carry only state-specific
   deltas. A dev session entering from any state frame must open the flow's
   first frame for the spec.
5. **User-facing microcopy is a component with cases** (e.g.
   `Onboarding Step / Note`), instanced by screens — never raw text
   copy-pasted between frames (that is how the stale-SMS-text drift
   happened).

Known naming in 2.0: form fields live on `↳ Inputs` as `Forms / Input`,
`Forms / Phone Field`, `Forms / Date`, `Forms / OTP Field`; flags are ONE
set `Flag` (Country x Style, 783 variants, ISO codes in variant
descriptions); RSVP is `Meetup RSVP` (Status=Idle/Interested/Going).
The 2.0 file's fonts (Google Sans / Google Sans 17pt) are NOT in Figma's
cloud font service yet — install the OFL download locally or text-bearing
nodes show missing-font warnings and MCP automation cannot move/edit them.
