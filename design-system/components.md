# Components

> Maor-maintained. Flows to Sivan via git. Clear English (dev docs).

## Principle

**One Figma component = one code component in the shared UI layer.** Reuse
before create. A design change happens in **one place** — the shared component —
and propagates everywhere it is used. Never fork a component to tweak it locally.

This registry is the **first stop of the mandatory Component Reuse Audit**
(see `../skills/momlee-design-system/SKILL.md`): before creating any component,
search BOTH tables below, then the Figma inventory, then the code — and show
the audit. Every CREATE verdict must add its row here in the same change.

## The shared component contract (ADR-016) — format + registry

> Added 2026-08-13 (Sivan's build, per the Story
> `platform.web_design_foundation.wire_tokens_and_contract`). **One Figma
> component = ONE row = up to two implementations** (native RN+NativeWind in
> `packages/ui`, web DOM+Radix+Tailwind in `apps/web/src/components/ui/primitives`)
> sharing the same name, the same props API, the same Figma node and the same
> tokens. A row with one implementation is a component the other platform has
> not needed yet — never a license to fork the API.

**Row format:** `| Component | Figma node | Shared props API | Native | Web | Notes |`

| Component | Figma node | Shared props API | Native | Web | Notes |
|---|---|---|---|---|---|
| AppText | none — text primitive | `align: 'right'\|'center'\|'ltr'` · `weight: regular\|medium\|semibold\|bold` | ✅ `packages/ui` | ✅ `apps/web` | sanctioned divergence: native = per-weight family (fontByWeight), web = font-weight utilities |
| Button | `219:6039` (2.0 `Buttons` set — FULL redesign, web rebuilt 2026-08-16) | `kind: text\|icon` · `hierarchy: brand-solid\|neutral-solid\|neutral-outline\|neutral-subtle\|inverse\|destructive\|link` · `size: xs(30)\|sm(40)\|md(44)\|lg(48)\|xl(58)` (icon boxes 32/36/40/44/48) · `loading` · `label` | ⚠️ pre-2.0 API (primary/secondary/… + social) — realign follow-up incl. screen usages | ✅ `apps/web` | Loading = hover bg + always-animated loader-circle beside the visible label; Disabled = 50% wash; Focused = white 2px inner + brand outer double ring. `Destructive Dialog Buttons` (868:15515) = destructive-md over neutral-solid-md, full width, gap-md — composed at screen level. Figma anomalies flagged: focus ring aliased to a Meetup-RSVP color, Link label un-tokenized, white-on-solid bound to text-on-unread |
| Icon | `3463:407484` (old file — 2.0 twin = the lucide set) | `name` (semantic `forward`/`backward` mirror per direction) · `size` · `color` | ✅ SVG assets | ✅ inlined paths | lucide is the official library (Maor 2026-07-30); pinned `lucide-react`/`-native` adoption is a build task |
| Input | `262:9244` (2.0 `Forms / Input`, remapped 2026-08-16) | `value` · `onValueChange` · `state: Empty\|Focused\|Filled\|Error\|Disabled` · `error` · `startExtras`/`endExtras` · `align` | ✅ | ✅ | verified vs the 2.0 master: Error turns the TEXT red (425:1312), focused/filled text = primary, Empty row at 80% opacity. Figma `State=Placeholder` ↔ code `Empty` — coordinated rename pending. **Input NEVER grows a label** (Maor 2026-08-10) |
| OtpField + OtpInput | `280:4284` (2.0 `Forms / OTP Field`) | cell: `digit` · `size: xs\|sm\|md\|lg` · `state: Placeholder\|Focused\|Filled\|Disabled\|Error`; composition: `value` · `onValueChange` · `onComplete` · `length=6` · `error` · `disabled` · `label` | ✅ `packages/ui` (OtpFieldBase/OtpInput — pre-2.0 visuals, realign follow-up; node remapped) | ✅ `apps/web` (2026-08-16) | 2.0 bindings: Focused/Filled = 2px border-brand + brand-display digits; Error = border-brand-subtle + error digits; Placeholder/Disabled digits at 40%; display-family type per size. Digits LTR; behavior (auto-submit/resend) lives in the SCREEN per the 03.1 annotations |
| BrandMark | `149:1316` (2.0 `Logo` set; heart = `Logomark` 35:1193) | `variant: wordmark\|heart` · `width` | ✅ `packages/ui` | ✅ `apps/web` (2026-08-16) | exact SVG assets in `/public/brand`, colors baked (never restyled/mirrored); heart = Pink S — White/Black tints when a screen needs them |
| Navigation | `280:4131` (2.0 `Navigation`, Type=Logo Only) | — (static top bar) | — | ✅ `apps/web` (2026-08-16) | bg-surface + thin border-subtle bottom, centered 121×44 wordmark; other nav types at their screens |
| OnboardingPageTemplate + ProgressBar | `809:23396` (2.0 `Page Templates / Onboarding`, State=Default) | `title`/`subtitle`/`showLogomark` · `progress` · `onBack`/`onSkip` · `children` · `onNext`/`nextDisabled`/`nextLoading` · a11y labels | native shell exists (pre-2.0 — realign follow-up) | ✅ `apps/web` (2026-08-16) | composes Navigation + TopButtons (425:4625 — back = neutral-solid lg icon Button, skip = link Button) + PageTitle (425:2889) + Bottom Next (brand xl icon Button). **State=Block (809:23397) not pulled — follow-up**; Heart Image (817:24454) deferred with it; ProgressBar node fills at the ↳ Progress slice |
| EmailField | `817:23649` (2.0 `Forms / Email`) | `value` · `onValueChange` · `button: Inline\|Block` · `buttonLabel` · `onSubmit` · `consentChecked`/`onConsentChange`/`consentText` (screen passes the Figma copy + policy links; unchecked by default — Amendment 40) · `success`/`successText` (master copy default) · `error` · `helpText` · `label` | — | ✅ `apps/web` (2026-08-16) | mail leading icon; Inline = 32px round brand arrow, Block = 40px pill; Success hides consent and shows the check indicator; consent timestamp + transactional/marketing split = the screen's job |
| FormTextIndicator | `818:25746` (2.0 `Form Text Indicator`) | `state: Disabled\|Default\|Error\|Success` · `text` · `showIcon` · `icon` (swappable, default lucide/circle) | — | ✅ `apps/web` (2026-08-16) | 16px icon + text-xs Medium; disabled/secondary/error/success text colors |
| RadioGroupItem | `454:7448` (2.0 `Radio Group Item`) | `label` · `selected` · `onSelect` · `icon` (Show Icon slot) | — | ✅ `apps/web` (2026-08-16) | Default bg-canvas · Hover bg-surface-hover (602:19404) · Selected bg-brand-subtle; 16px radio (border-interactive → brand-solid + on-solid dot); role=radio |
| AddressSearch | `583:17307` (2.0 `Forms/Address Search`) | `value` · `onValueChange` · `onClear` · `resultsState: Hidden\|Loading\|Results\|NoResults\|Error` · `children` (result items) · `errorText` (Figma `Show Error Text`) · `label` | — | ✅ `apps/web` (2026-08-16) | PRESENTATIONAL composite: Input + search icon (+ clear-x when filled) over the results panel, gap-lg; geocoding/Mapbox/city-validation live in the 06.x screens per their annotations |
| AddressResults | `575:16442` (2.0 `Forms/Address Results`) | `state: Loading\|Results\|NoResults\|Error` · `children` | — | ✅ `apps/web` (2026-08-16) | panel bg-surface/border-primary/rounded-md, h-244 scrollable, hairline dividers; non-Results states render SearchStatus inside |
| AddressResultItem | `572:14768` (2.0 `Forms/Address Result Item`) | `title: ReactNode` (screen highlights the matched part) · `subtitle` · `onSelect` | — | ✅ `apps/web` (2026-08-16) | 44px row, brand Featured Icon (32, map-pin); **Hover binds bg-canvas** (583:17650); 10px side padding = the master's raw value (flagged) |
| SearchStatus | `575:16566` (2.0 `Feedback/Search Status`) | `state: Loading\|Empty\|Error` · `text` (defaults = master copy) | — | ✅ `apps/web` (2026-08-16) | 30px lucide icon + text-xs/Medium; Loading = the canonical loader-circle spinner (always animated, 1s linear); Empty/Error copy in text-error |
| DateField | `439:3224` (2.0 `Forms / Date`) | `title`/`showTitle` (built-in title row) · `value {day,month,year}` · `onValueChange` · `helpText` · `error` · per-part overrides `dayState`/`monthState`/`yearState` (the Figma properties) · `disabled` | — | ✅ `apps/web` (2026-08-16) | three composed `Input`s (Day start-side in RTL), numeric 2/2/4, placeholders = master copy (יום/חודש/שנה); NO focus auto-advance — the 05.x screens carry no such annotation (added at screen build if Maor annotates it) |
| PhoneField | `425:1371` (2.0 `Forms / Phone Field`) | `value` · `onValueChange` · `helpText` (Figma `Show Help Text`) · `error` (renders in the help line) · `disabled` | ✅ `packages/ui` (pre-2.0 shape — realign follow-up) | ✅ `apps/web` (2026-08-16) | fixed non-interactive `+972` + IL flag (Country Calling Code annotation: Israel-only until international ships); dial box binds border-STRONG on focus while the phone input binds brand; composes `Input` |
| Field | `868:23214` (2.0 `Forms / Field`, ↳ Inputs) | `showTitle` (Figma `Show Field Title`, default true) · `title` (Figma `Text`) · `children` = the composed control (an `Input`; later Phone/Date/OTP per the node's instance-swap note) | — | ✅ `apps/web` (2026-08-16) | label = text-xs/Medium/text-secondary/tracking-text, gap spacing-md; rendered as `<label>` for implicit a11y association; Input NEVER grows a label (Maor's annotated contract) |

## Observed components (seed)

From the Mom onboarding Figma. **All are "planned" — not built yet.**

| Component | Where seen | Reuse status |
|-----------|------------|--------------|
| Status bar (iPhone) | Onboarding screens | planned |
| Create Account / primary CTA button | Welcome, onboarding | planned |
| Sign In button | Welcome | planned |
| Social buttons | Welcome | planned |
| Skip button | Onboarding | planned |
| Back button | Onboarding | planned |
| Page heading (text) | Onboarding screens | planned |
| Hint text | Onboarding screens | planned |
| Text input | Phone, Name, OTP, BirthDate | planned |

## Built in code (apps/mobile/src/components) — status 2026-06-11

| Component | Purpose | Notes |
|---|---|---|
| AppText | THE text primitive — RTL base, align (right/center/ltr), weight (Noto per-family) | every user-facing text |
| PageTemplate | onboarding chrome: heart, heading, content, CTA; keyboard container model; optional BackButton | single-container keyboard rules |
| Button | ONE component = the full Figma `Buttons/Button` set (3287:427074): `kind:'text'` with **Hierarchy** primary/secondary/tertiary/link-color/link-gray, **Size** xs32/sm36/md40/lg44/xl48, **State** loading (spinner); `kind:'icon'` (Icon only — forward/back, semantic); `kind:'social'` | Figma Variables = props; back-on-phone is a product decision (2026-06-11), pending a Figma variant |
| Input (+INPUT_TEXT_STYLE) | the Figma `Input` base (17297:8153): five states Empty/Focused/Filled/Error/Disabled + ExtrasBefore/After slots | field compositions build on it: Forms/PhoneField (built), Forms/FullNameField + Forms/DateOfBirthField (when Name/BirthDate screens land), Forms/CountryDropdown (to be re-composed on Input slots) |
| PhoneField | LTR digits row: pressable dial box + number | dial press reopens country selector |
| CountryDropdown (+flags.ts, 48 Figma flag SVGs) | inline type-to-search dropdown, Israel default | per screen-04 annotation |
| FullNameField | Figma `Forms / FullNameField` (17333:96818): two Input-base fields, first name at START, green check on filled (Icons-library `check`, token utility.success) | Name screens 12-15 |
| OtpFieldBase | the Figma `OTP Field base` (1106:66560, renamed 2026-06-12): Size xs64(display-sm digits — the screens)/sm64(display-lg)/md80/lg96 × State Placeholder/Filled/Focused(border-brand-alt, no ring)/Disabled/Error |
| OtpInput | composes 6× OtpFieldBase (xs); autofocus on entry; real Focused on the active cell; auto-submit 450ms after the 6th digit (so it's seen); Error state on failed code | |
| BrandMark | one component, `variant: 'wordmark' | 'heart'` (Figma variants) | brand marks never mirror |
| LegalText | SMS consent copy | exact Figma copy |

Libs: i18n (typed t, directionFor), icons (FORWARD/BACKWARD), phone (R1-R4), countries (+search), analytics (first-party, anon_id), auth (requestOtp/verifyOtp), supabase (lazy fail-soft), rtl.

### Taxonomy update (2026-06-11)
Code folders now mirror the design system: `base/` (AppText, Button, Input, Icon, BrandMark, ProgressBar) · `app/forms/` (PhoneField, CountryDropdown, OtpInput, flags) · `app/templates/` (OnboardingPageTemplate; future templates side-by-side) · `app/` (LegalText). New: **Icon** (Figma Icons library, semantic forward/backward) and **ProgressBar** (Figma 1085:57382, Label=False) — ready for the Name screen's top-nav.

### Social buttons (2026-06-11)
`Button kind='social'` mirrors `Buttons/Social button`: Size md(40)/lg(44), Supporting text (label-in-button), Theme per Maor: Google/Facebook = Brand (provider colors, as placed on the welcome frame), **Apple = Gray** (decision 2026-06-11); providers google/facebook/apple (twitter pending — note: **Sign in with Apple is REQUIRED by App Store review** once Google/FB login ship).

