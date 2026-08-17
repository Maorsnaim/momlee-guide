# Screen naming — no sequence numbers

> Decided by Maor, 2026-08-17. Executed the same day across all 62 numbered
> screens in Momlee 2.0 (`↳ Mobile`). This file is both the convention and the
> Rosetta stone for every older document, screenshot or message that still says
> "06.5".

## The convention

**A screen frame's name is its semantic path. Nothing else.**

```
Flow / Actor / Screen / State
Onboarding/Mom/MomLocation/Filled
AccountRecovery/Mom/ViaPhone/OTP/Error
```

- **No sequence number.** Not as a prefix, not as a suffix, not anywhere.
- **Order is not part of identity.** Order lives in the canvas position and in
  the Notion Story that delivers the screen. Grouping lives in the Figma
  `Section`.
- **The name maps 1:1 onto the Notion `Key`**: replace `/` with `.` and
  lowercase to `snake_case`. `Onboarding/Mom/Phone/Error` ↔
  `onboarding.phone.error`. This is the point of the convention: the Figma
  layer name and the OS key stop being two independent facts that can drift.

## Why the numbers were removed

Not preference. They had already broken, in exactly the way numbering always
breaks: **a number encodes position, and position is the most volatile property
a screen has.** The moment Sivan asked for a new mandatory-email screen in the
middle of onboarding, renumbering everything downstream was unthinkable, so the
number stopped being renumbered.

Measured state before the cleanup, across 62 numbered frames:

- **12 number slots were claimed by more than one screen.** `02.1` meant
  `Onboarding/Mom/Phone/Default` *and* `Login/Mom/Phone/Default` *and*
  `AccountRecovery/Mom/ViaPhone/OTP/Default`. `07.2` meant four different
  screens (`Email/Focused/Typing`, `Email/Focused/Empty`, `Email/Filled`,
  `Email/MarketingConsent/Checked`) — the email insert, absorbed by stacking.
- **"Build 02.1" was therefore not a resolvable instruction.** The number had
  stopped identifying a screen, which is worse than having no number, because a
  signal that lies costs more than a signal that is absent.
- **There was never one sequence.** `AccountRecovery` numbered itself `1.1`,
  `2.1` with a single digit while `Onboarding` used `01.1`, `02.1`. Two parallel
  series sharing one namespace, guaranteed to collide.
- **Nothing depended on them.** One markdown line in this repo referenced a
  numbered screen name (and referenced it *wrongly* — see below). Node IDs had
  106 references. The Notion Design Screens DB had zero numbered names and zero
  numbered keys; it already separated `Key` from `Figma Frame ID`.

The single documentation reference said `09.6_Onboarding/Mom/Verification/Pending`.
The frame was actually `10.6`. `09.6` was `Children/Twins`. **The one place that
used a number used the wrong one** — which is the whole argument in miniature.

## Why the rename was safe

**Figma node IDs do not change when a node is renamed, and annotations attach to
the node, not to the name.** All 106 node-ID references kept resolving, and 31
annotations across 29 frames were verified intact after the pass. Nothing needed
fixing downstream except the one wrong markdown line.

## The map (old number → current name)

Ordered as the numbers used to run. Node ID is the stable identifier; use it.

### Section: `Onboarding / Login`

| Was | Now | Node |
|---|---|---|
| `01.1` | `Onboarding/Shared/Mom-Welcome/Default` | `114:9149` |
| `02.1` | `Onboarding/Mom/Phone/Default` | `425:4871` |
| `02.2` | `Onboarding/Mom/Phone/Focused` | `464:4940` |
| `02.3` | `Onboarding/Mom/Phone/Focused/Filled` | `463:3359` |
| `02.4` | `Onboarding/Mom/Phone/Filled` | `439:1969` |
| `02.5` | `Onboarding/Mom/Phone/Error` | `257:8334` |
| `03.1` | `Onboarding/Mom/OTP/Default` | `425:4999` |
| `03.2` | `Onboarding/Mom/OTP/Filled` | `439:2226` |
| `03.3` | `Onboarding/Mom/OTP/Error` | `953:35196` |
| `04.1` | `Onboarding/Mom/FullName/Default` | `439:1338` |
| `04.2` | `Onboarding/Mom/FullName/FirstName/Focused` | `439:2551` |
| `04.3` | `Onboarding/Mom/FullName/FirstName/Filled` | `439:2349` |
| `04.4` | `Onboarding/Mom/FullName/LastName/Focused` | `470:3283` |
| `04.5` | `Onboarding/Mom/FullName/Filled` | `439:2439` |
| `05.1` | `Onboarding/Mom/MomBirthday/Default` | `439:2912` |
| `05.2` | `Onboarding/Mom/MomBirthday/Day/Focused` | `441:1956` |
| `05.3` | `Onboarding/Mom/MomBirthday/Day/Filled` | `441:2489` |
| `05.4` | `Onboarding/Mom/MomBirthday/Month/Filled` | `441:3090` |
| `05.5` | `Onboarding/Mom/MomBirthday/Year/Filled` | `441:3927` |
| `06.1` | `Onboarding/Mom/MomLocation/Default` | `542:5245` |
| `06.2` | `Onboarding/Mom/MomLocation/Focused/Loading` | `574:15466` |
| `06.3` | `Onboarding/Mom/MomLocation/Focused/Results` | `574:15185` |
| `06.4` | `Onboarding/Mom/MomLocation/Focused/NoResults` | `590:18903` |
| `06.5` | `Onboarding/Mom/MomLocation/Filled` | `574:15965` |
| `06.5.1` | `Onboarding/Mom/MomLocation/Unavailable/Subscribe/Placeholder` | `807:23062` |
| `06.5.2` | `Onboarding/Mom/MomLocation/Unavailable/Subscribe/Filled` | `818:26250` |
| `06.5.3` | `Onboarding/Mom/MomLocation/Unavailable/Subscribe/Success` | `818:26359` |
| `07.1` | `Onboarding/Mom/Email/Default` | `1084:10183` |
| `07.2` | `Onboarding/Mom/Email/Focused/Typing` | `1084:10352` |
| `07.2` | `Onboarding/Mom/Email/Focused/Empty` | `1084:10458` |
| `07.2` | `Onboarding/Mom/Email/Filled` | `1084:10539` |
| `07.2` | `Onboarding/Mom/Email/MarketingConsent/Checked` | `1084:10616` |
| `08.1` | `Onboarding/Mom/Status/Default` | `571:14325` |
| `08.2` | `Onboarding/Mom/Status/HasChildren` | `602:19287` |
| `08.3` | `Onboarding/Mom/Status/FirstPregnancy` | `602:19576` |
| `08.4` | `Onboarding/Mom/Status/NoChildren` | `602:19664` |
| `08.5` | `Onboarding/Mom/Status/PreferNotToSay` | `602:19741` |
| `09.1` | `Onboarding/Mom/Children/Empty` | `453:7334` |
| `09.2` | `Onboarding/Mom/Children/OneChild` | `474:7641` |
| `09.3` | `Onboarding/Mom/Children/MultipleChildren` | `508:18823` |
| `09.4` | `Onboarding/Mom/Children/WithPregnancy` | `648:7365` |
| `09.5` | `Onboarding/Mom/Children/AfterEdit` | `740:22209` |
| `09.6` | `Onboarding/Mom/Children/Twins` | `864:11403` |
| `10.1` | `Onboarding/Mom/Verification/Start` | `511:13588` |
| `10.2` | `Onboarding/Mom/Verification/Success` | `740:21751` |
| `10.3` | `Onboarding/Mom/Verification/SecondTry` | `802:22788` |
| `10.4` | `Onboarding/Mom/Verification/Failed` | `740:21823` |
| `10.5` | `Onboarding/Mom/Verification/Error` | `802:22906` |
| `10.6` | `Onboarding/Mom/Verification/Pending` | `858:9397` |

### Section: `Account Recovery`

| Was | Now | Node |
|---|---|---|
| `1.1` | `AccountRecovery/Mom/AccountRecovery/RadioButton/ViaPhone` | `961:35503` |
| `1.2` | `AccountRecovery/RadioButton/ViaEmail` | `1031:37754` |
| `2.1` | `AccountRecovery/Mom/ViaPhone/OTP/Default` | `967:36498` |
| `2.2` | `AccountRecovery/Mom/ViaPhone/OTP/Filled` | `1002:37096` |
| `2.3` | `AccountRecovery/Mom/ViaPhone/OTP/Error` | `968:36591` |
| `3.1` | `AccountRecovery/Mom/ViaEmail/OTP` | `967:36406` |
| `3.2` | `AccountRecovery/Mom/ViaEmail/OTP/Filled` | `1002:37191` |
| `3.3` | `AccountRecovery/Mom/ViaEmail/OTP/Error` | `961:36288` |
| `4.1` | `AccountRecovery/Mom/ManualTeamCheck/RequestSent` | `968:36670` |
| `5.1` | `AccountRecovery/Mom/MainPhoneSelection` | `976:36894` |
| `6.1` | `AccountRecovery/Mom/FinalizingRecovery` | `1031:37850` |

### Outside any section

| Was | Now | Node |
|---|---|---|
| `01.1` | `Home/Mom/FirstTimeLogin` | `950:16514` |
| `02.1` | `Login/Mom/Phone/Default` | `1084:9884` |

These two are top-level frames on `↳ Mobile` with no enclosing `Section`.
`Login/Mom/Phone/Default` belongs inside `Onboarding / Login`;
`Home/Mom/FirstTimeLogin` has no section yet. **Left for Maor** — moving a frame
into a section is a canvas-position change, not a rename.

## Known naming debt this exposed (not fixed here)

Renaming made three pre-existing oddities visible. None were introduced by the
cleanup and none were touched:

1. **`AccountRecovery/Mom/AccountRecovery/RadioButton/ViaPhone`** repeats
   `AccountRecovery`, and its sibling **`AccountRecovery/RadioButton/ViaEmail`**
   drops the `Mom` segment. The two entry-point screens should share one shape.
2. **`Onboarding/Mom/Email/Focused/Empty` vs `.../Focused/Typing`** — the state
   axis is doing two jobs. Compare the `Forms/Input` component, where `Placeholder`
   and `Empty` are separate axes.
3. **`Onboarding/Mom/Verification/Start`** was `10.1` and is the frame previously
   flagged for renaming from `MomBirthday/Placeholder`. Name is now correct;
   confirm the stray DOB annotation on it is gone.
