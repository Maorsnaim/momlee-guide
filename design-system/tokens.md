# Design Tokens

> Maor-maintained. Flows to Sivan via git. Clear English (dev docs).

> **Figma is the runtime source of truth; this file is a maintained snapshot.
> Update on every token change and log it in CHANGELOG.md.**

**SYNCED 2026-08-13 from the CANONICAL Momlee 2.0 file**
(`bPV6lWPTjZPN6pPC4S7J3j`, Mobile page node `191-3`) after Maor confirmed
(2026-08-12) that the entire token system was rebuilt and the old
"Momlee Design System" file is superseded. Harvested via the Figma MCP design
context from 8 onboarding screens (Welcome, Phone Default/Error, OTP,
Location/Results, Status, Children Empty/Twins, Verification Failed,
Waitlist). Code mirror: **`packages/tokens` (`@momlee/tokens`)** in the app
repo — raw tokens + a Tailwind/NativeWind preset. Items marked *carried-over*
come from the superseded old-file harvest and were not observed in 2.0 yet —
see the gap list at the bottom.

## Colors

| Token (Figma) | Value | Use |
|---|---|---|
| colors/background/bg-surface | `#ffffff` | Cards, nav, OTP boxes |
| colors/background/bg-canvas | `#fafafa` | Page wells, secondary/back buttons |
| colors/background/bg-neutral-subtle | `#f3f3f3` | Progress track |
| colors/background/bg-brand-solid | `#9d636b` | Primary brand CTA (rebrand of #b05f64) |
| colors/background/bg-brand-subtle | `#f0e4e6` | Selected radio item |
| colors/foreground/fg-brand-primary | `#b37a81` | Progress fill |
| colors/foreground/fg-on-solid | `#ffffff` | Radio check dot on brand |
| colors/text/text-primary | `#232323` | Headings/primary text |
| colors/text/text-secondary | `#5d5d5d` | Body / filled input |
| colors/text/text-muted | `#5d5d5d` | Legal/hint text |
| colors/text/text-placeholder | `#5d5d5d` | Placeholders (Figma renders at reduced opacity) |
| colors/text/text-tertiary-(600) | `#525252` | Welcome secondary line (legacy name kept in 2.0) |
| colors/text/text-error | `#b05550` | Error text |
| colors/text/text-on-unread | `#ffffff` | Label on brand-solid (naming — see gaps) |
| colors/border/border-subtle | `#f3f3f3` | Nav bottom border |
| colors/border/border-primary | `#e7e7e7` | OTP box border (revalue of #d4d4d4) |
| colors/border/border-secondary | `#d1d1d1` | Secondary button outline |
| colors/border/border-interactive | `#878787` | Input underline, checkbox outline |
| colors/border/border-brand | `#b37a81` | Selected address result |
| colors/border/border-error | `#b05550` | Error input underline |
| colors/base/transparent | `rgba(255,255,255,0)` | Ghost buttons (Skip, resend) |
| component-colors/badge/brand/background | `#f0e4e6` | Type Badge (brand) bg |
| component-colors/badge/brand/foreground | `#835258` | Type Badge (brand) fg — also the inline-link color observed in running text |
| component-colors/badge/blue/background | `#dae9f4` | Type Badge (blue) bg |
| component-colors/badge/blue/foreground | `#466c87` | Type Badge (blue) fg |
| component-colors/featured-icon/brand/background | `#f0e4e6` | Featured Icon circle (brand) |
| component-colors/featured-icon/blue/background | `#dae9f4` | Featured Icon circle (blue) |

## Spacing

Observed in 2.0 (same numeric scale as before):
`none 0 · xxs 2 · xs 4 · sm 6 · md 8 · lg 12 · xl 16 · 2xl 20 · 3xl 24 · 4xl 32 · 7xl 64 · 8xl 80`.
Carried-over pending the 2.0 Variables sheets: `5xl 40 · 6xl 48 · 9xl 96 · 10xl 128 · 11xl 160`.

## Radius

Observed in 2.0: `md 8 · lg 10 (OTP) · xl 12 · 2xl 16 (cards) · full 9999`.
Carried-over: `none 0 · xxs 2 · xs 4 · sm 6 · 3xl 20 · 4xl 24`.

## Typography

Families: **`font-family/font-family-body` = Google Sans 17pt** (body) and
**`font-family/font-family-display` = Google Sans** (display, incl. OTP
digits). Weights observed: 400 / 500 / 600 (SemiBold on the welcome signup
link). On web both families are served by the single variable font
(wght 400–700) with `font-optical-sizing: auto`.

Sizes/line-heights observed unchanged: `text-xs 12/18 · sm 14/20 · md 16/24 ·
lg 18/28 · xl 20/30 · display-sm 30/38 · display-xl (style referenced)`.
Carried-over: display-xs/md/lg/2xl px pairs.

### Letter-spacing — NEW group (`letter-space/*`)

| Token | Value | Use |
|---|---|---|
| letter-space/none | `0` | text-lg headings |
| letter-space/text | `0.4px` | text-xs/sm body |
| letter-space/text-half | `0.2px` | text-md/xl body |
| letter-space/display | `-0.4px` | display headlines |
| letter-space/display-half | `-0.2px` | display-sm (OTP digits) |

### Widths — NEW group (`width/*`)

`width/thin = 1px` (input underlines).

## ⚠️ Gaps / for Maor (2026-08-13 harvest)

- **`text-on-unread` names the on-brand label color** on every CTA — reads
  like a notifications token; confirm or rename before component work bakes
  it in.
- **Not yet observed in 2.0** (carried from the old file, flagged in code):
  input focused/disabled borders (`#404040`/`#d4d4d4`), disabled-arrow
  `fg-quaternary #a3a3a3`, link-color button `#6a393c`, `utility.success`
  `#16a34a`, shadows/focus-ring variables, the display-xs/md/lg/2xl px pairs
  and the outer spacing/radius steps. The ❖ Variables sub-page node-ids would
  close all of these in one pull.
- **Raw values still inside 2.0 components:** button vertical padding `10px`
  (off-scale, everywhere), `gap: 10px/12px` literals, welcome signup link
  `#2f67a6`, welcome headline spans hardcode `#9d636b`/`#b37a81`/black,
  inline links hardcode `#835258` instead of a link token, keyboard overlay
  `#e6e9ed`.
- **The welcome screen (01.1) shows a "Continue with Google" button** — the
  locked auth decision is phone-OTP only (2026-07-21). Design ↔ decision
  conflict raised to Sivan; not built.

## dim — component dimensions

2.0-observed geometry: CTA 48 · welcome/back buttons 44 · list CTAs 40 ·
glyphs 24/20/16 · OTP box min-64 · progress track 8 · checkbox 16 ·
featured-icon circle 32 · logomark 19 · nav logo 121 · welcome logo 126 ·
flag 22.5×15. Carried-over: button xs/sm boxes, OTP md/lg, dial box 63,
splash logo 217, focus-ring 2+2. Each value cites its source in
`packages/tokens` (provenance gate enforced).
