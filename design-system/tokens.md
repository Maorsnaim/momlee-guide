# Design Tokens

> Maor-maintained. Flows to Sivan via git. Clear English (dev docs).

> **Figma is the runtime source of truth; this file is a maintained snapshot.
> Update on every token change and log it in CHANGELOG.md.**

**COMPLETE as of 2026-08-16.** Source: the Momlee 2.0 Variables collections
(`bPV6lWPTjZPN6pPC4S7J3j`) — the full "1. Color modes" + "Primitives" plugin
export plus row-by-row confirmation of the Radius/Spacing/Typography/Border
collections from the Variables panel (Sivan). **Code mirror:
`packages/tokens` (`@momlee/tokens`)** — every value carries its variable
citation; nothing is carried-over or interpolated any more.

## Collections (466 variables total)

| Collection | Count | Contents |
|---|---|---|
| Primitives | 206 | base (white/black/transparent) · neutral 25–950 · **Brand: Velvet 50–950 (legacy #b05f64=500) + Dusty Pink 50–950 (the ACTIVE brand — solid #9d636b=600)** · Illustration set · 11-step ramps: Orange, Purple, Apricot, Honey Yellow, Teal, Lavender, Sage, Olive, Green, Red, Blue |
| 1. Color modes | 205 | the semantic layer (lightMode): backgrounds incl. hover/pressed/disabled states, text incl. link/disabled/on-*, borders incl. strong/interactive/disabled, foregrounds, focus rings, 17 shadow colors, presence, featured-icon (10 palettes), badge (15 palettes × bg/fg/border), notification badge, **Meetup RSVP (Idle/Interested/Going × 5 props)**, alpha white/black ramps |
| 2. Radius | 11 | none 0 · xxs 2 · xs 4 · sm 6 · md 8 · lg 10 · xl 12 · 2xl 16 · 3xl 20 · 4xl 24 · full 9999 |
| 3. Spacing | 17 | none 0 · xxs 2 · xs 4 · sm 6 · md 8 · lg 12 · xl 16 · 2xl 20 · 3xl 24 · 4xl 32 · 5xl 40 · 6xl 48 · 7xl 64 · 8xl 80 · 9xl 96 · 10xl 128 · 11xl 160 |
| 6. Typography | 35 | families: display=**Google Sans**, body=**Google Sans 17pt** · weights Regular/Medium/SemiBold/Bold · 12 sizes text-2xs 10 → display-2xl 72 · 12 line-heights (see below) · letter-space none 0 / text 0.4 / text-half 0.2 / display −0.4 / display-half −0.2 |
| 7. Border | 5 | width: none 0 · thin 1 · medium 2 · thick 4 · double 8 |

## Key semantic values (light mode)

- **Brand:** bg-brand-solid `#9d636b` (hover `#835258`, pressed `#694147`, disabled `#e7e7e7`) · bg-brand-subtle `#f0e4e6` · fg-brand-primary `#b37a81` · text-brand-display `#9d636b` / -soft `#b37a81`
- **Text:** primary `#232323` · secondary/muted/placeholder `#5d5d5d` · disabled `#9e9e9e` · **link `#835258`** (hover `#694147`) · error `#b05550` · on-solid/on-unread white
- **Borders:** subtle `#f3f3f3` · primary `#e7e7e7` · secondary `#d1d1d1` · interactive `#878787` · strong `#232323` · brand `#b37a81` · error `#b05550` · disabled `#d1d1d1`
- **Input states (as bound on the phone screens):** Empty=interactive · **Focused=brand** · **Filled=strong** · Error=error · Disabled=disabled
- **Status:** error solid `#b05550` · warning solid `#a7611b` · success solid `#5f8568` (+subtle/fg/on-* each)
- **Focus rings:** `#494949`, error `#cf6963`, inner white
- **Type scale (size/line-height):** 2xs 10/14 · xs 12/18 · sm 14/20 · md 16/24 · lg 18/28 · xl 20/30 · display-xs 24/32 · sm 30/38 · md 36/44 · **lg 48/56 · xl 60/68 · 2xl 72/80**

## ⚠️ For Maor — stale bindings found while syncing (collections win)

- The ↳ Typography SHEET's specimens bind old line-heights (display-lg 60, display-xl 72, display-2xl 90, text-2xs 8) — the collections say **56/68/80/14**; please rebind the sheet.
- Earlier flags stand: some display specimens still bound to Noto; Display xl styles carry a raw −2px letterSpacing; `text-on-unread` naming; raw 10px button paddings and gap literals in components (the former "raw link hexes" are RESOLVED — they are the `text-link`/`text-brand-display` tokens).

## dim — component dimensions

Unchanged from 2026-08-13 (2.0-observed geometry; each value cited in `packages/tokens`).
