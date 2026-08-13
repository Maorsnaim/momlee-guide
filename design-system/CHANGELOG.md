# Design System Changelog

> Maor-maintained. Flows to Sivan via git. Clear English (dev docs).

## 2026-08-13

- **Full token re-sync from the canonical Momlee 2.0 file** (Maor's 2026-08-12
  confirmation: "the entire Tokens system has been completely changed — scan it
  and change accordingly"; canonical URL fixed to node 191-3). The color system
  is restructured and re-valued: brand `#b05f64 → #9d636b`, text
  `#232323/#5d5d5d`, error `#b05550`, new surface/border semantics
  (bg-surface/bg-canvas/bg-neutral-subtle/bg-brand-subtle,
  border-subtle/primary/secondary/interactive/brand/error), badge +
  featured-icon component colors. NEW groups: `letter-space/*` and `width/*`.
- **Font flipped to Google Sans** (body = "Google Sans 17pt", display =
  "Google Sans") per the 2026-07-19 decision of record — on web via
  self-hosted variable woff2 subsets + `font-optical-sizing: auto`; native
  static TTFs land at native revival.
- Spacing/radius/type numeric scales observed unchanged; unobserved outer
  steps carried over and flagged. Gap list (incl. the `text-on-unread` naming
  question and raw 10px paddings) in tokens.md.
- Mirrored into `@momlee/tokens` (tokens.ts + preset.ts) with per-line Figma
  citations; web primitives Button/Input updated to the new classes.

## 2026-06-09

- Initial snapshot seeded from the Mom onboarding flow. Font set to Noto Sans
  Hebrew (replaced Google Sans).

## 2026-06-10
- Full token harvest from onboarding screens (Welcome, Phone Empty/Error, BabyType Selected): 18 colors, full spacing scale (none-5xl), radii (md/xl/full), typography scale (xs-xl + display-xs line-height). Mirrored into `@momlee/tokens` in the app repo.
- Logged Figma gaps: stray `Brown LL Heb` font on error text, untokenized literals (CTA 14px padding, `#739fd2` boy border, `text-black`, screen-08 white bg), missing `spacing-2xl`/`radius-sm`.

## 2026-06-11
- Full type scale confirmed from the Typography sheet (11 steps; fixed text-lg 18/28, display-lg 48/60; added display-md/xl/2xl; display-xs confirmed 24/32). OTP digits = display-sm.
- Buttons/Button full API adopted in code: Hierarchy (incl. tertiary + link-color #6a393c text-brand-secondary-(700) + link-gray), Size xs-xl, Loading state.
- New tokens: text-brand-secondary-(700), input/disabled pair (#d4d4d4); Input base renamed/aligned to the Figma component (5 states + slots).
- Complete Variables sync: FULL spacing (none-11xl) + radius (none-4xl, sm fixed 4→6) scales from the documentation sheets; icons now from the Figma Icons library via base/Icon (Ionicons stand-ins removed); ProgressBar built; component folders reorganized to base/ + app/forms/ + app/templates/ per the design-system taxonomy.
- Monorepo restructure: design system moved to packages/ui (primitives/forms/brand, folder-per-component, ready for .web variants), business logic to packages/core (i18n/countries/validation), templates to apps/mobile/src/shells. Import surface: `import { Button } from '@momlee/ui'`. LegalText dissolved (inlined — not a DS component).
- Audit round-2 fixes: secure-store session, __DEV__-gated demo bypass, @momlee/core analytics wrapper (typed taxonomy), lint gates extended to packages/ui+core, CTA Loading state, fontByWeight token.

