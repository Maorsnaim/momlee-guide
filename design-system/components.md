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
| Navigation | `280:4130` (2.0 `Navigation` set: Logo Only 280:4131 + Full Header 149:2056) | `type: logo-only\|full-header` · `onMenu`/`menuLabel` · `onSearch`/`searchLabel` | — | ✅ `apps/web` (2026-08-16) | bg-surface + thin border-subtle bottom, centered 121×44 wordmark; Full Header adds the 24px hamburger (start side) + 24px search (end side); the master's container gap is a raw 10px (the flagged raw-10px family — moot while the center is flexible) |
| OnboardingPageTemplate + ProgressBar | `809:23396` (2.0 `Page Templates / Onboarding`, State=Default) | `title`/`subtitle`/`showLogomark` · `progress` · `onBack`/`onSkip` · `children` · `onNext`/`nextDisabled`/`nextLoading` · a11y labels | native shell exists (pre-2.0 — realign follow-up) | ✅ `apps/web` (2026-08-16) | composes Navigation + TopButtons (425:4625 — back = neutral-solid lg icon Button, skip = link Button) + PageTitle (425:2889) + Bottom Next (brand xl icon Button). **State=Block (809:23397) not pulled — follow-up**; Heart Image (817:24454) deferred with it; ProgressBar = the 2.0 `Progress bar` set **280:1163** (↳ Progress; shell uses Label=False — Label Right/Bottom/floating variants + Tooltip 280:1455 at first use) |
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
| CheckboxBase | `450:6814` (2.0 `Checkbox base` set, ↳ Checkboxes) | `type: checkbox\|radio` · `size: sm(16)\|md(20)` · `checked` · `indeterminate` · `disabled` · `error` · `borderStyle: primary\|interactive` | — | ✅ `apps/web` (2026-08-16) | a native input drives a drawn box; checked = brand-solid + on-solid lucide check/minus glyph (sm 12 / md 14 — consolidate into Icon at the Icons-page slice), radio dot 6/8; Disabled = 50% wash (+ bg-neutral-subtle unchecked); Hover has NO visual delta on the masters; Focus double ring bound BY NAME to focus-ring-inner/focus-ring-default — the master binds unexported `bg-primary` + focus-ring resolving #b05f64 (flagged, from-sivan 2026-08-16 evening) |
| Checkbox | `450:6541` (2.0 `Checkbox` labeled set) | `label` · `supportingText` · `id` + the CheckboxBase props | — | ✅ `apps/web` (2026-08-16) | gap-md row, box pt-xxs to the first text line; label text-sm Medium tracking-text, supporting text-sm Regular text-placeholder; Error = border-error box AND both lines text-error (818:25117); the labeled masters bind border-interactive (vs the bare base's border-primary); md labeled row type style = verify at first md use |
| Badge | `730:9618` (2.0 `Badge` set, ↳ Badges) | `size: xs(18)\|sm(22)\|md(24)\|lg(28)` · `color` (14 badge-palette colors + base1/base2) · `dot` · `icon` + `iconPosition: leading\|trailing` · `iconOnly` · `avatarSrc` · `onClose` + `closeLabel` | — | ✅ `apps/web` (2026-08-16) | rounded-full Medium tracking-text pill (xs text-2xs / sm text-xs / md+lg text-sm); ⚠ every master binds the badge FOREGROUND token as the border (the palette's border tokens go unused — flagged); Base 1/2 = bg-surface + border-primary + fg-primary/fg-secondary; slot metrics verified on md — non-md icon/avatar sizes verify at first use; X Close restyles the pill px-md py-xs text-xs; Avatar hairline = alpha-black20 (master bakes raw 16% black — the future Avatar primitive resolves it) |
| BadgeCloseX | `740:9547` (2.0 `Badge Close X` set) | `label` (a11y) · `onClick` · `disabled` | — | ✅ `apps/web` (2026-08-16) | built as the in-badge form (2xs Square: p-0, 12px x, currentColor tint); the standalone sizes (xl = p-xs + 24px x) and Hover/Disabled/Border/Round variants at first use |
| Badge group | `506:13337` (2.0 `Badge group` set) | — | — | ⛔ blocked | binds `component-colors/utility/brand/*` (utility-brand-50/200/700) which is ABSENT from the 2026-08-16 variables export — build starts when the utility ramp lands in an export (from-sivan 2026-08-16 evening) |
| NumberBubble | `15:89` (2.0 `Number Bubble` set, ↳ Navigation) | `size: md(20)\|sm(16)\|dot(10)` · `count` | — | ✅ `apps/web` (2026-08-16) | brand-solid circle; md count = text-xs Regular on-solid tracking-text; the sm master's count type is RAW 8px/11.2/0.32 (un-tokenized in Figma — transcribed as-is); overflow formatting (e.g. "9+") = the screen's job |
| BottomNavItem | `16:102` (2.0 `Bottom Nav Item` set) | `icon` · `label` · `active` · `count` (number or `dot`) · `onClick` | — | ✅ `apps/web` (2026-08-16) | flexible column tab: px-xl py-md gap-xs, 24px icon + text-xs SemiBold text-primary tracking-text; Active = 2px (width/medium) border-brand bottom edge + aria-current; bubble pinned top 1px / start 6.4px (master raw values); the composite's inactive items carry the same border at width 0 — a no-op, omitted |
| BottomNavigation | `17:44` (2.0 `Bottom Navigation`) | `children` = BottomNavItems | — | ✅ `apps/web` (2026-08-16) | h-102 bg-surface px-lg py-md gap-md `<nav>`; the master's five items (פרופיל/שמורים/הודעות/התראות/מיטאפים) come from the screen; the master's `Vertical Divider` nodes are degenerate zero-width frames rendering nothing — omitted, revisit if a screen shows one |
| MenuItem | `161:2967` (2.0 `MenuItem` set, ↳ Navigation) | `label` · `icon` (18px swappable slot, master placeholder = lucide/circle) · `showIcon` · `showDivider` · `onClick` | — | ✅ `apps/web` (2026-08-16) | full-width py-md gap-md row hugging the start side, label text-sm Medium tracking-text; ShowDivider=True = thin border-subtle bottom |
| FlyingMenu | `155:2436` (2.0 `FlyingMenu`) | `children` = MenuItems | — | ✅ `apps/web` (2026-08-16) | the PANEL only: 273px full-height bg-surface column, py-3xl, items container px-3xl; open/close/overlay behavior = the screen summoning it from the Full Header hamburger |
| SearchBox | `155:2656` (2.0 `SearchBox`; content 155:2653) | `value` · `onValueChange` · `onSubmit` (Enter + the round button) · `placeholder` (master copy "חפשי כל דבר..") · `label`/`submitLabel` | — | ✅ `apps/web` (2026-08-16) | ON-SOLID design (white text/underline — sits on a brand surface): composes the brand-solid md icon `Button` (the master literally instances the Buttons set, 276:9754) + the underline `Base Input` look (instance of 257:9083: thin border-on-solid bottom, py-xs, text-md Medium tracking-text-half); container rounded-2xl p-[10px] (raw-10px family) |

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

