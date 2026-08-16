# From Sivan → Maor

> Two-way channel. Sivan: add tasks/questions/updates for Maor here (commit +
> push — you have write access), or log them in Notion (see skill
> `momlee-worklog`). Maor reads this on every pull and clears handled items.

## Tasks for Maor

### 🚨 2026-08-16 — URGENT BLOCKER: we need the Variable COLLECTIONS (the Local-variables tables) to finish the token basics + full ❖ Variables scan findings

The token basics are being closed against Momlee 2.0 (Sivan pasted every ❖
Variables sub-page). Typography is fully synced from the sheet; the asset pages
are inventoried (registry on the Web Design System Foundation Feature page in
the OS). **One thing blocks completion:**

- [ ] **The color/spacing/radius/width definitions are nowhere on the canvas.**
  The ↳ Colors page contains raw swatches (no variable bindings, auto layer
  names), and there are no sheets at all for spacing/radius/widths — so the
  ONLY source of the name→value tables is the **Local variables window**
  (the `{}` / left-rail "Variables" button — a popup, not a page). Please open
  it and, for EVERY collection listed on the popup's left side, send a
  screenshot of the full table (scroll so every row's name + value is visible)
  — via Sivan or any channel. Until these arrive the code carries flagged
  stand-in values for: input Focused/Disabled colors, shadows/focus-ring, and
  the outer spacing/radius steps. **This is the current blocker for the whole
  build.**

**Findings from the scan — design hygiene, fix at your convenience:**

1. **↳ Typography (22-98):** some display specimens are still bound to
   Noto Sans Hebrew; the Display xl styles carry a RAW `-2px` letterSpacing
   (off-token); `font-size/text-2xs` (10px) is paired with
   `line-height/text-xxs` = **8px** (smaller than the font — intended? also
   note the 2xs/xxs naming mismatch); one specimen binds `display-2xl → 60px`.
2. **↳ Icons (91-299):** 8 glyphs lack the `lucide/` prefix (coffee,
   footprints, waves-ladder, ice-cream-cone, utensils-crossed, dumbbell,
   loader, mail); `lucide/heart` exists TWICE (92:480 and 199:5682); only
   `chevrons-left` exists (no right/up) — fine, mirroring is code-side, just
   noting.
3. **↳ Avatars (8-190):** the property name "Placholder text" is misspelled
   (we will never copy it into code); `Avatar Add Button` md/lg states are
   unnamed defaults ("State5"–"State12"); `User Avatar` sizes skip `xl`
   (2xs…lg, then 2xl); the `Logo` wordmark skips `sm` (xs→md).

### 🧩 (previous entries below)

### 🎨 2026-08-13 — Token sync DONE from Momlee 2.0 + three findings from the scan (Sivan approved raising all of them)

The full token re-sync you green-lit yesterday is built: `@momlee/tokens` now
mirrors the 2.0 system (brand `#9d636b`, the new surface/border/text
semantics, letter-space + width groups) and web flipped to **Google Sans**
(self-hosted variable woff2, Hebrew+Latin subsets). The updated snapshot is in
this same push — `design-system/tokens.md` + CHANGELOG 2026-08-13. The scan
surfaced three things for you:

- [ ] **`colors/text/text-on-unread` is the on-brand label color** — it's the
  white label on every brand-solid CTA, but the name reads like a
  notifications token. Please confirm it's intentional or rename it before
  the component build bakes it into code and components.md.
- [ ] **Raw (untokenized) values inside 2.0 components:** button vertical
  padding `10px` (off-scale, on every button), `gap: 10px/12px` literals, the
  welcome signup link `#2f67a6`, the welcome headline spans hardcoding
  `#9d636b`/`#b37a81`/black, inline links in running text hardcoding
  `#835258` (= badge/brand/foreground — is that the intended link token?),
  and the keyboard overlay `#e6e9ed`. Tokenize when convenient; code follows
  whatever you name.
- [ ] **A few groups were not visible on any screen and are carried over from
  the old file, flagged in tokens.md:** input Focused/Disabled borders
  (`#404040`/`#d4d4d4`), the disabled back-arrow `fg-quaternary #a3a3a3`,
  link-color button `#6a393c`, `utility.success #16a34a`, shadow/focus-ring
  variables, display-xs/md/lg/2xl px pairs, and the outer spacing/radius
  steps (5xl/6xl/9xl-11xl, radius none-sm/3xl/4xl). **Sharing the ❖ Variables
  sub-page node-ids would close all of these in one pull** — or just confirm
  the carried values.

✅ **Recorded, no action:** the "Continue with Google" button on the welcome
screen — you already removed it (confirmed via Sivan today). Phone-OTP-only
stands.

### 🧩 2026-08-12 — Please announce every NEW UI component through the plugin (and publish the library)

The Web Design System Foundation work started today: all Momlee 2.0 tokens go
into the web CSS and every component you created gets a code implementation,
before any onboarding/verification screen is built. The inventory pass surfaced
a channel gap Sivan asked me to raise with you:

- [ ] **When you create a new UI component, update us through the plugin** — a
  line in `design-system/components.md` (or this file's counterpart,
  open-tasks) with the component's name and Figma node-id. Code sessions can
  only build what an official channel names; a component that exists only
  inside the Figma file is invisible to the build until someone stumbles on it.
- [x] ~~**Please also PUBLISH the Momlee 2.0 library after component work.**~~
  **HANDLED same day (2026-08-12):** you republished at 09:16 and
  `Forms / Field` (+ refreshed Phone Field / Address Search Field) is now in
  the published index — verified from the code side; no node-id needed from
  you anymore. The standing ask above (announce new components + publish
  after component work) remains the ongoing protocol.

### 🚩 2026-08-12 — Sivan's ruling: no standing kill-switch flags; please remove `identity_verification_enabled` from the plugin

- [ ] The Feature-Flags gate (2026-06-11) lists `identity_verification_enabled`
  as a flag every session should expect to exist. **Sivan decided (2026-08-12)
  it will NOT be built, and removed it.** Her rule: MomLee feature flags exist
  to switch a live user-facing experience between two implementations during a
  real test — not as standing kill switches; a flag with no experiment behind
  it is unneeded code. Please remove `identity_verification_enabled` (and the
  kill-switch framing around it) from the plugin's gate lists and skills so
  Claude sessions stop proposing it. The decision is recorded on the IDV-9
  task in the OS.

### 📏 2026-08-11 — Product Rule change (Sivan): family limit is 12 children, not 8

`onboarding.family_entry_limits` currently says up to **8** children. Sivan's
decision: raise the cap to **12**. Everything else in the rule stands (exactly
one active pregnancy, the add action disables at the cap, a soft delete frees a
slot). Please update the Product Rule in the OS so the build reads the right
number.

### 🎨 2026-08-11 — verification is fully tested, and it changed THREE things in your screens

Identity verification is now built and tested end to end against staging — every
decision path, plus the admin approve / reject / reset flows. Thank you for the
before-and-after screens; the brief was exactly right and the build task is
unblocked because of it.

Testing it surfaced three things that touch your designs. **One is a correction
to a screen you already delivered**, so please read that one first.

---

**1. ⚠️ The "Under review" state promises an email we CANNOT send.**

Your return-screen brief says, for *Under review*: *"A person will look at it and
we will email her."*

**We have no email address for her.** Signup is phone-OTP only, onboarding never
asks for an email, and there is no SMS channel either. So that sentence promises
something that will never arrive, and a mom who waits for it simply never comes
back.

**How she actually finds out (Sivan's decision, 2026-08-11):** she is told the
next time she logs in. So the copy needs to point her back to the app rather than
to her inbox — something closer to *"we are looking at it; open MomLee again
shortly and you will see the answer here."* Your wording, not ours.

---

**2. The "Not confirmed" state now has to show a REASON written by an admin.**

When a person rejects a verification, they must give a reason of at least 20
characters, and that text is shown to the mom. So the not-confirmed state needs
to display a **variable-length Hebrew sentence** written by a human — for example
*"בסריקה לא רואים היטב את מין המשתמשת"*.

Please design that state with room for real text, not a fixed one-liner. The
principle you already set still holds and matters more here: it must read as
*"let's try another way"*, never as an accusation.

---

**3. 🆕 A NEW SCREEN is needed — she already has an account.**

Notion task (yours, with the full brief):
**[DESIGN (Maor): the DUPLICATE-ACCOUNT verification screen](https://app.notion.com/p/3b7450ad0ae68159a562c71957f55775)**

Verification compares her face against faces that verified before. If it matches
an account that is **still live**, we do not create a second account for the same
person. Almost always this is simply a mom who forgot she already signed up, or
signed up with a different phone number.

**Why it cannot be the ordinary not-approved screen:** the way back in is *"log
in to the account you already have"*, not *"try verifying again"*. Sent to the
generic error screen she retries against a wall until both attempts are gone.

**What the screen has to work with:** Sivan decided we show a **masked hint** at
the existing account — the backend already produces it, e.g. `••••0902`, or
`n•••@momlee.app` where there is no phone. Never the full number: what the vendor
tells us is that a *face* matched, not that it is provably the same person, so a
full identifier would hand one woman another woman's contact details.

⛔ **The one hard constraint:** this screen must render **only when a hint
exists**. A face matching a *blocked* account produces no hint at all, on purpose
— nothing in the product ever tells a mom she is barred, and a screen that
appeared only for blocked matches would make its own appearance the giveaway.
That case falls back to the ordinary not-approved screen.

---

**4. ⚠️ PLEASE AUDIT YOUR EXISTING SCREENS AGAINST THE NEW LOGIN ROUTING.**

New task: **[Login routing: send a returning mom to the right screen for her
status](https://app.notion.com/p/3b9450ad0ae68172b902dc7605fa96be)**.

**What changed conceptually:** until now the verification screens were designed
for a mom moving forwards through signup in one sitting. From now on, **every one
of them is also a screen a RETURNING mom can land on directly at login**, possibly
days later, with no memory of where she stopped. That is a different state of mind
and it affects copy more than layout.

Here is every state, the screen it needs, and what we think has to change. Please
confirm or push back on each — you own these calls, we are only flagging what the
build exposed.

| At login she is… | Screen | Status | What likely needs to change |
| --- | --- | --- | --- |
| **Rejected**, attempts left | Not confirmed | ✅ exists | Must show the admin's **reason** (see item 2) |
| **Rejected**, 2/2 attempts used | Not confirmed | ⚠️ **variant missing** | See below — this one is important |
| **Mid-onboarding** | Her onboarding steps | ✅ exist | Needs a "welcome back, carry on" moment + steps that render **pre-filled** |
| **Finished onboarding, not verified** | Intro / "have your ID ready" | ✅ exists | Written as a first-time screen mid-signup; it is now also a **returning** screen |
| **Under review** | Under review | ✅ exists | Copy correction — no email (item 1) |
| **Duplicate account** | — | 🆕 **new** | Separate task, item 3 |
| **Approved** | — | n/a | Straight into the app, no screen |

**⚠️ The one we think is genuinely missing: rejected with NO attempts left.**

Your brief says the not-confirmed state is *"never a dead end — she can try
again"*, and that she has 2 attempts in total. But once both are used **there is
no try-again**, and only an admin can give her attempts back. If that screen still
offers "try again" it becomes exactly the dead end you designed against — a button
that cannot work.

So the not-confirmed state probably needs **two variants**: one that offers
another attempt, and one for a mom who has none left, which should give her a real
way to reach a human rather than a disabled button. Your call on how, but she must
not be stranded.

**On the resume case:** if you think "continue where you left off" deserves its own
frame rather than dropping her straight onto the step she stopped at, say so —
otherwise we will reuse the existing steps as they are.

---

### ✅ 2026-08-02 — family-status: YOUR Figma wins. And the Figma move is done, thank you

**Family status — decided in your favour.** The onboarding family-status frames
carry four options (has children / first pregnancy / no children / prefers not
to say) which are **not** the four agreed on the 2026-07-28 call (בהריון /
בחופשת לידה / אמא מנוסה / אחר). Sivan's call: **go with the four in your
Figma.** The proposed enum keys from that call (`pregnant | maternity_leave |
experienced | undisclosed`) are dead and will not be used; the database values
will be named from your options, and the Hebrew wording will be taken
word-for-word from the frames.

One request: **please confirm the four labels are final** before the step is
built, since they become a fixed list every mom's answer is stored against and
changing it after real moms have answered is expensive.

**Figma — thank you, and one thing changed.** Momlee 2.0 now lives on **MomLee's
own Figma Professional plan** (Sivan's team, with a full seat for you). The
reason was practical: her View seat capped automated design reads and blocked a
build session mid-task. The other reason is simply that MomLee's design system
should sit on MomLee's account rather than a personal one — nothing to do with
trust, everything to do with it being a company asset. Verified after the move:
the Momlee 2.0 library still resolves and nothing broke. **If you spot anything
odd with library publishing or linked files, flag it** — you would notice before
we would.

**Also aligned (Sivan, 2026-08-02):** the MVP web app is a **mobile PWA**, and
the `Mobile` pages ARE the product — not a phone variant of a desktop app. The
`Web` page currently holds two desktop meetup-list explorations and no
onboarding, which is consistent with that. A mom who opens MomLee on a laptop
will get a **message telling her to open it on her phone**, so onboarding needs
no desktop layouts. That screen's copy is a real brand surface, not an error
page — it will need your words when we get to it.

### 🚨 2026-08-02 — READ BEFORE YOU CONTINUE THE PRO-MEETUP SCREENS: the RSVP states are settled, and "Interested" is not one of them

Maor: Sivan settled the meetup registration state model today, and it changes
the component you are drawing right now. Sending it immediately for that reason.
**Full model, authoritative:** the *Meetup Registration* feature page, section
**STATE MODEL SETTLED** —
https://app.notion.com/p/38d450ad0ae681fd8688e21976de3910

**1. The heart is NOT a registration state — it never holds a spot.**
This is the answer to the mismatch you flagged on 2026-07-28. "Interested" is
the **heart**, stored in `favorites`, on both meetup types. It is an
**independent control that sits alongside the RSVP control, not a value of
it**. It never registers her, never reserves a place, never counts toward
capacity. She can heart a meetup she is registered to, and heart one she never
joins. "X are interested" is a count of hearts, separate from the attendee
count. So `Status = Idle / Interested / Going` on the RSVP component is wrong on
both counts: Interested does not belong there, and Idle/Going is far too few.

**2. One word: Meetup.** `baby_meetup` is dead as a term (the table was renamed
on 2026-07-20). Two **types**: **pro** (admin-published, paid via Paybox) and
**mom** (mom-created, free). They behave differently and need different controls.

**3. PRO meetup — she asks, an admin approves.** She sends a registration
request; she pays through Paybox outside the app; an admin checks the payment
manually and approves her. **Only an admin can confirm a spot.** States the
control needs:

- Idle — "בקשי להצטרף"
- **Request sent, awaiting approval** (new — she has asked, no spot held yet)
- Registered / going
- **Cancellation requested, refund in progress** (new — see below)
- Cancelled
- **Rejected** (new — an admin declined; she is told why by email)
- Meetup full
- Meetup cancelled / ended

**4. The refund case, because it drives a state you do not have.** If she
cancels **before** approval it is immediate and final, no money moved. If she
cancels **after** being approved, **her spot frees immediately** — she is not
coming, so someone else can take it — while the money is chased separately: the
admin asks the provider to refund her, confirms in Paybox that it happened, and
closes it. So there is a period where she is already out of the meetup but her
refund is still open. Registration and payment are two independent tracks, and
the UI should not imply the seat is held while the refund runs.

**5. MOM meetup — no approval, no money.** She taps "הצטרפי" and is **in
immediately**, auto-approved, no request step and no admin. She can leave freely
at any time. Plus the heart, same as pro. Also decided today: **the mom who
creates a mom-meetup sets her own participant limit**, so the creation form
needs a capacity field and the join control needs a full state.

**6. Rejection is explained, and never looks like a ban.** When an admin
rejects, she types a reason in a reject popup and that reason is **emailed to
the mom**. A rejected mom **may ask again**, and Sivan's rule is that she is
**never shown that she is blocked** — no blocked state, no locked-out screen,
nothing punitive. After a rejection the UI returns to a normal, requestable
state. If the reason still stands, the admin simply rejects again.

Three of these states need database work that does not exist yet
(`rejected`, `refund_pending`, `refunded` + the stored rejection reason) — a
migration task is open on Sivan's side. Nothing blocks your design.

**Still open on Sivan's side, not yours:** what happens when the creator of a
mom-meetup wants out of her own meetup. She had decided this previously but it
was never written down and could not be found, so she is re-deciding it. It may
add a state to the mom-meetup screens later.

### 🗺️ 2026-08-02 — Mapbox: Sivan sent you her own credentials by WhatsApp

Mapbox turns out to have **no member invites at all** — an account is a single
shared login, with no user list, no per-user permissions and no sub-accounts.
The only native multi-user mechanism is SAML SSO, which needs a corporate
identity provider (Okta / Azure AD / OneLogin); MomLee runs none, so it was
never an option. There was nothing to "invite you to".

**So Sivan has sent you her own Mapbox credentials over WhatsApp.** You are in —
go ahead with the map design (style, markers, pins, popups) against the design
system. Please keep the credentials in a password manager rather than in the
chat thread, and note that this login also reaches billing and the production
tokens, so treat the account as production.

### 🧭 2026-07-31 — NEW Feature + Story in the OS: MomLee is women-only, and it is now written down

Maor: a decision of Sivan's needed a home in the model and did not have one, so
there is a new Feature and Story. Flagging it because it touches the OS
structure you built — the shape follows your conventions, but you should know it
exists.

**New Feature — Platform Policies & Terms** (`trust_safety.platform_policies`,
Epic: Trust & Safety, Scope MVP) —
https://app.notion.com/p/3ae450ad0ae681969842de2c9778f0ec
The written rules governing who may use MomLee: terms of use, privacy policy,
and the positions behind them. It did not exist, and the women-only rule was
consequently living only inside a verification flow.

**New Story — MomLee is open to women only, declared in the terms of use**
(`…women_only_declaration`, Access Behavior, Critical) —
https://app.notion.com/p/3ae450ad0ae6810a9a4bd1ac084584d8

**The decision (Sivan, 2026-07-31):** MomLee launches women-only and says so
openly in the terms of use, reserving the right to admit men later. It is
enforced by identity verification that reads sex from an identity **document**,
not from a face — a deliberate choice, because face-based gender guessing fails
most often on women with short or covered hair, which is a large part of our
audience.

**Why it was written down at all:** until now the rule existed only as a
verification setting, which meant "who is MomLee for" would have been answered
implicitly by a threshold and by whoever staffed a review queue. It now needs an
Israeli lawyer's review before launch.

**What changes for you:** nothing immediately. Your two design tasks are
unaffected. But the verification flow is now **ID scan + selfie**, not selfie
only — that change is already reflected on your verification-screens task, and
its biggest consequence is that the intro screen must make "have your ID ready"
impossible to miss.


### 🎨 2026-07-30 — TWO DESIGN TASKS ARE ASSIGNED TO YOU IN NOTION (both blocking)

Maor: two sets of screens are the only thing standing between MomLee and a
working mom onboarding. Everything behind them is built, tested and — for
login — already live. Full briefs are on the Notion tasks; both are assigned to
you and marked High.

**1. Phone-OTP login screens** (top blocker) —
https://app.notion.com/p/3ad450ad0ae6817bbddfce07e6e536fa
Two screens: enter phone number, then enter the 6-digit code. The web login is
now **phone + SMS code ONLY** — no Google, no Apple, no Facebook (Sivan,
2026-07-28; offering any social login would oblige us to build Sign in with
Apple for native). The native phone screens are the reference; the web version
is the same flow minus the social buttons. The whole server side shipped in
PR #12 and works on staging.

**2. Identity-verification screens** —
https://app.notion.com/p/3ad450ad0ae681d1b618ca2df1c59b5e
Two screens: an intro explaining why we ask for a selfie, and the screen she
returns to afterwards. **The return screen needs FIVE states**, because the
result usually is not known the moment she comes back — it arrives a moment
later, server to server. The two that get skipped and matter most here are the
**waiting** state (without it the page reads as broken) and the
**not-confirmed** state (which must never read as an accusation — an automated
check misreading a real mom must feel like *"let's try another way"*, not
*"you failed"*).

Sivan is handling the verification-flow branding on Didit's side separately —
you two have already spoken about it.


### 📋 2026-07-29 — how to log into STAGING + phone-OTP is waiting on your Figma

**1. Logging into STAGING** (`MomLee Staging` Supabase project; the Vercel
Preview deployments point at it). Two ways in:

- **Email + password:** `admin@momlee.test` / `TestAdmin123!`
- **Phone code:** number `972528547424`, code `123456` — no SMS is sent, it is a
  registered test number, so it costs nothing and works every time.

Both land on the same seeded admin account (`אדמין בדיקה`), which has the admin
and parent roles. Staging holds only seed data, so nothing there is precious.

**2. Production has its own test number for you** — Sivan will send you the code
separately. It is deliberately NOT written here: a phone number plus a fixed
code is a working login to production, and anything committed to this repo is in
its history permanently.

**3. ⛔ PHONE-OTP LOGIN IS BLOCKED ON YOU.** The whole server side is built,
tested and live (PR #12): requesting a code, verifying it, creating the account,
Israeli number handling, error messages, abuse limits — all proven end to end on
local and staging. **The only thing missing is the two screens** — phone entry
and code verification.

Sivan understands you are reworking the onboarding screens. When they are ready,
please deliver those two: the screens are thin, they just call the logic that
already exists, so this should turn around quickly.

Context you may not have: the web MVP is **phone-OTP only** — no Google, no
Apple, no Facebook (Sivan, 2026-07-28). The deciding factor was that offering
any third-party social login obligates us to also build Sign in with Apple for
the native app. So the login screen is one phone field and one code field,
nothing else.


### ⚠️ 2026-07-29 — `check-web-arch.mjs` counts the WORD "supabase", including in comments

Second gate finding this week (the RTL one is below). `check-web-arch.mjs` line 43
flags a file when the **raw source text** matches `/supabase/i` — so a file that
merely *mentions* Supabase in a **comment** counts as "touching Supabase outside
the data layer".

**How it surfaced.** A new hook (`hooks/usePhoneOtp.ts`) that imports **no**
client at all — it goes through a service, exactly as the Architecture Gate
requires — failed the gate at 29/28. The trigger was its own doc comment, which
said the file *deliberately avoids* importing Supabase. Reworded to get CI green,
but the gate was rejecting correct architecture for describing itself.

Two consequences worth your call:

1. **The baseline is inflated.** Some of the grandfathered 28 are probably
   comment-only mentions, not real violations, so the ratchet is protecting a
   number that overstates the debt.
2. **It penalises documentation.** The natural way to explain a layering
   decision is to name the thing you are not importing.

**Suggested fix:** match an actual import/usage rather than the bare word — e.g.
`from ['"]@/lib/supabase|from ['"]@momlee/supabase|supabase\.(from|auth|storage|rpc)\(`
— and re-baseline once, expecting the count to DROP. Not changed from our side:
the gate is yours, and re-baselining inside a feature branch would hide the
change.

### ⚠️ 2026-07-28 — `check-rtl-web.mjs` has a BLIND SPOT: responsive-prefixed physical classes

Maor: your RTL web gate does not see **prefixed** physical direction classes.
Its pattern matches bare `text-left`, `ml-`, `pl-` etc., but **not**
`sm:text-left`, `md:pl-4`, `lg:text-right` — so any physical class behind a
breakpoint prefix passes silently, in a gate whose entire purpose is to catch
exactly those.

**How it surfaced (a real RTL bug, not a hypothetical).** Stock shadcn ships
`text-center sm:text-left` on the header of **AlertDialog, Dialog, Sheet and
Drawer**. In an RTL app that pins every dialog **title and description** to the
LEFT at ≥sm while the dialog body stays right-aligned, so the header and the
content visibly disagree. Sivan hit it on the meetup rejection dialog. Fixed in
the app repo (commit `ead8fd1`) by switching all four to `sm:text-start`.

**The count did not move** — still 543 — because those four instances were never
being counted. RTL is release-blocking per the plugin, so a gate that cannot see
half the surface is worth a look.

**Suggested fix:** allow an optional variant prefix in the pattern, roughly
`(?:^|\s|")(?:[a-z0-9-]+:)*(ml-|mr-|pl-|pr-|left-|right-|text-left|text-right)`.
Expect the baseline to jump when you do — that jump is pre-existing debt the
gate was blind to, not new violations. **Not changed from our side**: widening
the gate would move the ratchet baseline inside an unrelated feature branch, and
the gate is yours.

**Also still present, deliberately untouched:** `text-left` on shadcn's
`TableHead` (`components/ui/table.tsx`). Same class of bug, but fixing it
re-aligns table headers across many existing screens — Sivan's call, not a
drive-by.


### ✅ 2026-07-27 — two of your three open questions are ANSWERED (they were Sivan's, not yours) + Plan C scope narrowed

Maor: of the three decisions you logged as pending on 2026-06-11, **two were not
UI/UX and therefore Sivan's to make** per the ownership split. She has decided
both. Only the third is still yours.

**1. Feature-flag kill-switch — DECIDED (Sivan).** Pre-launch, feature flags are
**Vercel environment variables** (kill = flip the var + redeploy, about a
minute; acceptable with no users and no revenue at risk, and zero infra to
maintain). **Post-launch, they move to a `feature_flags` table read at runtime**
so a broken or abused feature can be killed instantly without a deploy. Pulled
forward if payments or identity verification ship first. Your Feature Flags gate
(kill = flip the flag, never delete code) is satisfied by the env-var mechanism
meanwhile. Recorded in the app repo's `LAUNCH_CHECKLIST.md` (new Post-launch
section) and parked as a Notion task with its trigger.

**2. Account-deletion cascade map — DECIDED (Sivan).** This is what you need for
the **retention column in `knowledge/data-inventory.md`**. When a mom deletes
her account:

| Data | Verdict |
|---|---|
| Profile PII (name, phone, email, address, avatar) | DELETE |
| Children's records | DELETE |
| Her meetup registrations | DELETE |
| Favorites | DELETE |
| Meetups **she created** that other moms joined | **ANONYMIZE** — deleting would silently cancel other people's plans |
| Chat messages | **ANONYMIZE** — deleting shreds the other side of the conversation |
| `public.audit_log` | **RETAIN** — PII-free by design, it is the compliance record |
| Identity-verification result | DELETE (store-result-only; the selfie never lived on our servers) |

Rules: soft delete stays the default mechanism; *anonymize* means a tombstone
identity, never nulling an FK others depend on; every deletion path writes to
`audit_log`. **Please apply this to the data-inventory retention column** — it
gates the privacy policy and the store privacy labels.

**3. Shared Empty/Error state designs — STILL YOURS.** Unchanged, still blocking
the four-states component.

**4. ⚠️ PLAN C SCOPE NARROWED — flagged, not silent** (per your 2026-07-21 note
about contradicting a written directive). Your directive's area order was
meetups → organizations → subscriptions modeling → messaging. Sivan has removed
two: **organizations SKIPPED** (2026-07-19, already reported) and now
**subscriptions/entitlements modeling PARKED as POST-MVP** (2026-07-27). Reason:
every table in AREA 15 of the target data model is keyed to `provider_profiles`,
and the MVP has **no provider user** — PRO meetups are admin-published and the
₪99/mo provider fee is billed on a separate platform, not in-app. Modeling four
tables no code can read, on top of the still-open **D6** (entitlements as a table
vs computed via `has_entitlement()`), would recreate exactly the dead-shapes
problem Plan C exists to prevent. Nothing is lost: the full future scope plus the
five open questions to resolve then are parked in a Notion task with an explicit
unpark trigger ("a real paying provider exists"). **The live Plan C order is now
meetups → onboarding** (mom signup / profile-completion). Veto welcome if you
disagree.

> # 🔴 MAOR — ALL WEB UI WORK IS BLOCKED ON YOU (2026-07-15)
>
> **Sivan is a VIEWER on your Figma account — she cannot make any of these calls.
> Every item below needs YOU.** The web primitive foundation is shipped and CI is
> green (AppText · Input · Button(text|icon) · Icon), but **UI work cannot continue
> until these are answered.** Ordered by how much they block:
>
> 1. **⛔ FIGMA PLAN / SEAT — blocks EVERYTHING UI.** Code Connect needs
>    **Organization/Enterprise**; the account is **Professional** and Sivan's seat is
>    **View**. Worse, the View seat has a **Figma MCP call quota** that also throttles
>    `get_design_context` — today's primitives work **exhausted it**, so no design can
>    be pulled at all right now (a new session does NOT reset it; it's server-side per
>    seat). **You own the account — please upgrade the seat/plan, or tell us to drop
>    Code Connect permanently and work within the quota** (screens would then need the
>    instance-map done manually: grep `figma: <node>` + components.md).
> 2. **⛔ `Buttons/Social button` NODE ID — blocks Button `kind='social'`** (the
>    Google/Facebook/Apple login buttons). `search_design_system` returns only a
>    componentKey (`cfa87e645c6c872eef7866b4f5dc367153914e96`); `get_design_context`
>    needs a **nodeId**. Please open the component in Figma and send the URL — it ends
>    in `?node-id=...`. (Sivan can't hunt for it: no quota.)
> 3. **⛔ `components.md` CONTRACT-ROW FORMAT** (your OS task, High) — the web
>    implementations of AppText/Input/Button/Icon still need their rows. **We did NOT
>    invent a format** — it's your call.
> 4. **⚠️ REUSE-GATE RULING** — your `check-component-dupes` Rules A+D contradict your
>    own ADR-016 (see the task below). A temporary `isAdr016PlatformPair()` keeps CI
>    green; please confirm or replace it.
> 5. **⚠️ `@svgr/webpack` DEPENDENCY** — so web icons single-source from your
>    `packages/ui/assets/icons/*.svg` instead of inlined copies (drift risk).
> 6. **📋 Dev Changelog DB** — still missing, so the worklog stays in this file.
> 7. **📋 PLAN C (schema re-baseline)** — still awaiting your decision; it blocks the
>    product-core service work (meetups/favorites/forum).
>
> **Net effect: Sivan is idle on UI.** Items 1 + 2 unblock the most. If you'd rather
> she work on something else meanwhile, say which — the remaining board items are
> provider/admin-side (not MVP) or blocked on Plan C.

- [ ] 2026-07-15 (**Maor + Sivan — business decision: Figma plan/seat. BLOCKS Code
  Connect, and THROTTLES all figma-first work**): **Code Connect needs an
  Organization/Enterprise Figma plan — the account is Professional with a View seat.**
  Next after the primitives was the OS task *"Set up Figma Code Connect mappings for
  the primitives"* (it's the right order — mapped nodes make `get_design_context`
  hand back the existing component for every screen instance = the natural
  re-implementation prevention, before we build screens). It cannot be done:
  - Figma's own `figma-code-connect` skill states: *"Organization or Enterprise plan
    required — Code Connect is not available on Free or Professional plans."*
  - Verified empirically — `get_code_connect_suggestions` returns: *"You've reached
    the Figma MCP tool call limit for your **View seat on the Professional plan**.
    Upgrade your seat or plan for more tool calls."*
  - `whoami`: `Sivan Bittan's team` (tier **starter**), `Webecy` (tier **pro**) —
    **seat = View on both**.
  **Impact beyond this task:** the same View-seat quota throttles `get_design_context`
  — so it limits **all** figma-first UI work (incl. rebuilding the Mom onboarding
  screens), not just Code Connect. Today's primitives work exhausted the quota.
  **Decision needed:** upgrade the seat/plan (cost), or accept: no Code Connect + a
  hard cap on Figma MCP calls per session. If we skip Code Connect permanently, screen
  work must do the instance-map manually (grep `figma: <node>` + components.md per
  momlee-figma-first step 5) — doable, just less safe.
  Notion task marked **Blocked** with this reason.

- [ ] 2026-07-15 (**Maor — need the Figma node id; blocks Button `kind='social'`**):
  **What is the node id of `Buttons/Social button`?** Building the web Button's
  social kind figma-first, `search_design_system` returns only a `componentKey`
  (`cfa87e645c6c872eef7866b4f5dc367153914e96`) — `get_design_context` needs a
  **nodeId**, and `components.md` records the set by name but no node. Rather than
  mirror the native impl blind, I **deferred the social kind** and am asking. Send
  a node-specific URL (`…?node-id=1-2`) and I'll build it. Same for `Buttons/Social
  button group` if it's in scope. *(For reference the nodes I do have and used:
  Button set `3287:427074`, icon-only `3287:428811`, Input `17297:8153`, Icons
  library `3463:407484`.)*
- [ ] 2026-07-15 (**Maor — dependency approval, low priority; a workaround is in
  place**): **`@svgr/webpack` for web, so icons are single-sourced.** Native
  imports the Figma glyphs from `packages/ui/assets/icons/*.svg`; web cannot import
  `.svg` as a component without svgr, so I **inlined the identical Figma paths**
  into the web `Icon` (explicitly NOT lucide — your rule "never substitute
  look-alike glyphs" is respected). The cost is **drift risk**: if you update an
  icon in Figma, native's `.svg` updates and web's inlined copy silently doesn't.
  Proper fix = add `@svgr/webpack` (devDep, web only) so both platforms consume the
  SAME files → DEPENDENCY GATE, your call. Not urgent — 5 glyphs, marked
  "keep in sync" in the file.

- [ ] 2026-07-15 (**Maor — please rule; a TEMPORARY fix is in place, CI is green**):
  **Your reuse gate contradicts your own ADR-016 for every primitive.**
  `check-component-dupes.mjs` Rule **A** (no two files share a base name) and Rule
  **D** (one Figma node = one component file) both assume ONE component per
  name/node. But ADR-016 mandates each primitive be implemented **once per
  platform under the SAME name and the SAME node** — native (`packages/ui/src`) +
  its web twin (`apps/web/src/components/ui/primitives`) — because that pair IS
  the shared contract. So **every** primitive fails CI; `button`/`input` only
  passed **by accident** (they sit in `GRANDFATHERED_DUP_NAMES` for the unrelated
  demo-shadcn dups). `AppText` was the first to hit Rule A; Button + Input then hit
  Rule D on merge (real red CI on `momlee-web`).
  **What I did (temporary, easily reverted — you own the rules):** added
  `isAdr016PlatformPair()` to the gate, recognising EXACTLY that one native + one
  web primitive pair and nothing else. **Verified with negative tests:** a 3rd file
  claiming a node still FAILS; a web-vs-web duplicate name still FAILS. Commit
  `d387289` on `momlee-web`, CI green. **Revert = delete the helper + its two call
  sites.** Please confirm this is the rule you want, or tell me your preferred
  shape and I'll switch to it.
  *(Minor papercut, not blocking: the gates' grandfather lists use forward slashes,
  so on Windows `path.relative` yields backslashes and they misfire LOCALLY —
  pre-existing files look "new". CI (Linux) is fine. Worth normalising paths.)*

> **❌ RETRACTED 2026-07-14 — the previous "✅ ANSWERED" note here was WRONG.**
> An earlier session claimed the Dev Changelog "is NOT deleted, it's alive and
> written to daily", that rows were synced into it, and that Sivan's 404s are an
> ACCESS gap ("your Notion account was never shared into Maor's workspace").
> **Both claims are false — verified four ways this session:**
> 1. `fetch self` → authenticated as **Sivan Bittan (sivan@applee.dev) INSIDE the
>    "Maor Naim's Notion" workspace**; Momlee OS pages read fine and the OS Tasks
>    DB queried successfully. **There is no access gap.**
> 2. The hardcoded id `ee6d4bbb-1444-479c-b818-36f7e3951988` **404s**.
> 3. An API search across the workspace finds **no Dev Changelog database**
>    (Momlee OS = 01–08 + Page Template; 07-Engineering = Principles / Stack /
>    ADRs / Validators only).
> 4. Sivan's Notion UI search finds only **text mentions**, and his **unfiltered
>    Trash is empty** — so it wasn't deleted inside the 30-day recovery window.
>
> **Maor — the real ask (see the task below): does a Dev Changelog DB exist
> anywhere?** If yes → send the URL/id. If no → create it (or tell us which DB
> should receive worklog rows) **and update the id hardcoded in the plugin skills
> + worklog hook** — until then the enforced hook points at a dead id for every
> session/contributor. **Sivan's decision (2026-07-14): he keeps logging to THIS
> file (git) and will migrate into the real Changelog once it exists — deliberately
> NOT creating a second DB, to avoid duplication.**
>
> **✅ FIXED (Claude, 2026-07-14): the hook/skill cache-path bug.** The worklog
> hook message + skill fallback now explicitly require writing to a REAL CLONE
> of momlee-guide (never the plugin cache) + commit+push+verify.

- [ ] 2026-07-14 (**Maor — please decide + re-sync tokens; BLOCKS the web Button**):
  **`bg-brand-solid` token drift — Figma vs the package/snapshot disagree on the
  primary CTA.** Building the first web primitives figma-first, the live Figma
  node `3287:428579` (Buttons/Button · xl/Primary/Default) renders
  `bg-brand-solid = #f0e4e6` (light pink) with label `text-primary-(900) #171717`
  (dark). But your own snapshot `design-system/tokens.md` AND the `@momlee/tokens`
  package AND native Button all say `bg-brand-solid = #b05f64` (mauve) with WHITE
  primary text. So the whole primary treatment differs: **(mauve bg + white text)
  in code vs (light-pink bg + dark text) in live Figma**, and the file is marked
  "not final". Two questions:
  1. **Is the Figma retint final** (`bg-brand-solid` → `#f0e4e6`)? If yes it's a
     **token re-sync** of `@momlee/tokens` (via momlee-sync-tokens) that affects
     **web AND native**, and likely the broader "palette retint".
  2. **Primary button text — white or dark `#171717`?** (pairs with whichever bg
     is canonical.)
  I've **held the web Button** until you rule (per figma-first "never ship a
  guess"). AppText + Input were unaffected — their tokens matched Figma exactly
  (`#a3a3a3`/`#404040`/`#b54141`/`#d4d4d4`) — and are built + verified. Details in
  the worklog below.

- [ ] 2026-07-12 (**Maor — please handle**): **Dev Changelog DB appears GONE.**
  The standup skill's collection id (`ee6d4bbb-1444-479c-b818-36f7e3951988`)
  and the 2026-07-08 changelog row (`397450ad0ae681c7bceee1cfae7414ac`) both
  404, and no search hit surfaces the DB (only a "Decision Log"). The worklog
  can't write to Notion — using this git fallback. Please restore it (Notion
  Trash keeps deleted DBs ~30 days) or point the worklog + standup skills to
  the current Dev Changelog data-source id (the id is hard-coded in the
  momlee-guide plugin, so a brand-new DB needs a plugin update to be picked up
  automatically).
  ⚠️ **Take into account: the LIVE DB was changed this session.** Migration
  `20260712000000` is applied to production — it dropped the 17 provider-group
  (M03.6) `audit_action` enum values and deleted 16 pre-launch audit_log rows
  that used them. When you restore/reconcile the changelog (and any OS
  Schema-Registry / Decision-Log entries that reference provider-group audit
  actions), note those values no longer exist in the DB. Details: commits
  `f6cced7` + `08cfa3c` on `momlee-web`.
- [x] 2026-07-12: **LIVE `audit_log` had 16 provider-group (M03.6) rows** —
  RESOLVED. Inspected: 16 rows across 5 actions, all 2026-06-01..09 (the M03.6
  build window; DB is pre-launch/seed-only). Sivan confirmed disposable, so
  migration `20260712000000` was amended (commit `08cfa3c`) to `DELETE` those 16
  rows first (destructive-approved), then trim the enum. FYI only — no action
  for Maor.
- [ ] 2026-07-14 (**Maor — please fix**): **The worklog HOOK's git fallback
  writes to the plugin CACHE, so entries never reach you.** The momlee-worklog
  hook/skill fallback appends to `<plugin-cache>/planning/from-sivan.md`, but the
  cache dir is NOT a git repo — so weeks of worklog/task entries piled up locally
  and never reached this repo (this is why the entries above only landed now, via
  a manual flush — commit `b9f77e5`). Sivan's `/checkpoint` skill now writes to a
  real clone (`C:\Users\sivan\workspace\momlee-guide`) + commits/pushes, but the
  ENFORCED hook still points at the cache for every session/contributor. Please
  repoint it (write to a cloned repo + commit/push, or another synced location).

## Updates / questions

- [ ] 2026-07-15 (**FYI + one small UI question at the end**): **Identity Verification
  is moving — Sivan answered your "Open Questions (Owner: Sivan)" and we've broken it
  into tasks.** *Why now:* all web UI work is blocked on your Figma decisions (see the
  red block above), so Sivan moved us onto the **backend of the mom onboarding's
  verification step**, which is decidable without you. Nothing was invented — your
  model already covered it (we checked first and did **not** create duplicate
  Features/Stories).

  **Sivan's decisions** (recorded on the Feature
  [Identity Verification](https://app.notion.com/p/38e450ad0ae681e9894beab124a12942),
  new section 11):
  1. **Vendor = Didit** (supersedes **Persona** everywhere in the spec) — a
     backend/integration call, hers.
  2. **Selfie only, NO ID document.** Rationale: moms on maternity leave holding a
     baby won't go find a wallet mid-flow — they'd abandon onboarding.
  3. **Retry = max 2 attempts** (answers your "כמה ניסיונות?").

  **New implementation tasks — IDV-1 … IDV-10** in the Tasks DB, ordered, all linked
  to your story *"Mom completes identity verification after the children step"*:
  [IDV-1 Didit spike](https://app.notion.com/p/39e450ad0ae6811fa57be0563816da56) ·
  [IDV-2 migration](https://app.notion.com/p/39e450ad0ae681e59f70d720aa823f24) ·
  [IDV-3 session+webhook](https://app.notion.com/p/39e450ad0ae6813fb6e4ea672f86a107) ·
  [IDV-4 policy engine](https://app.notion.com/p/39e450ad0ae681d288dbdc6a0b339f89) ·
  [IDV-5 admin queue](https://app.notion.com/p/39e450ad0ae6812f8430fa15d2be1688) ·
  [IDV-6 approval email](https://app.notion.com/p/39e450ad0ae68102b0a9eac3081791ea) ·
  [IDV-7 eligibility/permissions](https://app.notion.com/p/39e450ad0ae6816481ebcae6f7054b64) ·
  [IDV-8 analytics](https://app.notion.com/p/39e450ad0ae6818c9264d50022caafe4) ·
  [IDV-9 privacy/biometric](https://app.notion.com/p/39e450ad0ae681df9b2bfe33918fdef2) ·
  [IDV-10 screens (Blocked)](https://app.notion.com/p/39e450ad0ae681c5997ecd5f5cfac1f7)

  **IDV-1..IDV-9 need no Figma** — that's the work that can proceed. We kept your
  entity's core principle intact: *the vendor result is evidence only; the decision is
  Momlee policy.*

  **✅ The admin-queue UI question is WITHDRAWN — Sivan already ruled it (2026-07-15):**
  the admin dashboard **does not need your design**. It's fully blocked from non-admin
  users, so it isn't part of the designed product surface — we'll reuse the existing
  admin patterns and skip design for the MVP. She says she'd already told you this.
  **Nothing for you on IDV-5.**

- [x] 2026-07-15: **✅ Dev Changelog — verified and synced, thank you.** Your archived-DB
  diagnosis reconciles everything (my "does not exist" was a correct observation with a
  wrong conclusion — archived ≠ deleted, and Notion search doesn't index archived pages;
  my Trash check couldn't see it either). Fetch works on the same id. **Synced 7 pending
  entries as rows** (Noto self-host; primitives Phase 1; primitives Phase 2; the reuse-gate
  fix; Sivan's Didit decisions; the Didit spike result; the IDV-1..10 breakdown).
  **Logging directly to Notion from now on**; this file stays for outages + for things
  that need you.

- [x] 2026-07-15: **✅ Thank you for the ADR-016 ruling + the Windows path fix.** Nothing
  to change on our side; `isAdr016PlatformPair` stays as shipped and the gates now match
  CI locally. Also noted: `Buttons/Social button` node still pending (Claude hit the same
  plan limit server-side — `kind='social'` stays deferred, agreed), and `@svgr` is parked
  behind your bigger icon-library question. **The Figma plan/seat decision is still the
  single biggest blocker on all UI work** (red block at the top).

- [x] 2026-07-14: **Web token-preset wiring is now DONE** (supersedes the
  2026-07-12 "on HOLD" note). `apps/web` declares `@momlee/tokens` + registers
  `presets:[momleePreset]`; token classes (`bg-bg-primary`, `text-text-primary`,
  `gap-xl`, `font-sans`=Noto, …) resolve and emit. **Additive / zero visual
  regression** — the demo shadcn theme is untouched (Heebo font swap intentionally
  left for its own task). CI assertion added to `check-tokens-web.mjs` so the
  wiring can't regress. See worklog below.
- [ ] 2026-07-14 (**Maor — please decide**): **Radius A/B — should the Figma token
  radii win on `rounded-sm/md/lg`?** The preset and the demo's shadcn theme both
  define `sm/md/lg`; on a Tailwind preset merge the **main config wins**, so the
  demo's CSS-var radii keep those keys and the preset's Figma values
  (`radius-md`=8px, `sm`=6px, `lg`=10px) are **shadowed** (verified: `rounded-md`
  still resolves to `calc(var(--radius) - 2px)`). Consequence: a NEW Figma
  component writing `rounded-md` expecting 8px silently gets the demo's ~10px.
  - **Option A (shipped, non-breaking):** demo radii keep winning during the
    demo-coexistence window; new primitives must NOT rely on `rounded-md/sm/lg`
    meaning the Figma value — use non-colliding keys until the demo is retired.
  - **Option B:** pull the demo's `sm/md/lg` radii out of `theme.extend` so the
    token radii win — a **3-line follow-up**, but it changes the corner radius of
    every Radix/shadcn component across all 32 demo pages at once. Your design call.
- [ ] 2026-07-14 (**Maor — please handle**): **[GAP] Validator "flip to Active".**
  The wiring that the Validator tracks is now live, so it can be flipped. The
  status entry lives in the momlee-guide plugin's `knowledge/`/`planning/` channel
  (not the app-repo checkout), so it can't be edited from a repo session — please
  flip it, or tell Sivan where it lives.
- [ ] 2026-07-14 (**Maor — design sign-off**): **Palette retint (separate, future).**
  Retinting the demo shadcn CSS variables onto the MomLee brand tokens (a real
  visible design change per ADR-016) is deferred to you — not part of the wiring
  above, needs your sign-off before anyone ships it.

## Worklog (pending Notion sync)

- 2026-07-31 (`apps/web` + `supabase`, `momlee-web` merge `0f80df7`, PR #13, CI
  GREEN, **2 migrations LIVE on prod** — Sivan + Claude):
  **Mom identity verification — entire server side SHIPPED.** Migrations
  `20260729000000` (verification_application + `parent_profiles.
  is_identity_verified`) and `20260730000000` (webhook idempotency ledger)
  applied to production. Also shipped: the webhook receiver (HMAC signature over
  canonical JSON, constant-time compare, 300s replay window, duplicate rejection
  via an event-id primary key, **fails closed** without the secret), session
  creation on Didit's hosted flow with `language:'he'`, the policy decision
  engine, and the start-verification action. pgTAP 17/17, Vitest 133/133.
  **⛔ The ONLY remaining piece is your two screens** — the intro that tells her
  to have her ID ready, and the return screen with its five states.
  **The flow changed and your task page already reflects it:** verification is
  now **ID scan + liveness + face match**, not selfie-only. Reason: a real Didit
  payload proved a liveness-only workflow returns **no gender at all**. Reading
  sex from a document also removed the need for a second vendor and removed the
  failure mode where face classifiers misread women with short or covered hair —
  postpartum and religious mothers, a large part of our audience.
  **Also live since PR #12:** the whole phone-OTP login server side, which is
  likewise waiting only on your two screens.

- 2026-07-28 (`apps/web` + `supabase`, `momlee-web` merge `4839b03`, PR #11, CI
  GREEN, **migration LIVE on prod** — Sivan + Claude):
  **Meetups area — admin registration handling SHIPPED.** Migration
  `20260727000000` applied to production (verified `Local == Remote`): three
  `audit_action` values + a `sync_meetup_full_status` trigger that keeps
  `meetups.status` honest against `capacity` (`full` existed as a status but
  nothing ever set it). Admin can now approve / reject registrations; on a PRO
  meetup approving also records the Paybox payment in one action ("שולם"),
  since the admin only ever approves because the payment is already visible.
  **Rejection now REQUIRES a reason** — reused your shared
  `RejectionReasonDialog`, recorded in the audit metadata, and emailed to the
  mom via a new template. Pending-registration badges on the admin tab and per
  meetup so nothing waits unnoticed; approve is disabled when full rather than
  clickable-then-failing (server-side capacity check unchanged and still
  authoritative). pgTAP 8/8, Vitest 81/81, all gates green.
  **Plan C status:** meetups admin work is complete; the only remaining meetups
  item is the mom-facing registration UI, which is **Figma-blocked on you**.
  Next area is **onboarding** (subscriptions parked Post-MVP, see above).
  ⚠️ **Ops note worth knowing:** the first post-merge run failed on a transient
  *Gateway Timeout* fetching the Supabase CLI in `rls-tests`; `db-deploy` was
  skipped as a dependent, so merged code briefly sat on `momlee-web` with the
  migration NOT on prod. `gh run rerun --failed` fixed it. A green PR is not a
  green deploy — worth knowing if you ever merge while Sivan is away.

- 2026-07-15 (`apps/web`, `momlee-web` merge `86246d1`, CI GREEN — Sivan + Claude):
  **Phase 2 — Icon primitive + Button `kind='icon'` SHIPPED.** `Icon` (figma
  `3463:407484`): semantic `forward`/`backward` mirrored per layout direction,
  sizes, `currentColor` tint — built on the **same Figma Icons library glyph paths
  as native**; **no lucide** (your "never substitute look-alike glyphs" rule caught
  a planned lucide wrapper before it shipped — thank you, that rule earned its
  keep). `Button` now a discriminated union with `kind:'icon'` (figma
  `3287:428811`): forward = 48 brand-solid CTA, back = outlined, and the **disabled
  back arrow uses the `fg-quaternary` token** (explicit token, not an opacity wash,
  per your 12-15 note). Verified: tsc + next build + gates + visual pass on a
  throwaway preview (deleted). The `isAdr016PlatformPair` gate fix generalised —
  Icon's native/web pair passed with no further change.
  **Open for you:** the `Buttons/Social button` **node id** (blocks `kind:'social'`)
  and the optional `@svgr` dependency — both above. Web primitives now: **AppText ·
  Input · Button(text|icon) · Icon**.

- 2026-07-15 (`apps/web`, `momlee-web` merge `8abf0f6` + gate fix `d387289`, CI
  GREEN — Sivan + Claude): **Phase 1 web primitives SHIPPED — AppText · Input ·
  Button** (OS task `platform.web_design_foundation`). Built figma-first on the
  shared `@momlee/tokens` preset in `apps/web/src/components/ui/primitives/`,
  mirroring the native `@momlee/ui` contract (same name/props/node/tokens, ADR-016);
  RN → DOM+Tailwind. **Thank you for the fast rulings** — the reverted retint
  (`bg-brand-solid` back to `#b05f64` + white label) and radius **B** (`51ed790`)
  both verified flowing through: built CSS shows `bg-bg-brand-solid` =
  `rgb(176 95 100)` and `rounded-md` = `8px`. Verified tsc + next build + token CSS
  + a visual pass on a throwaway preview route (since deleted). Sanctioned
  web/native divergences: AppText weight → CSS font-weight (browsers pick Noto
  weights natively; the per-family rule is an iOS quirk) and Input uses the true
  Figma line-height (the iOS display-xs→md hack doesn't apply). **Next:** Phase 2 =
  Icon + Button `kind:'icon'|'social'`. **Needs you:** the gate ruling above, and
  the components.md contract-row format (I did NOT invent one — the rows still need
  their web implementation noted, pending your format task).

_(entries logged here only when the Notion MCP wasn't available; synced to the
Dev Changelog later)_

> _(2026-07-14: the 07-13/07-14 entry clusters below were SYNCED to the Dev
> Changelog by Claude — tokens-preset wiring `39d450ad...8d14` (In progress,
> awaiting the 6d7d4e1 push) + the consolidated security closeout
> `39d450ad...2735` (Done). The Noto-font entry below lands as its own row.
> Reminder: commits `6d7d4e1` + `6b3bf0d` are still LOCAL on your machine —
> push them so CI runs and the Validator can flip Active.)_

- 2026-07-14 (`apps/web`, branch `sivan/web-primitives`, commit `17cda9b` — Sivan
  + Claude): **First web primitives, figma-first — AppText + Input built + verified;
  Button held.** Per ADR-016 (shared contract, per-platform impl), mirrored native
  `@momlee/ui` as DOM+Tailwind on the token preset, in
  `apps/web/src/components/ui/primitives/`. AppText (RTL text primitive; weight via
  CSS font-weight — Noto weights are native on web) + Input (figma `17297:8153`:
  underline, 5 states, Extras slots; tokens confirmed against Figma). tsc clean,
  next build ok, token classes emit. NOT merged to `momlee-web` (task incomplete —
  Button blocked on the `bg-brand-solid` drift above; throwaway `/primitives-preview`
  harness stays on the branch until merge). Next: Maor's Button ruling → build
  Button → delete preview → merge via handle-next-issue.

- 2026-07-14 (`apps/web`, `momlee-web`, commit `6b3bf0d` — Sivan + Claude):
  **Web is now on the official font — Noto Sans Hebrew, self-hosted** (OS task
  `platform.web_design_foundation`; Heebo removed per ADR-016). Swapped the
  runtime Google Fonts Heebo `@import` for `next/font/google` Noto Sans Hebrew,
  self-hosted at build → **zero runtime external font request**, no layout shift
  (display:swap + size-adjusted fallback). `--font-sans` variable; web
  `tailwind.config` maps `font-sans`→`var(--font-sans)`; `heebo` key removed. The
  shared `@momlee/tokens` literal family is untouched, so **mobile is unaffected**.
  Verified: `tsc` clean, `next build` ok, 4 woff2 self-hosted, no Google Fonts
  refs remain. FYI (visible typography change) — no decision needed from you.
- 2026-07-14 (`apps/web`, `momlee-web` — Sivan + Claude): **Wired the shared
  `@momlee/tokens` preset into the web Tailwind config** (OS task
  `platform.web_design_foundation`, Critical). Declared `@momlee/tokens` as a web
  workspace dep, added `import momleePreset from "@momlee/tokens/preset"` +
  `presets:[momleePreset]` to `apps/web/tailwind.config.ts`, and added a
  non-regressable CI assertion in `check-tokens-web.mjs` (fails if the preset
  import/registration is dropped). **Additive, zero visual regression** — the demo
  shadcn theme stays; new token-namespaced classes (`bg-bg-primary`,
  `text-text-primary`, `gap-xl`, `font-sans`) now emit. Verified end-to-end: pnpm
  symlink present, gate green, a Tailwind compile emits the token classes and
  confirms `rounded-md` is still demo-shadowed (the radius A/B decision above).
  Heebo→Noto font swap intentionally left for its own task. Committed as `6d7d4e1`
  and **pushed** to `momlee-web` (`08cfa3c..dc3ee32`, alongside a `/checkpoint`
  skill tweak `dc3ee32`); `db-deploy` is a no-op (no migration). Follow-up
  `fe25276` fixed a **tsc-only** type error the first push missed (the
  framework-agnostic `as const` preset needed `as unknown as Partial<Config>`;
  runtime was always fine) — CI is now **GREEN** end-to-end (checks ✓ rls-tests ✓
  db-deploy ✓, run 29314383827). Lesson: run `pnpm type-check`, not just a
  Tailwind compile, before pushing a `tailwind.config.ts` change.
- 2026-07-14 (security cleanup CLOSED OUT — Sivan + Claude): **All live
  security-audit cleanup is DONE.** Edge Functions deleted (curl → 404) after
  archiving (commit `8ffe06f`); Supabase test phone numbers configured with a
  "Valid Until" expiry and verified end-to-end via the Auth API (`/otp` → 200
  no-SMS, `/verify` with the fixed code → session). Added `LAUNCH_CHECKLIST.md`
  (app-repo root) with "clear test OTPs before launch". service_role + Resend
  keys marked Sensitive in Vercel (verified never leaked — no rotation). Mapbox +
  Twilio rotated (see prior entries). Remaining are Maor-only: delete the June
  temp Supabase access token + fix the worklog hook (see Tasks for Maor).
- 2026-07-13 (commit `8ffe06f`, `momlee-web` — not yet pushed): **Archived the 7
  legacy Edge Functions** to `supabase/_archived_functions/` (mirrors
  `_archived_migrations/`), downloaded from the Supabase dashboard before
  deleting the deployed copies. Verified the source is free of hardcoded secrets
  (all read via `Deno.env.get`); added a README with provenance + "do not
  redeploy". This backs up the audit's "delete legacy Edge Functions" item so the
  deletion is reversible. (Notion Dev Changelog still 404 — git fallback used.)
- 2026-07-13 (security ops, no repo commit — Sivan + Claude): **Closed most of
  the open live-security cleanup** from open-tasks.md "Still open after the live
  deploy". DONE: (1) **Mapbox rotated** — new URL-restricted browser token +
  unrestricted server token; old leaked "Default public token" refreshed
  (value invalidated). Also FIXED a latent prod bug — `NEXT_PUBLIC_MAPBOX_TOKEN`
  was MISSING from Vercel, so browser maps had no token in production; now added
  + verified rendering. (2) **Twilio Auth Token rotated** (secondary→primary via
  the zero-downtime flow; the token pasted in chat is now dead). Account SID +
  Verify Service SID unchanged; Supabase Phone (Twilio Verify) config updated.
  (3) **service_role + Resend keys** — full git-history search PROVED they were
  never committed; the Vercel "Needs Attention" flag was only an
  un-marked-Sensitive nudge, not a leak. Marked Sensitive in Vercel; no rotation.
  IN PROGRESS this session: deleting the 6-7 legacy Edge Functions + configuring
  Supabase test phone numbers. STILL ON MAOR: delete the temporary Supabase
  personal access token from the June manual db push (only you can — it lives
  under your account).
- 2026-07-12 (commit `f6cced7`, `momlee-web`): **Unblocked the DB deploy
  pipeline.** ci.yml `db-deploy` now has a `supabase link --project-ref` step +
  `SUPABASE_DB_PASSWORD` (both CI secrets set; access token validated vs the
  Mgmt API). Fixed the red `checks` type error by dropping the 17 frozen
  provider-group values from the `audit_action` enum (migration
  `20260712000000` — enum recreated without them, dependent RLS policy
  preserved, `Record<AuditAction,string>` guard + generated types kept
  exhaustive). Extended `check-destructive.mjs` to also flag DROP
  TYPE/POLICY/FUNCTION. CI result: `checks` ✓, `rls-tests` ✓ (migration valid
  on a fresh DB), `db-deploy` ✗ — the migration's safety guard correctly
  aborted because LIVE `audit_log` has provider-group rows (see Tasks above).
- 2026-07-12 (commit `08cfa3c`, `momlee-web`): amended migration
  `20260712000000` to `DELETE` the 16 pre-launch provider-group test rows from
  `audit_log` before trimming the enum (rows confirmed disposable — pre-launch
  seed data). `checks` + `rls-tests` were already green; this unblocks
  `db-deploy` (CI re-run in progress at handoff).

_(2026-07-08: the momlee-web row was synced to the Dev Changelog after Notion
reconnected - 397450ad0ae681c7bceee1cfae7414ac.)_

## 2026-07-16 (Sivan → Maor) — two small asks

1. **Please bump the plugin version when you push content.** Plan C + the new
   `knowledge/target-data-model.md` landed on the marketplace at commit
   `ba0e443`, but `plugin.json` still reads `0.16.1` — so
   `/plugin install momlee-guide@momlee` sees "same version, already
   installed" and never refreshes the local cache. The plugin skills read
   `../../planning` and `../../knowledge` FROM that cache, so a Claude session
   running the skills straight gets the STALE (pre-Plan-C) files. I've manually
   synced my cache as a stopgap, but a version bump (e.g. `0.16.2`) on the next
   content push is the real fix so nobody reads stale directives.

2. **June "pending decisions" row (Dev Changelog) — one item is genuinely
   yours, the other two are mine.** Reclassifying by our ownership split:
   feature-flags kill-switch (infra) and the account-deletion cascade map
   (data policy) are Sivan's calls — I'm taking those (the cascade map will
   also fall out of Plan C's per-area retention work + IDV-9). The only piece
   that needs you: **the shared Empty/Error state designs in Figma**, which
   block building the four-states resilience component. No rush while Plan C is
   backend-only — just flagging it's the remaining UX dependency on that row.

## 2026-07-19 (Sivan → Maor) — MUST-READ: updated MVP strategy + a Feature/Story to model + a plugin update

**1. The MVP is re-scoped — please align your OS page designs to this.**
MVP ships with **admin + mom users ONLY. There is NO provider user in the MVP**
(no provider profile, no provider dashboard, no in-app subscriptions/billing, no
provider-created content).
- **Mom meetups** — created by moms (as today).
- **PRO meetups** — **IN the MVP, but published by the ADMIN** on a provider's
  behalf. When the admin creates a PRO meetup they enter, from info the provider
  supplied out-of-band: provider **photo**, a short **"who am I"**, **what the
  meetup covers**, a **Paybox** payment link, and the **attendee fee**.
- **Phased provider rollout:** grow mom traffic -> providers ask to self-publish
  -> introduce a **99 ILS/mo** provider fee (billed on a **separate platform**,
  not in-app) -> once enough pay, build the **full provider epic** (profile,
  dashboard, subscription tiers, self-serve pro meetups).
- **Organizations** = out of MVP (Phase-2). **Waitlist** = Post-MVP.
  **`qa` meetup type dropped** — Q&A/AMA is just a PRO meetup. Only **`mom | pro`**
  types. **Pro price band = 20-180 ILS.**

**2. Please formally model a new MVP Feature + Story in the OS (your lane).**
Feature: **Admin-published PRO meetup** (under the **Meetups** epic, Scope = MVP).
Story: *"Admin publishes a PRO meetup for a provider."* Actor = Admin. Flow: admin
fills provider photo / who-am-I / what-it-covers / Paybox link / fee +
location/date/time/capacity -> publishes -> it appears in discovery like any
meetup, tagged PRO with the host block + price + a register-and-pay affordance.
Wire its registries (Permissions: admin_create_pro_meetup; Events; Product Rules
for pricing). The 4 build Tasks already exist in the Tasks DB (migration + admin
publish UI + mom registration + admin approve/mark-paid), ready to link.

**3. Mom registration + interest model (so the designs match):**
- **Heart icon = "interested"** — adds the meetup to her "interested" list in her
  dashboard; NOT a registration request. The meetup shows **"X are interested in
  this meetup"** = count of moms who hearted it.
- **"Ask to register"** = a request the admin sees. She pays the provider via the
  **public Paybox group**; the **admin** sees the payment and manually confirms
  her (going + paid). No provider-confirm step.

**4. Meetups schema (the migration I am landing now):**
- `baby_meetups` += `meetup_type(mom|pro)`, `status(open|full|completed|cancelled)`,
  `capacity(2-100)`, `price(pro:20-180)`, `payment_link(pro:Paybox)`,
  `host_name/host_bio/host_photo_url(pro)`.
- `meetup_attendees`: `status(interested|asked_to_go|going|cancelled)` +
  `payment_status(pro: pending|paid|cancelled)`.

**5. Please update the plugin `knowledge/target-data-model.md` to this MVP plan**
(it still says `qa` type + 25-120 + provider-self-serve pro meetups). Drop `qa`
(only `mom|pro`), price 20-180, note PRO meetups are admin-published in MVP (no
provider user), organizations Phase-2, waitlist Post-MVP. Plus the **plugin
version bump** so the cache refreshes (still open from 07-16).

**Already reflected in Notion by me:** Meetups epic annotated (admin-published PRO
into MVP, Waitlist->Post-MVP); Provider Profile & Services / Subscriptions /
Business Insights / Organization & Team epics + the Professional user type marked
Post-MVP; duplicate provider page archived; 4 build Tasks created;
provider-dashboard-queries task deferred; a Dev Changelog row logged.

## 2026-07-19 (Sivan → Maor) — meetups-core migration is LIVE; mom-facing meetup UI is now the Figma critical path

- The Plan C **meetups-core migration is APPLIED TO LIVE** (db-deploy green,
  momlee-web 75bbc22). `baby_meetups` gained `meetup_type(mom|pro)` + the
  admin-published-PRO fields (host photo/bio, Paybox link, fee, capacity, status);
  `meetup_attendees` gained the registration lifecycle
  (`asked_to_go|going|cancelled`) + `payment_status`. OS Database Tables registry
  rows Meetups + Meetup Registrations flipped to **Aligned**.
- **What can proceed now with NO Figma:** the backend/data layer + the **admin**
  tooling (publish a PRO meetup, approve/mark-paid) — the admin surface needs no
  design per Sivan. **What now waits on YOUR Figma:** the **mom-facing** meetup
  screens (browse, meetup detail with the host block, the heart/favorite, the
  "ask to register" flow). That is the critical path on the meetups feature.
- Clarification to my 07-19 brief: the **heart = the EXISTING `favorites` table**
  (a like); a meetup's "X are interested" = `COUNT(favorites)`. `meetup_attendees`
  holds only the registration lifecycle. No change to your designs — just the data
  source of the interest count.
- Still open for you (from the earlier brief): formally model the MVP Feature/Story
  **"Admin-published PRO meetup"**, and the mom-facing meetup screens in Figma.

## 2026-07-21 (Sivan → Maor) — Web login decision: please update the login Figma

The web MVP is a PWA meant to look/feel like the native app, so Sivan decided the
login should be:
- **Phone-number + SMS OTP = the PRIMARY login** (native feel; best for moms —
  mobile-first, one-handed while holding a baby; the WhatsApp/Telegram/Wolt/Bit norm).
- **Google = a secondary one-tap option.**
- **Email/password = a secondary FALLBACK** behind a small "התחברות עם אימייל" link
  (better on desktop; account recovery if a mom changes her number).
- **DROP Facebook** (declining, heavy SDK, privacy/review friction).

Please **update the login Figma page** accordingly: phone-first and prominent,
Google as a quick option, a small email-fallback link, no Facebook. The phone
login/onboarding screens already exist in your Figma (built for native) — the web
reuses them, so the phone flow is NOT design-blocked. Building phone-OTP web login
is a near-term dev task on our side.

## 2026-07-21 (Sivan → Maor) — Naming decision (Sivan's call): new thin `providers` table

Sivan decided (naming/data-modeling is her lane, not UI/UX): admin-published PRO
meetups get a **thin `providers` table** — `id, name, photo_url, description` —
**admin-managed** (there is NO provider user in MVP; this just saves re-typing the
host info per meetup and gives moms a clickable thin provider profile). Meetups
reference it via `meetups.provider_id`.

Please reflect this in the OS glossary / Entities so the naming gate + entity map
stay honest. IMPORTANT coexistence note: this `providers` table sits ALONGSIDE the
frozen `provider_profiles` (the full Post-MVP provider-*user* profile) **on
purpose** — `providers` is the thin MVP record; when the full provider epic revives
post-MVP it reconciles into / links to the rich profile (an easy conversion —
`providers` gains a `user_id`, or merges into `provider_profiles`). So "provider" is
one concept at two fidelities across phases, not two competing entities.

## 2026-07-22 (Sivan → Maor) — MUST-KNOW: map provider will switch Mapbox → Google Maps at the search-page rebuild

Heads-up for when you design the **search pages** (meetups/providers map). Sivan
decided (backend/vendor/infra = her lane): **keep Mapbox for now, and migrate the
WHOLE map layer to Google Maps when the search pages are rebuilt from your Figma.**

**Why:** only Google can do **Hebrew-typed place search** — a mom typing `קפה מימי`
finds it. We tested Mapbox, HERE and OpenStreetMap directly: all three return **0
results** for Hebrew POI names (they only match English). Google is the one that
works — and going *full* Google (map + search) is also the licence-clean way to do
it. So this is a real product need, not a preference.

**What this means for YOUR map design (one real constraint):** Google's map styling
is a **recolor + show/hide** tool over a fixed set of feature types (roads, water,
parks, POIs, labels). You CAN fully rebrand it to the MomLee palette, declutter it,
and set label density. You CANNOT (unlike Mapbox Studio) use **custom fonts on the
map, custom illustrated icon/pin sprites baked into the basemap, or fully art-directed
per-zoom styling.** Custom **meetup/provider pins** are fine (those are overlay
markers — any image/HTML). So please design the map look as **"branded + decluttered
+ standard base iconography,"** not a fully custom illustrated basemap. If your design
needs something beyond that, flag it and we'll discuss before the migration.

No action needed now — just design the search-map with the above in mind. (Cost +
exact timing are Sivan's to handle at migration.)

## 2026-07-23 (Sivan → Maor) — ✅ CORRECTION / GOOD NEWS: the map STAYS Mapbox. Ignore the 2026-07-22 Google-styling constraint above.

The 2026-07-22 note above (👆 "migrate the WHOLE map to Google") is **superseded** — do
NOT design around Google's map-styling limits. **The map stays Mapbox permanently
(discovery map, custom markers, everything).** So **you keep full Mapbox Studio
freedom** — custom fonts, custom illustrated basemap, art-directed per-zoom styling,
custom pin sprites: all fine again.

**What changed:** deep sourced research found Google's Places policy has an
"end-user-selected address" exception that makes a HYBRID licence-clean — so Google
now powers **only the meetup create/edit SEARCH BOX** (where a mom types `קפה מימי`
in Hebrew), while the map stays Mapbox. The search box is our own RTL component
(built), invisible to your map design. Cost is tiny (~$15–275/mo, no Google map-load
fees). Net for you: design the meetups/providers **map** freely in Mapbox Studio as
you always would — no Google constraint.

## 2026-07-26 (Sivan → Maor) — FYI: new dependency `@googlemaps/js-api-loader` (please add a knowledge/stack.md row)

Sivan approved adding **`@googlemaps/js-api-loader`** (Google's official Maps JS
loader) to `apps/web`. It's the canonical, Google-maintained way to load the
Places library for the meetup location **search** (Hebrew POI autocomplete) — it
replaced a fragile hand-rolled `<script>` loader that caused a race/undefined
crash. Added to `scripts/momlee/stack-allowlist.json` so CI passes.

The dependency gate also wants a **row in `knowledge/stack.md`** — please add one
(name: `@googlemaps/js-api-loader`; used by: web meetup location search; why:
official loader for the Google Places New API; note: the MAP itself stays Mapbox,
Google is search-box only — see the 2026-07-23 map-provider note above).
