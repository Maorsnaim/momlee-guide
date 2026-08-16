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
| Button | `3287:427074` (old file — 2.0 twin remap pending) | `kind: text\|icon(\|social native-only)` · `hierarchy: primary\|secondary\|tertiary\|link-color\|link-gray` · `size: xs\|sm\|md\|lg\|xl` · `loading` · `label` | ✅ | ✅ (no social yet) | colors via token classes only; 2.0 values since the 2026-08-13 sync |
| Icon | `3463:407484` (old file — 2.0 twin = the lucide set) | `name` (semantic `forward`/`backward` mirror per direction) · `size` · `color` | ✅ SVG assets | ✅ inlined paths | lucide is the official library (Maor 2026-07-30); pinned `lucide-react`/`-native` adoption is a build task |
| Input | `262:9244` (2.0 `Forms / Input`, remapped 2026-08-16) | `value` · `onValueChange` · `state: Empty\|Focused\|Filled\|Error\|Disabled` · `error` · `startExtras`/`endExtras` · `align` | ✅ | ✅ | verified vs the 2.0 master: Error turns the TEXT red (425:1312), focused/filled text = primary, Empty row at 80% opacity. Figma `State=Placeholder` ↔ code `Empty` — coordinated rename pending. **Input NEVER grows a label** (Maor 2026-08-10) |
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

