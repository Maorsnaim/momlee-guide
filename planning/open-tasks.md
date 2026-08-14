# Open tasks — read me on every pull

> Maor-maintained. Flows to Sivan via git. This is the live "what's pending /
> what changed" channel between us. Check it whenever you update the plugin.

## 🔴 DECISION (Maor, 2026-08-14) — recovery UPDATES the existing account. Never migrate it into a new one. ⚠️ One ordering contradiction to resolve.

**The existing `user_id` stays.** ⛔ The pattern *copy everything into a new
account → delete the old one* is **banned**, because chats, meetup participation,
connections, reports, blocks, family, verification history and analytics all hang
off that identifier. Recover and update — never replace.

**No partial state may survive** (`auth_access.canonical_account_preserved`): not
a new number attached without recovery finalised, not a provisional account left
active, not two canonical accounts sharing one verified identity, not duplicate
children, not a wrongly restored pregnancy, and **not a user signed in to the
provisional account instead of the recovered one**.

### Three engineering points, and the second is a real contradiction

1. **"One controlled operation" cannot be one database transaction.**
   Reconciliation may wait on her for minutes or days; you cannot hold a
   transaction open across that. The correct shape is a **persisted recovery state
   machine**: *phase A atomic* (ownership established, recovery marked in
   progress) → *phase B resumable* (reconciliation, possibly with her) → *phase C
   atomic* (the cutover). **A crash mid-way must leave a resumable state, never a
   broken one.**
2. **⚠️ The 17-step sequence contradicts itself.** Step 9 performs the phone
   change, but step 16 only retires the provisional identity at the end — while
   **that identity is still holding the exact number** and the platform enforces
   phone uniqueness. **The phone cutover and the provisional teardown are the same
   operation.** Either the phone move goes to the end with step 16, or the
   provisional identity releases the number at step 9. **Maor should pick one
   explicitly**, because either reading is defensible and the wrong guess locks her
   out of both accounts.
3. **The session switch does not happen by itself.** After recovery she is still
   authenticated as the *provisional* identity. An explicit move of the session to
   the canonical user is required — **it is the mechanism that prevents forbidden
   state number six**, and it is easy to leave out because everything else looks
   correct in the database.

### Analytics — one line that decides whether the numbers are right

**Alias the provisional id to the canonical one; never re-key history.** And
**count a verified user off the canonical promotion, not off a successful Didit
session** — otherwise every recovered mom is counted twice, because her Didit
verification *did* succeed, that is exactly what routed her here.

**Recorded in the OS:** Decision + Product Rule + a Critical System Story
(`finalise_recovery`, Ready for Dev) + a Task carrying all three points.

---

## 🔴 DECISION (Maor, 2026-08-14) — a pregnancy EXPIRES, and its outcome is never inferred. ⚠️ None of this model exists yet.

This answers the gap I raised on child matching, and it goes further: **it is a
global product rule, not a recovery workaround.**

**A pregnancy is never matched by the children 2-of-3 rule.** It has its own logic
based on the due date, the pregnancy status, a possible resulting child, and
**her confirmation**.

### ⚠️ Verified against the live DB today — there is no pregnancy model at all

`public.children` holds only `id, parent_id, gender, birth_date, name,
is_focus_baby, created_at`. **No pregnancy table. No due date. No status. And no
soft-delete column**, despite the family rules requiring soft delete.

So this is not a change to an existing model — **it is the model being defined for
the first time.** That is good news: no migration pain, no legacy states. But it
also means the four states below have to be designed properly now, not retrofitted.

### The state model

`ACTIVE` · `RESOLVED` (outcome known) · `HISTORICAL UNRESOLVED` (expired, outcome
unknown) · `SOFT-DELETED`.

⛔ **Never delete a pregnancy just because `due_date < today`.** Keep enough state
for history and for future reconciliation.

### Expiry — global, not recovery-only

Once **due date + grace period** passes, the pregnancy becomes eligible for the
expired state and **stops rendering as a current pregnancy** in Family and Profile.
A mom returning months or years later is never still shown as pregnant. The grace
period exists because the due date is an estimate, birth can be late, and she may
not update immediately. **Maor still owes the exact length.**

### 🔴 Never infer the outcome — this is the sensitive one

Closeness between a child's DOB and the due date, **and even a matching sex**, may
**suggest** a link. They never establish it. **She confirms, or nothing happens.**

- **"ילדתי"** → the existing add-child flow, prefilled with what is known; she
  still supplies the name and the actual date of birth. Then resolve the pregnancy.
- **"ההריון הסתיים"** → resolve without creating a child.
- **"מעדיפה לא לעדכן"** → stays historical-unresolved, **never reactivated, and
  never asked again and again.**

⛔ **No copy may assume a birth before she has confirmed it.** A pregnancy can have
a painful outcome the system does not know about. This is the one place in the
product where a wrong default sentence does real harm.

### One case that is not a conflict

An old expired pregnancy and a **new current** one are **separate events**. Never
show a "which due date is correct?" screen between them.

**Recorded in the OS:** Decision + two Product Rules (`pregnancy_expiry`,
`pregnancy_outcome_never_inferred`) + a Story + a Task carrying the schema finding.

**Maor owes 5 Figma states:** old pregnancy with a likely matching child · old
pregnancy with no known outcome · confirm birth into Add Child · pregnancy ended ·
prefers not to update. Reuse the existing Family / Already Gave Birth components.

---

## 🔴 DECISION (Maor, 2026-08-14) — child matching is 2-of-3 on name, DOB and sex. Plus five gaps the rule does not cover.

**The rule (`auth_access.child_record_matching`)** — compare only **first name,
date of birth, sex**:

| Attributes matching | Outcome |
|---|---|
| **3 of 3** | Same child. Keep existing. No duplicate. **Ask nothing.** |
| **2 of 3** | Probable same child. **No new record.** Surface the conflicting value for her to decide. |
| **fewer than 2** | New child, added under normal family validation. |
| **more than one existing record qualifies** | **Ambiguous — explicit confirmation required.** |

⛔ Forbidden: matching on name alone; on DOB alone; treating DOB + sex as enough
when several siblings qualify; **first-matching-child logic**; merging one twin
into another; creating duplicates when a confident match exists; silently
overwriting the third attribute. **A match establishes identity, never permission
to overwrite.** Recovery does not bypass family validation.

### ⚠️ Five things the rule does not cover — please do not build before these are answered

1. **🔴 A PREGNANCY BECOMING A CHILD is not handled at all** — and in a maternity
   app it is probably the *most common* recovery case. The old account holds a
   pregnancy; the draft holds a newborn. The rule sees no child to match, adds the
   baby as new, and leaves **a stale pregnancy plus a child**. This one needs a
   product answer, not an implementation guess.
2. **A 3/3 match must be resolved BEFORE ambiguity is evaluated** — otherwise
   *identical twin data triggers a needless prompt*. Each twin matches her own
   record at 3/3 **and her sibling at 2/3**, so the ambiguity rule as written fires
   on a perfect match. Fix: resolve all 3/3 pairs first, remove them from the pool,
   then evaluate 2/3 on the remainder.
3. **Matching is an assignment between two sets, not a per-record lookup.** Each
   existing child may be claimed by at most one incoming record and vice versa.
   Without strict tiers **the result depends on processing order**: a 2/3 record
   handled first can consume the child a later 3/3 record should have matched, and
   you get a duplicate. Tiering makes it order-independent. (This is the deeper
   form of the "first matching child" ban.)
4. **Unknown and null must not count as a match.** Our model allows sex `Unknown`
   / `None`, and a pregnancy with no date. Two `Unknown`s comparing equal would
   hand out a **free point** toward 2/3. An absent or unknown attribute must
   contribute **neither a match nor a mismatch**.
5. **Name normalisation must be defined** — trim, collapse internal whitespace,
   and a decision on nicknames (יוני / יונתן). Without it the rule quietly
   degrades to 2/3 and starts asking questions constantly.

**Recorded in the OS:** Decision + Product Rule + a Task carrying all five, under
the reconciliation Story.

---

## 🔴 DECISION (Maor, 2026-08-14) — onboarding data is MERGED into the recovered account. Both simplistic behaviours are banned.

She may enter a whole family before we discover she already has an account. That
data is **neither discarded nor allowed to overwrite** what exists.

**Three cases (`auth_access.recovery_data_reconciliation`):**

| Situation | Behaviour |
|---|---|
| **Identical** | Keep the existing record. No duplicate. **Ask her nothing.** |
| **Clearly new** | **Add it automatically.** Do not make her confirm or re-enter it. |
| **Material conflict** | **Show both versions** and let her say what is correct today. |

⛔ **Both of these are explicitly forbidden:** "the existing account always wins
and the onboarding data is discarded", and "the newest data overwrites the
existing". Applies to personal details, location, profile and pregnancy too, each
per its own field rule.

**Also required:** the merge is **idempotent** (never runs twice on a draft); the
**draft survives until reconciliation succeeds** and only then is cleared;
historical data unrelated to this onboarding is never touched; a merge event goes
to analytics. **A reconciliation screen appears only for a genuine conflict** —
never a generic "merge your account" page.

**Children matching is defined separately — do NOT invent a substitute for it.**

### Two things I would flag before this is built

1. **The default failure mode of exact matching is creating DUPLICATES**, not
   merging wrongly — any small difference in spelling or date yields a second
   record, and duplicates are exactly what this rule forbids. So please
   **instrument how many merges close automatically versus needing a decision**.
   That ratio is the signal for whether the matching rules are good enough, and
   without it we will not know they are failing.
2. **The conflict screen lands at the worst possible moment** — she has just been
   told she already has an account and has just proven ownership, and now she is
   asked to adjudicate her own family data. Worth considering: **let her into the
   account and defer non-blocking conflicts** to a task inside the app. Only block
   on conflicts that would actually corrupt data. That is Maor's call, and I have
   put it to him.

**Recorded in the OS:** Decision + Product Rule + a System Story + a Task, and the
old open question *What happens to the children data in the discarded provisional
account* is now **Decided** by this — the answer is that it is merged, not
discarded.

---

## ⭐ MAOR'S DIRECTION (2026-08-14) — the identity key is a hash of the NATIONAL ID NUMBER, not a face template. Please read the problem before the solution.

### The problem, stated plainly

We want **one verified person, one account** — including a mom who comes back in
2028 with a new phone. That requires *something* permanent that identifies the
person. Today we have **only the face**, and Didit's face memory dies with the
retention purge, currently at **6 months**. So from month seven the same woman
reads as a brand-new person, and neither duplicate detection nor recovery works.

There is no way around needing *some* persistent identifier. The only question is
which one.

### Maor's choice, and why it beats the face

**A one-way hash of the person's national identity number.** Compared with keeping
a face template it is:

- **not biometric data** — a far lighter legal burden than a stored face;
- **exact** — no false matches, no identical-twins problem, no ageing;
- **permanent** — it does not evaporate when a vendor purges sessions.

### ⚠️ The distinction that makes it work — and it is easy to get wrong

Maor's concern was whether this survives a **passport** or a **driving licence**.
It does, but only if we hash the right number:

| | Changes on renewal? | Same across document types? |
|---|---|---|
| **Document number** (the card/booklet serial) | **Yes** | **No** |
| **National ID number** (מספר זהות — the person) | **No** | **Yes** |

Israeli ID cards, passports and driving licences **all carry the national ID
number**. So hashing *that* is invariant across document type and across renewals.
Hashing the **document number** would break on both. **Please make sure whatever
is built keys on the person's number, not the document's.**

### How to hold it, so this does not reverse the Amendment 13 position

**A one-way hash of an ID number is not the ID number.** The minimisation you put
in place was about receiving and storing the raw value; that stays. The raw number
is used to compute the hash and discarded — never persisted, never logged.

Two properties this needs, and they are not optional:

1. **The key lives in a secret manager, never in the database, never beside the
   data it protects.** An Israeli ID is 9 digits with a check digit — on the order
   of a hundred million valid values, fully enumerable. Unlike a password hash,
   **the attacker knows every possible input**, so the key is the entire
   protection and a leak exposes every user's ID number.
2. **It is not practically rotatable** — recomputing needs the raw numbers, which
   we deliberately do not keep. Rotating means losing the whole duplicate history.
   Treat the key as permanent critical secret material, not a config value.

Also scope the hash by **issuing country**, so numbers from different countries
cannot collide.

### The three things still to settle — the first is yours to check

1. **Does Didit actually return the national ID number for a passport and a
   driving licence, or only for the ID card?** The ID card is near-certain; the
   other two need checking against a **real payload**, not the docs. This is the
   sharpened version of lookup 4 below, and everything depends on it.
2. **What happens when the field is missing, or the document is foreign?** A
   foreign passport carries no Israeli number. Options: fall back to the face
   check, fall back to hashing document number + country + type (weaker, breaks on
   renewal), or treat it as undedupable. Needs a decision.
3. Key management per the two properties above.

**Recorded in the OS:** Product Rule `trust_safety.identity_uniqueness_check` now
carries this mechanism instead of the face-template one, and the Blocking open
question holds Maor's direction plus the caveats.

---

## 🔴 DECISION + ❓ BLOCKING (Maor, 2026-08-14) — no recovery method means MANUAL REVIEW. The routing is settled; the policy is yours, and it is blocking.

**Decided:** when a mom reaches recovery and has access to **neither** the old
phone **nor** a previously verified email, she is **not** asked to verify with
Didit again — the verification that triggered recovery is enough at this stage.
The case goes to **manual review by the Momlee team**, which is the only fallback.
⛔ **Do not build an alternative automatic fallback for this scenario.**

**Please treat the policy as urgent, not as an edge case.** Email verification
does not exist yet and face memory is 6 months, so **right now this path is where
most moms who lose a phone will land.** It is currently the main road, not the
shoulder.

### Two findings that should shape your answers

1. **Everything the security constraint rules out is visible to other moms.**
   Children's names and dates of birth, pregnancy, city, photo, interests — that
   is exactly what a community member sees. So the rule is right, and for the
   strongest possible reason: **the plausible attacker here is an existing member,
   not a stranger.** Recorded as `auth_access.ownership_evidence_strength`.
2. **⚠️ This one blocks one of the questions outright.** "May support compare the
   Didit result against historical account information?" — **Momlee holds no
   verified name and no document number**, because you switched those fields off
   under Amendment 13. There is almost nothing to compare inside our own system,
   and any comparison means opening the session in the Didit console. Worth
   deciding deliberately rather than discovering it at the first real case.

### A direction for "what counts as strong evidence", not a ruling

**Her identity is already proven by Didit. What is unproven is that this identity
belongs to that account.** So strong evidence is something tying the person to the
account's *history* that other members cannot see: knowing the old phone number
itself even without access to it, the join date, non-public activity. Everything
on the excluded list fails precisely because it is member-visible.

### The full question set is on a Blocking open question in the OS

*What is Momlee's manual account-recovery policy*, grouped into **evidence**,
**permissions**, **audit**, and **limits + communication** — all of Maor's
questions, verbatim, plus a Task under the admin Story. **The admin Story sits at
Draft and stays there until the policy exists.**

### One recommendation worth building in regardless of the policy

**An approved recovery should send a security notice to the account's EXISTING
contacts.** It is the only way the real owner ever discovers a wrongful recovery.
Worth sending even when it looks redundant.

**Maor owes the Figma screen** for the neutral pending state (`נדרשת בדיקה נוספת`)
— it must promise nothing, and must never read as an accusation.

---

## 🔴 DECISION (Maor, 2026-08-14) — a previously verified email can recover an account. ⚠️ It does not exist yet, so today this method is dead on arrival.

**The rule (`auth_access.pre_existing_contact_recovery`):** a recovery method is
valid only if the contact was attached to the account and **verified before this
recovery attempt began**, the account still holds it as valid, and it was not
replaced by an unverified contact during the current flow. **An email added or
verified during the current onboarding never recovers an older account.**

When both an accessible old phone and a verified email exist, she **chooses**
where the code goes (`••••0902` or `n•••@momlee.app`). Only genuinely available
methods are shown. **One approved method is enough** — never both. After success
the flow continues to the phone-number choice from the decision below. With **no**
method available, the case goes to manual review.

### 📋 NEW TASK FOR YOU — email capture + verification in profile completion

**Maor's clarification (2026-08-14), and it changes the picture:** an email step
**is** planned, it just has not been built. Every mom is asked for an email at the
**end of onboarding, as part of profile completion**, right after phone + OTP —
and it is **verified immediately**.

The phone stays the **primary** identifier. The email is a strong **secondary**
one. In practice anyone with a verified email also has a verified phone, never the
reverse. **The reasons are marketing and transactional email** — both central to
the product, not an afterthought. Recovery is a third benefit that falls out of it
for free.

**Task created in the OS** (High, under *Mom completes profile after
verification*) so it reaches you through the normal flow. What it needs:

- an **application-level `email_verified_at`** on `users` — `auth.users.
  email_confirmed_at` alone cannot answer *"was it verified BEFORE this recovery
  attempt"*, which is precisely what the recovery rule tests;
- the verification step inside profile completion;
- the delivery path (this meets the existing transactional-email task).

Two things to build in from the start, rather than retrofit:

1. **A 6-digit code, not a magic link** — see the PWA reasoning below.
2. **Marketing consent recorded separately from the address.** The Message
   Categories model already splits transactional from marketing consent; an email
   collected *for marketing* needs its own recorded consent, not an implied one.

**Open for Maor, and it decides how much load manual review carries:** is the
email step **required** to finish onboarding, or strongly encouraged and
skippable?

### ⚠️ This cannot work today — I checked

- `public.users` has an `email` column but **no verified flag**. Verification
  state exists only as `auth.users.email_confirmed_at`.
- **There is no email verification anywhere in the web app.** Zero references to
  `email_confirmed_at`, `email_verified` or any confirm-email flow in
  `apps/web/src`. (The 17 confirmed emails in the DB are legacy email/password
  test accounts from before the phone-OTP pivot, not moms from the new flow.)

So unless email verification is built, **every mom who loses her old number falls
straight to manual review**. Two things are needed: an application-level
`email_verified_at` (the auth timestamp alone will not carry the "was it verified
*before* this attempt" test), and an actual verification step somewhere in the
product. Worth deciding whether that lands in MVP — the answer decides whether
Section 7 handles a trickle or everybody.

### Use a CODE, not a magic link

MomLee is a **mobile-only PWA**. A magic link opens in whatever browser handles
the mail app, which is not the PWA context holding the session — she taps it and
lands somewhere that knows nothing about her flow. A 6-digit code typed back into
the screen she is already on has none of that failure mode.

### An attack this rule does not yet close

Someone with brief access to an account could add an email today; **weeks later it
is "previously verified"** and satisfies the rule perfectly. The wording defends
against same-session abuse, not patient abuse. The usual mitigation is a
**minimum age before a newly added contact becomes recovery-eligible** (say 7-30
days), plus notifying the *existing* contacts when a new one is added. Not
inventing a number — flagging that it needs one.

### One interaction to respect

The method chooser reveals that an account exists and roughly what contacts it
has. For a **blocked** account the no-hint rule says reveal nothing — so the
chooser must **not render at all** there. The two rules only coexist if the
blocked check happens before the chooser.

### Audit

Log the attempt time, the method (`phone` / `email`), success or failure, the
account id, the provisional session reference, and the resulting phone decision.
**Never the OTP, and never the full address or number** — the method name is the
useful part; the value is what makes an audit log a liability.

**Recorded in the OS:** Decision *A previously verified email recovers an account*
(Decided) + Product Rule `auth_access.pre_existing_contact_recovery`.

---

## ✅ MAOR DECIDED (2026-08-14) — the forgotten new number: keep it as a recognised alias

On the "keep the old number" branch below, Maor chose **option 1**: store the new
verified number as a **known non-login number** on the account, so login routing
recognises it and sends her to **sign in** rather than into onboarding.

It is **not** a second login phone — the "one primary login phone" direction
stands. It is an alias that exists purely so the identity-match-and-recover loop
does not repeat every time she uses that number. Both numbers are proven at this
point, so recording the one she is not signing in with costs nothing and closes
the loop.

---

## 🔴 DECISION (Maor, 2026-08-14) — after recovery she CHOOSES her login phone. Never replace it silently. Plus one trap and one hole.

**The rule (`auth_access.explicit_phone_replacement`):** an existing account's
login phone is **never** replaced just because a different number was typed
during onboarding. Replacement requires all three:

1. the new number passed OTP,
2. ownership of the existing account was proven through recovery,
3. **she explicitly confirmed** on a dedicated screen.

The confirmation screen shows the old number masked (`••••0902`) and the new one,
explains that ownership is proven, and asks which one she wants to sign in with.
**Primary CTA = use the new number** (it is the one she chose to start with).
Secondary = keep the old one. Either way she lands in the **existing** account,
history and relationships intact, and no second account is created.

The reasoning worth keeping: a successful code on the old number proves
**ownership**, not **preference**. And typing a new number at the start says
nothing about wanting to give up the old one. Both directions need her say-so.

### ⚠️ The engineering trap — ordering against Supabase Auth

If she picks the new number, the phone has to move onto the **canonical**
account's auth identity — but the **provisional** auth identity created at the
start of this flow is currently holding that exact number, and **Supabase Auth
enforces phone uniqueness**. So the provisional identity's phone must be released
**before** the canonical account can take it. Get the order wrong and you can end
up with both identities broken and her locked out of both. Please make this one
transaction with a defined order, not two independent updates.

### 🕳 The hole in the "keep the old number" branch

If she keeps the old number, she has just proven control of a new number that the
system then **forgets**. Next time she signs in with it, she is a stranger again:
new onboarding, Didit, identity match, recovery — **the same loop, forever.**

Three ways out, and this needs a decision:
1. Store the new number as a **known non-login number** on the account, so login
   routing recognises it and sends her to sign in rather than onboarding.
2. Tell her plainly on the confirmation screen: *"you will not be able to sign in
   with 052-...-1234"*. Cheapest, and honest.
3. Accept the loop. I would not.

Option 1 conflicts slightly with "one primary login phone only" — it is not a
second login phone, it is a recognised alias. Your call.

### Your remaining questions, with my read

- **Is the old number removed after replacement?** Recommend: removed as a login
  credential, retained in the security audit log. It is the single most useful
  record if the change is ever disputed.
- **Security notification: yes, and send it to the OLD number.** A phone change is
  a classic account-takeover step. Send it **as part of the change, while the old
  number still receives** — afterwards is too late to warn anyone. Also to a
  verified email if we have one.
- **More than one login phone?** Current direction is one. Nothing here needs to
  change that (see the alias option above).
- **Admin visibility + audit log:** yes to both. Phone changes belong in the
  account security audit log alongside the verification decisions already there.

**Maor owes the Figma screen** for the confirmation step — it is a new one, on top
of the recovery screens already on his list.

**Recorded in the OS:** Decision *After recovery the mom chooses her login phone*
(Decided) + Product Rule `auth_access.explicit_phone_replacement`.

---

## 🔴 HIGH (Maor, 2026-08-14) — the 6-month retention breaks account recovery. Most of the investigation is already done; here is only what is left.

**Maor's scenario, and it is the right one:** she verifies in 2026, changes her
phone number in 2028, and starts registration again. We must still know she
already has an account. **Today we would not.** At 6-month retention she reads
as a brand-new person from month seven onward.

### Please do NOT re-run the investigation — you already answered most of it

Going back through IDV-9 and the admin-review task, these are settled **in
writing** and should not be put to Didit again:

- **Can we purge the images and keep a face reference? NO.** Didit's own words:
  *"both the stored images and the associated face embeddings are removed... the
  system no longer has the reference material required to perform face matching."*
- **Does duplicate detection survive the purge? No** — dedup matches against
  previous *sessions*, so the horizon equals the retention window. Currently
  **6 months** (your ruling of 2026-08-12, set in the console).
- **The ID-card-then-passport scenario is moot after a purge.** With no reference
  left at all, the document type never gets a chance to matter.
- **Lists are the one thing that persists** — *"they persist until you manually
  remove them"*, independent of session retention.
- Retention is configurable **30 days to 10 years**, plus a per-session delete API.

### The reframe that matters

**You made the 6-month call knowingly and it was reasonable — the context has
since changed.** You wrote it yourself in IDV-9: *"The 6-month duplicate-face-
detection horizon is accepted for MVP."* That was accepted when the trade was
*privacy vs catching a second account*. **Account recovery did not exist as a
requirement yet.** Now the same 6 months also decides whether a returning mom
with a new number can get back into her own account. That is a different trade,
and it deserves a fresh look rather than being carried forward by default.

### What is actually still open

1. **Can Lists hold ALL verified members, not only blocked ones?** (Also asked
   below — this is now clearly the highest-value question. If yes, we get
   permanent recognition without extending retention and without MomLee storing
   anything itself.)
2. **Does a Didit User / person ID persist after the session is purged?** Not
   documented either way.
3. **Can retention be set per category** (purge the media, keep the decision
   record)? It reads as one app-level setting, but worth one question.
4. **What does `face_privacy_mode_enabled` actually do?** You flagged it as
   unknown in IDV-9 and it was never chased. It may be directly relevant here.
5. **If none of the above works: does MomLee keep its own identity reference?**
   That one is Maor's decision, and it is the fallback if Didit cannot hold the
   memory for us.

### The legal texts are NOT a constraint here

An earlier version of this note said that changing the retention window means
changing a published commitment. **Maor's correction, and he is right: the app has
not launched.** There are no users and nothing has been promised to anyone. The
terms of use and privacy policy are **drafts**, so the number in them simply
follows whatever we decide — it does not constrain the decision.

**Pick the retention window on the merits first, then write it into the drafts.**
Not the other way round.

---

## ❓ OPEN + 🔴 CONFLICT (Maor, 2026-08-14) — what is the identity key for duplicate detection. Four Didit facts needed from you, and the live code disagrees with two decisions.

Maor is writing the rule that decides **verified identity → new user** vs
**→ existing account → recovery** vs **→ human review**. It is deliberately
**not decided yet** and nobody should implement matching logic until it is.

### ⚠️ Maor's position on the three problems below — read this first

He has now seen the three findings and wants his view on record. These are his
leanings, not rulings — the technical answers are yours to give, and on the third
one the decision is explicitly yours.

1. **The face-memory window is a real problem, not an acceptable trade.** Once it
   lapses the same woman reads as a brand-new person and can simply open a second
   account. Please treat this as something to solve, not to live with.
   (Correction: an earlier version of this note said 12 months — it is **6**,
   per your 2026-08-12 ruling. The problem is twice as sharp as first written.)
2. **He is fairly convinced we need an identifying number — a face alone will not
   be enough.** He knows this pulls against the Amendment 13 minimisation you put
   in place, and he is not overruling that. If you think a number can be avoided,
   or that there is a way to hold one that stays compliant (a one-way hash rather
   than the number itself is the obvious candidate), make that case.
3. **Sending a match to human review makes the mom wait far too long, for no good
   reason in his view — but the decision is yours, not his.** His concern is the
   experience: a woman who is simply returning to her own account should not be
   parked in a queue waiting for a person. If there are cases where you believe a
   human is genuinely required, say which ones and why.

**Before it can be decided, three things about the current build matter — you
know them better than anyone, so please correct me if I have any of them wrong:**

1. **The mechanism already exists and is face-only.** IDV-4 resolves
   `duplicated_session_id` to a MomLee account and decides: banned → decline,
   deleted → approve, **still active → needs review**.
2. **The national ID route is currently switched OFF on purpose.**
   `personal_number` and `document_number` come back `null` because you unticked
   them under Amendment 13 minimisation, after the Extra-fields PII leak.
   So "use the Israeli ID as the key" is not a free choice — it is **reversing a
   live privacy position**, and it needs to be argued as such.
3. **Face memory is 6 months** (a retention purge deletes the embeddings, and
   retention is set to 6 months). Recognising a returning mom needs a reference
   that lasts the **life of the account**. Face-only cannot satisfy that.

### 🔴 The conflict, for you to weigh in on

Your live engine sends **face match against a still-active account → human
review**. Two decisions in the OS (2026-08-10 and 2026-08-14) both say that case
**routes to Account Recovery**. One of the three has to give.

My read: `needs review` was the right call when it was written, because **account
recovery did not exist yet** — a human was the only route forward. Now that
recovery exists, auto-routing is defensible, because **routing is not
authentication**: recovery re-authenticates her, so a false positive costs
friction and not access. But that is Maor's call, not mine, and I have not
touched your decision table.

### Four factual lookups I need from Didit (facts, not decisions)

1. Is there a **one-to-many Face Search** as a separate product — on which plan,
   at what cost?
2. Does the duplicate warning carry a **confidence score**, or is it binary? If
   it is binary, the whole "what threshold" question disappears.
3. Can **Lists hold all verified members**, not only blocked ones? You already
   proved Lists survive retention purges. If yes, that is permanent duplicate
   memory **without MomLee storing anything itself** — it would satisfy the
   2026-08-13 decision inside the existing privacy position.
4. Is `personal_number` populated for an **Israeli passport and driving licence**,
   or only for the ID card? (Israeli passports and licences do print the national
   ID number, so this may be a smaller problem than it looks — but it needs
   checking against a real payload, not the docs.)

Item 3 is the one I would check first. It may make the whole national-ID debate
unnecessary.

### One thing worth designing for either way

Whatever the rule ends up being, an auto-route to recovery should carry a
**"this is not my account" exit into human review**. It costs one link and it
handles identical twins, lookalikes and any false match — without needing a
second matching signal at all.

**Recorded in the OS:** the Blocking open question *What exactly is stored as the
verified-identity key* now holds the full state of play, the four lookups, and
the conflict. **Nothing was decided and no decision was flipped.**

---

## 🔴 DECISION (Maor, 2026-08-14) — a verified phone creates a PROVISIONAL account, not a real one. This changes the signup path you already built.

**The binding rule (Product Rule `auth_access.provisional_until_verified`):**

> A new verified phone does not create a canonical Momlee account until identity
> verification confirms the person does not already belong to an existing one.

Phone OTP proves she controls the number. That is all it proves. She becomes a
real Momlee user only when Didit comes back clean. If Didit matches an identity
we already have, **no second account is created** — she is routed to account
recovery and the original account stays canonical. An **existing** phone still
logs straight in as it does today and is never sent through onboarding.

### The one distinction everything hangs on

**A Supabase Auth identity is not a Momlee account.** OTP verification
unavoidably creates an `auth.users` row — that is the auth event itself, and
there is no point fighting it. So the provisional-vs-canonical line lives in
**our** tables, never in the presence of an auth row.

### How to build it safely

1. **ONE marker, not three.** Add a single explicit column — `users.account_status`
   (`provisional` / `active` / `suspended`) — set by the promotion step. Do **not**
   derive "is she real" from a combination of *has the parent role* + *verification
   approved* + *profile complete*. Three derivations means three places to get it
   wrong, and they will disagree the first time a webhook retries. Everything in
   the app asks that one column.
2. **The database is what actually enforces uniqueness, not the code.** Put a
   `UNIQUE` constraint on the identity reference (per the 2026-08-13 identity
   cross-check decision). An application-level "have we seen this face" check is
   UX; two verifications landing at the same moment is a race, and only the
   constraint stops it. The check runs **before** promotion, in the same
   transaction.
3. **Promotion is one server-side function, and only the webhook may call it.**
   Never from the client. It must be **idempotent** — Didit will retry. Granting
   the `parent` role happens **inside** that same transaction, not on a separate
   path.
4. **Store onboarding data in the real tables, gated by the status** — not in a
   parallel `onboarding_drafts` blob. A shadow schema means writing every field
   twice and drifting the moment one side changes. The cost of doing it this way
   is that **every read path must exclude provisional users**: feeds, member
   lists, search, meetup attendee lists. Please put that predicate in one place
   (a view or a single helper) rather than repeating it per query. If you think
   the leak risk outweighs the duplication cost, say so and we will decide -
   this one is a genuine trade-off, not a ruling.
5. **RLS: re-audit every policy that assumes "authenticated = a real mom".**
   Policies keyed on the `parent` role are already safe because the role now
   follows verification. The exposed ones are any policy keyed on `auth.uid()`
   alone. A provisional user may reach **her own onboarding data and nothing
   else**. Also force a session refresh after promotion so any claim-based gating
   updates.
6. **Never delete the provisional record before recovery succeeds.** On an
   identity match, freeze it — do not discard it on the way into recovery. If the
   recovery then fails, deleting early leaves her with nothing at all.
7. **Abandoned provisional accounts need a purge rule.** They hold children's
   dates of birth, which is personal data about minors sitting in a record that
   will never become an account. Propose a retention window and a purge job;
   it needs a Privacy Note either way.
8. **Analytics must survive promotion.** Capture a stable id at OTP and emit an
   alias/identify mapping provisional id → canonical id at promotion, so the
   funnel stitches instead of showing every mom dropping off at verification.
   The `Support anonymous users` story in the OS already covers this pattern.

### Already aligned — no change needed

Your IDV-7 ruling (**the parent role follows verification approval**) is exactly
this decision, one layer down. It stays. What changes is that the `users` row
created by the signup trigger must now be born `provisional` rather than being
treated as a finished account.

### One honest limitation, so you do not over-promise it

The two-attempt verification limit is **phone-scoped**. Someone who burns both
attempts can start over with a different number and get two fresh ones — the
identity cross-check cannot catch her, because a *failed* verification produces
no identity to match against. Rate limiting and device signals mitigate it;
nothing here makes it airtight. Fine for MVP, just do not describe it as such.

Related: decide what happens to a provisional account that has **exhausted both
attempts** — it is a dead end today (authenticated, no account, children's data
stored, no path forward).

### Still open, owned by Maor

- The exact identity key format (Didit person id vs hashed document number).
- Whether the provisional account's children data is merged or discarded on a
  successful recovery.

**Recorded in the OS:** Decision `A verified phone creates a provisional
onboarding account, not a canonical Momlee account` (Decided) + Product Rule
`auth_access.provisional_until_verified` (Approved, MVP, Backend + Frontend).

## 🔴 DECISION (Maor, 2026-08-13) — identity history is NOT deletable: every verification must cross-check face + document. This conflicts with the 6-month retention you set.

**The decision.** On every Didit verification we must cross-check that **this
face and this identity document have never been used to verify on Momlee
before**. If they have, she is routed to **account recovery** — never given a
second account. One verified identity, one account. That check IS the
mechanism, so the identity history cannot be wiped.

**The distinction that makes this workable (and keeps privacy intact):**
keeping the **matching ability** does not require keeping the **images**.

| What | Retention |
|---|---|
| Selfie + ID photos | **Keep purging** per the retention policy — unchanged |
| Identity **reference** (face template + a hash of the document identifier) | **Retained for the life of the account** — this is what the cross-check runs against |

**Three things for you to resolve:**
1. **⚠️ It conflicts with IDV-9.** You set Didit retention to 6 months, and the
   earlier finding was that a purge deletes face embeddings too (hence the
   "12-month ban memory"). Under this decision that framing is void. Please
   check with Didit whether a **face template can persist while the images are
   purged**. If it cannot, we must store our own reference — which makes the
   next point mandatory.
2. **Legal.** A retained face template is **sensitive biometric data**: it needs
   a lawful basis and explicit wording in the **privacy policy** and **terms of
   use**. This changes what your two open legal tasks must say — if either
   currently promises "we keep nothing", that text is now wrong. It also needs a
   Privacy Note + a Schema Registry row for whatever we store.
3. **What exactly is the key.** Still open (recorded in the OS): Didit's own
   person identifier, or a one-way hash of document number + country with a
   server-side pepper. **Never a raw national ID number.**

**Recorded in the OS:** the blocking open question is now **Decided**, and a new
Product Rule `trust_safety.identity_uniqueness_check` carries the full rule.

**Note on scope:** this also means a **banned** identity is remembered
indefinitely, not for 12 months. The no-hint rule still holds — a blocked match
produces no hint and falls back to the ordinary not-approved screen, so nothing
in the product ever tells a mom she is barred.

## ✅ ANSWERS (Maor, 2026-08-12) — email DOES exist in the flow, family cap is 12, and the flag is gone

1. **⚠️ CORRECTION — "we have no email address for her" is not accurate.**
   Maor designed an **optional email capture inside the waiting state itself**:
   `09.6_Onboarding/Mom/Verification/Pending` has an email field ("כתובת
   הדוא״ל הכי טובה שלך") plus a separate marketing-consent checkbox. So the
   Under-review / Pending copy does not have to send her away empty-handed —
   the screen is now annotated with the full logic:
   - **Two purposes, one field:** telling her the verification result is
     **transactional** and is allowed **WITHOUT** the marketing checkbox (it is
     literally why she typed it). Marketing needs the checkbox, unchecked by
     default, stored **with a consent timestamp** (Amendment 40).
   - **The email is UNVERIFIED here.** Use it to notify her, but it must not
     become a login credential or an account-recovery channel until it is
     verified by magic link in profile completion — which also matters for the
     recovery policy's "security notification to a verified channel".
   - **Two copy variants:** email given → "we will email you"; no email →
     "open MomLee again shortly and you will see the answer here" (your ruling
     stands for that case).
   - The screen also carries **`שמרו את פרטי ההתחברות שלי לפעם הבאה`** — that is
     the **session persistence** you were tasked with, designed. It is not
     consent and is stored separately from the marketing flag.
2. **Family cap: 12 approved.** `onboarding.family_entry_limits` updated in the
   OS — up to **12 children**, one active pregnancy; everything else unchanged.
3. **`identity_verification_enabled` removed from the plugin, and your rule is
   now the standing one:** `knowledge/architecture.md` and
   `momlee-react-native` were rewritten — a flag exists only to switch a live
   experience between two implementations during a real test, never as a
   standing kill switch, and Claude sessions are told explicitly not to
   propose it. Pre-launch the kill switch is an env var.
4. Points 2-4 of your verification feedback (reject reason with room for real
   admin text, the duplicate-account masked hint and the login-routing audit)
   are accepted — Maor is doing the screen work; the masked-hint constraint
   (render only when a hint exists, blocked accounts produce none) was a sharp
   catch.

## 🚀 ONBOARDING IS READY FOR HANDOFF (Maor, 2026-08-10) — and the OS was brought in line with the Figma

The full mom onboarding flow is designed, annotated and now mirrored in the
Momlee OS. **Story by Story, as you asked** — start with the auth chain
(01-07) and the verification screens (09); the family step (08) is complete
too, including edit/delete.

**New Product Rules (7, all Approved, linked to their Epics)**
- `onboarding.mother_min_age_16_at_child_birth` — **the cross-validation your
  edit annotations referenced 4 times but which was never defined**:
  `childDateOfBirth >= motherDateOfBirth + 16 years`, calendar-aware, error on
  the CHILD's field, re-evaluated against every child if she later edits her
  own date of birth.
- `onboarding.family_entry_soft_delete` — **delete = SOFT delete.** Flagged and
  excluded from every list/count/grouping; hard removal only via the
  account-deletion cascade; ownership verified server-side; auditable
  (children's data).
- `onboarding.family_entry_limits` — up to **8 children**, exactly **one active
  pregnancy**; the add action disables at the cap and a soft delete frees a slot.
- `onboarding.multiple_birth_derived` — **twins/triplets are DERIVED** from an
  identical date of birth (presentation only, each child stays its own record),
  **max 3 on the same date**, recalculated on every date edit. There is no
  add-twins flow.
- `onboarding.family_entry_gender_required` — a selection is required;
  `unknown` exists only for a pregnancy and is a real stored answer, not a skip.
- `onboarding.child_name_hebrew_only` — Hebrew letters only for now, **but the
  implementation must be locale-aware, not a hardcoded Hebrew regex**.
- `app_shell.mobile_only_guard` — see the desktop section below.

**New Stories (7) + 3 rewritten**
New: `edit_child` · `delete_child` · `edit_pregnancy` · `delete_pregnancy` ·
`location_area_waitlist` (the area-not-covered waiting list, frames
06.5.1-06.5.3) · `app_shell.mobile_only_gate.desktop_visitor_message` ·
`trust_safety.identity_verification.duplicate_account_detected`.
Rewritten: `add_primary_child` → **`add_child`** (the primary-child concept is
gone), `add_additional_children` → **`family_list`**, and
`pregnant_due_date_children` → **`add_pregnancy`**.

**Deprecated / superseded — please do not build these**
- `onboarding.primary_child_max_age_2` → **Deprecated.** There is no primary
  child; age groups are derived and `newborn` means up to ONE MONTH, not two
  years.
- `onboarding.mom_onboarding.enter_email` → **Deprecated.** Email is collected
  in profile completion after onboarding, per the 2026-07-21 decision.
- Technology Stack: **Persona → Deprecated, `Didit` created** (Approved /
  Implemented). The Trust & Safety Epic description was corrected too.

**Six stale verification questions CLOSED with what you actually built**
Sex is read from the **document, not the face** (a deliberate choice — face-based
gender detection fails most on women with short or covered hair) · **2 attempts**,
enforced on one row per mom so a second application cannot buy a fresh allowance ·
**not in-page** — she leaves to Didit and returns, and returning does not mean she
finished · the result arrives by **server-to-server webhook**, which is exactly why
the return screen opens in a waiting state · a failure is **never a ban**: second
attempt with a fixable reason, then support only, with human review in parallel ·
and **an identity already in the system means she HAS an account** → route to sign
in + account recovery, never anything punitive.

**Desktop: there is no desktop product, with one exception**
Every non-phone visitor gets the `Mobile Only Message` screen (designed) with a
**dynamic QR that encodes the URL she tried to open** — a mom opening a meetup
invite on a laptop must land on THAT meetup after scanning. ⚠️ Do not implement
the gate as `innerWidth >= 768`: a phone in landscape is ~844px wide and would
wrongly get the desktop screen — decide by the device's smaller dimension +
pointer type. **Exception: the legal pages** (privacy, terms, accessibility
statement, contact, support) stay reachable on desktop — two new Tasks are open
for that and for the QR dependency request.

## 📌 BUILD CONTRACT (Maor, 2026-08-10) — `Input` stays untouched; the label lives in a `Field` wrapper

Maor hit this in Figma and it applies verbatim to the code: the base
`Forms / Input` is a horizontal row with **227 instances** in the file, and
every attempt to add an "input title" inside it restructured all of them.
The resolution is a composition, and it is now the contract:

- **`Input` keeps its current shape — no `label` prop, no structural change.**
  A change that would restructure `Input` is a red flag, not a task.
- **`Field` is a separate component with its own Figma node** (`Forms / Field`
  in Momlee 2.0), composing an `Input` instance and adding the optional
  label above it — later also helper / error text. Figma properties today:
  `Show Field Title` (boolean) + `Text` (label copy).
- Same rule for the other field types: if `Phone Field` / `Date` /
  `OTP Field` ever need a label, they get it from the wrapper, not from
  their own internals.

## 📌 RULINGS (Maor, 2026-08-02) — design leads and schema follows + women-only scope refinement

1. **Standing principle: the schema, actions and data model for user-facing
   behavior track what Maor designs in Figma — not the other way around.**
   Your settled registration model STANDS as the server-side mechanics (it
   encodes mechanics Maor approved: admin approval, Paybox, refunds,
   emailed rejection reasons). But the screens' states and controls follow
   his Figma as he finishes the meetup components — and where his FINAL
   design differs from the model, the schema aligns to the design; if that
   creates a real mechanical conflict, raise it as a specific question
   rather than keeping the DB shape silently. His meetup RSVP component is
   still in progress; treat the Figma as the binding spec even before it is
   complete. (Mediation note, agreed context: a UI may collapse or rename
   backend states — DB richer than UI is fine; the one semantic point still
   open is what "מתעניינת" maps to in his final component — the
   heart/favorites split remains the default unless his design says
   otherwise.)
2. **Women-only — scope refinement for the new Platform Policies story:**
   the women-only rule applies AT THIS STAGE and to the MOM side of the
   platform. **Professionals (providers) may be men.** There is no provider
   self-registration in MVP — you register providers manually and manage
   their pages — so provider records are admin-curated and
   gender-unrestricted. Please reflect this in the story/ToS wording and in
   the lawyer brief: user signup = women only for now (right reserved to
   change later); provider directory = curated, not gender-limited.

## 📞 CALL OUTCOMES (Maor + Sivan, 2026-07-30) — session-persistence task for you + a status correction on the two design tasks

> A dated call log now lives at `planning/call-log.md` — summaries of every
> Maor↔Sivan working call.

1. **⛔ NEW TASK (Sivan, agreed on the call) — onboarding session
   persistence + resume.** Configure auth cookies/session lifetime to keep a
   mom signed in as long as possible, and make onboarding RESUMABLE: if she
   starts and stops mid-way, she continues from the step where she stopped —
   never from scratch. That means persisting per-step progress
   (the target model's `onboarding_progress` table is the natural home:
   current_step + completed_steps) and routing a returning session straight
   to the saved step. Applies to the phone-OTP session from the very first
   step.
2. **Status correction on your two Notion design tasks:** per Maor, most of
   it already exists in Momlee 2.0 — the phone/OTP screens are the reworked
   `02_-06_` onboarding frames, and the verification INTRO is built. His
   actual remaining work, replacing those task briefs: polish the
   `Birth List` component and design the **Didit failure states** (the
   five-state return screen you asked for is inside that). Pull the file and
   check whether the existing frames satisfy the login task — if something
   is missing for web, list it concretely.
3. **Working principle (recorded in `design-system/figma.md`):** screens are
   representative; **full scenario coverage lives in the COMPONENT variants
   inside them.** When a screen seems to miss a state, check the component
   set before asking.

## 📌 DECISION (Maor, 2026-07-30) — Lucide is the OFFICIAL icon package + the spinner behavior contract

1. **Lucide is Momlee's official icon library from now on.** The curated set
   lives on the Momlee 2.0 `↳ Icons` page as stock 24x24/stroke-2 components.
   Consumption rule when icon work resumes: install `lucide-react` /
   `lucide-react-native` at a PINNED matching version and map Figma
   `lucide/x` names to package exports — NEVER export package icons as SVG
   (this also retires most of the `@svgr` question; svgr remains only for
   genuinely custom glyphs). Directional icons are consumed ONLY via the
   Icon primitive's semantic forward/backward mapping (RTL: forward =
   arrow-LEFT); tint via currentColor from text tokens, never baked color.
2. **`lucide/loader-circle` is the canonical loading spinner**, and its
   Figma annotation is a binding behavior contract: always animated while
   visible — rotate 0→360deg clockwise, 1s, linear, infinite; animate the
   ICON, not its container; start on render, stop/remove when loading ends
   or on unmount; the static Figma appearance is design reference only.
   Web: `animation: spin 1s linear infinite`. Native: Reanimated/Animated
   equivalent, same numbers.
3. Reminder codified for every dev session: **read the node's annotations,
   not just its structure** — behavior contracts live there
   (momlee-figma-first already mandates it; this decision is proof of why).

## ⛔ ACTION (Sivan, 2026-07-29) — give Maor design access to Mapbox + gate fixed + your decisions acknowledged

1. **Open Mapbox access for Maor.** He is about to design the app's maps to
   match the Design System (map style, markers, pins, popups). Please give
   him access to the Mapbox account/Studio (invite his email as a member
   with Styles permissions on the MomLee account). Ties into your pre-launch
   note about the unrestricted preview token — same account housekeeping.
   His task "design the maps per the DS" is queued on his side behind it.
2. **Your `check-web-arch` finding — fixed and pushed** (`1cadb2a` on
   `momlee-web`): the gate now matches real Supabase usage (imports /
   client-call shapes), not the bare word, so comment mentions no longer
   count; re-baselined 28 → 27 (one grandfathered file was comment-only).
   Good catch, and the right call to leave the fix out of your feature
   branch.
3. **Your phone-OTP-only decision (no Google/Apple/Facebook, because any
   social login obligates Sign in with Apple on native) — acknowledged and
   recorded.** It closes the Google question Maor delegated to you, and the
   Apple-obligation argument is exactly the kind of reasoning that should
   decide it. The two login screens are on Maor's list as a priority — he
   knows the server side is waiting on him.
4. Staging credentials received; Maor will get the production test-number
   code from you directly (correct call not to commit it).

## 📞 CALL OUTCOMES (Maor + Sivan, 2026-07-28) — onboarding decisions, DS handoff BEGINS, and one schema↔design mismatch to resolve

**Decisions locked on the call (logged in the Dev Changelog):**

1. **Family-status step (your request — accepted).** The onboarding question
   about the mom's current situation becomes 4 options: `בהריון` /
   `בחופשת לידה` / `אמא מנוסה` / `אחר / מעדיפה לא לומר`. Maor redesigns the
   Figma step accordingly. PROPOSED enum keys (Maor to lock in the glossary
   before any migration): `pregnant | maternity_leave | experienced |
   undisclosed`.
2. **Location step CONFIRMED in onboarding** — the mom picks her "Sweet
   Spot": the area where she wants to see meetups around her. Maor adds the
   step to the Figma flow. (Data-wise this lands on the mom-profile location
   fields from the target model — region + lat/lng; not a migration yet.)
3. **Verification provider DECIDED: Didit** (closes the old Didit-vs-Persona
   open decision). The verification step comes right after family-status.
   The verification intro screen gets a reassurance note (via the
   `Onboarding Step / Note` cases component): we do not collect the photo or
   sensitive data; at the end of verification she is returned to Momlee.
4. **THE DS HANDOFF BEGINS (partial freeze lift).** Maor starts transferring
   to you, piece by piece, whatever is READY in Momlee 2.0: Variables, Text
   Styles, screen components. Intake protocol: variables land via the
   sync-tokens flow (one deliberate sync per drop, web+native); components
   resume ONLY for handed-off items (the general freeze stays for the rest).
   **ACTION for you: reply with your preferred intake format/cadence** (e.g.
   one drop per week? per flow? a checklist page in the file?).

**⚠️ Schema ↔ design mismatch found while preparing (resolve BEFORE meetup
screens are designed/built):** the live `meetup_attendees.status` is
`asked_to_go | going | cancelled` (your 2026-07-19 fix removed `interested` -
heart = favorites). But the Figma `Meetup RSVP` component has
`Status=Idle/Interested/Going`. Either the UI drops "מתעניינת" (and the
heart/favorite carries that intent), or `interested` returns as an attendee
status. Maor+you need to pick one; everything downstream (RSVP chips, admin
registrations, capacity counting) depends on it.

**Live Pro-Meetup field reference (pulled from production today — design
will follow it):** `meetups`: title, description, meetup_date, meetup_time,
location_address + lat/lng + google_place_id, meetup_type mom|pro, status
open|full|completed|cancelled, capacity 2-100 (nullable), price 20-180 ILS
(REQUIRED for pro, forbidden for mom), payment_link (REQUIRED for pro),
provider_id → thin `providers` (name, photo_url, description). Attendee:
status asked_to_go|going|cancelled, payment_status pending|paid|cancelled.
Notable: there is NO meetup cover-image field — the visual comes from the
provider photo / creator avatar. If the design wants a per-meetup image,
that is a schema request, not a given.

## ✅ RULINGS (Maor, 2026-07-21) — rename ratified · login model locked · Google = YOUR call · providers reflected

Answers to everything from your 2026-07-19→21 push. First: excellent run —
the meetups core + preflight + pipeline discipline were exactly the Plan C
directive executed well, and your worklog rows now land straight in the Dev
Changelog (the Notion loop is officially closed).

1. **`baby_meetups` → `meetups`: RATIFIED.** Right call, right timing
   (pre-launch = the one free window), and it matches the target model. The
   glossary is updated accordingly. Two notes: (a) the frozen `momlee-native`
   branch still references `baby_meetups` — logged as a revival-time task,
   nothing to do now; (b) process: this contradicted a written line in the
   Plan C directive ("do NOT rename live tables") — you were right on
   substance, but when a step contradicts a written directive, flag it BEFORE
   merging (a one-line note in the log is enough). The directive line itself
   was stale and is hereby superseded: renames during the re-baseline are
   allowed as flagged decisions.
2. **Web login — DECIDED (Maor + Sivan by phone, 2026-07-21), supersedes the
   2026-07-21 from-sivan proposal:**
   - **Phone + SMS OTP is the ONLY initial signup/auth method.** One screen,
     one field, the native flow reused. No email login at signup, no
     Facebook (your drop confirmed).
   - **Email is collected AFTER onboarding** (profile-completion step), as a
     value proposition ("account recovery if you change your number +
     updates"), verified via magic link, nudged until verified —
     **eventually required**. Once verified it becomes the recovery channel
     (changed number → email link → update phone) and a secondary login.
   - **Marketing consent is a SEPARATE opt-in checkbox** next to the email
     field + consent timestamp (Israeli spam law, Amendment 40 — without it
     the email is operational-only). Schema additions when you get there:
     `email_verified_at`, `marketing_consent` + timestamp.
   - **Google sign-in — Maor delegates this to YOU, with his framing:** its
     relative value right now is marginal (signup is phone-only anyway, so
     Google only speeds up RETURNING login, mostly desktop; and it brings
     real identity-merge complexity - phone-created account + Google email
     identity). What is SUPREMELY important to him is **email collection**
     - Welcome email, marketing, activation CTAs - which the
     profile-completion step delivers regardless of Google. Decide
     whichever way you judge best and log it; either answer is accepted.
   - Figma: no new design needed - the phone screens exist from native; the
     web login simply loses the social buttons (pending your Google call).
3. **Thin `providers` table — acknowledged and reflected.** Maor reviewed
   the coexistence plan (thin MVP record ↔ frozen `provider_profiles`, one
   concept at two fidelities) and it is now codified in
   `knowledge/glossary.md`. OS Entities/glossary registries follow.

## 📌 DECISION OF RECORD (Maor, 2026-07-19) — the app font is now Google Sans (reverses Noto Sans Hebrew). No action for you yet.

Google released **Google Sans on Google Fonts under SIL OFL 1.1** (2026 —
after older docs/skills were written; verified: official download's OFL.txt +
the live specimen page). The released variable font (GRAD/opsz/wght)
**includes full Hebrew** (all alef-tav, verified in the TTF cmap) — there is
no separate "Google Sans Hebrew" family. The Momlee 2.0 Design System file
is built in it, and it is now the official app font.

- **No code change now.** Web stays on self-hosted Noto until the DS-closure
  token sync — the font flips then (next/font local files for web, static
  TTFs for native), through the `fontFamily.sans` role token as always.
  Plugin docs/skills that still say "Noto Sans Hebrew is official" are
  superseded by this note and will be updated in that same sync.
- **Figma caveat:** Figma's cloud font service has not synced the family yet,
  so it renders only where the font files are installed locally. When you
  return to UI work, install the OFL download (fonts.google.com → Google
  Sans, install the `static/` TTFs incl. the 17pt set) or the file will show
  missing-font warnings.

## 🚀 TOP PRIORITY (Maor, 2026-07-16) — Plan C APPROVED: re-baseline the product-core schema. This is the next block of work.

Maor has formally approved **Plan C**. Everything Plan-C-related is now the
priority track — it outranks the remaining component/icon items (those stay
parked below and resume later).

**WHY (the audit conclusion, in short).** The live DB is **pre-launch** (seed
data only) and **provider-heavy**: the provider / verification / admin /
audit / RLS subsystem is mature and is **KEPT as-is**. The product core —
meetups, organizations, subscriptions/entitlements, messaging, role/status
enums — is barely modeled or missing entirely. Patching it incrementally
would drag the old shapes forever; a full rewrite would throw away a good
backend. Plan C is the middle path: **keep the backend spine, cleanly
re-baseline the product core on top of it, with the Momlee OS as the spec —
now, while migration cost is near zero.** It also unblocks the tasks marked
"blocked on Plan C" (meetups queries/data layer) and gives every future
screen real tables to stand on.

**WHAT it requires (scope).**
1. **Preflight:** re-verify baseline == live (last verified 2026-07-02),
   regenerate `packages/supabase/src/database.types.ts` from live, confirm
   migration bookkeeping is clean before the first new migration.
2. **Reconcile the Mismatch rows** in the OS **Database Tables registry**
   (the 41-row honest mirror — it is the living map of this work): e.g.
   `baby_meetups` gains `meetup_type` / `capacity` / `price` / `status`;
   `meetup_attendees` gains `attendance_status`; enum hardening
   (`user_roles.role` TEXT → `app_role` enum — flagged top risk).
3. **Create the Planned tables** per the target model: organizations +
   `organization_members` (D1: IN MVP, M:N), subscriptions/entitlements
   (**D6 is still open — MODEL ONLY, implement no entitlement values**),
   messaging/conversations, `onboarding_progress`, notifications family,
   reports/safety. Deprecations (forum, provider CRM, dead mocks) per the
   same doc.
4. **After every landed migration:** regenerate types + flip the affected
   Database Tables registry rows to `Aligned`. The registry must stay honest.

**HOW (rules of engagement — all existing gates apply).**
- Spec sources in precedence order: **OS registries (Database Tables /
  Entities / Schema Registry) → `knowledge/target-data-model.md` (attached
  to the plugin today) → ask Maor.** Never invent a column, enum value, or
  behavior. Where the target model marks an open question, that is a task
  for Maor, not a guess.
- Work **area by area** (suggested order: meetups core first — it unblocks
  the queries task — then organizations, then subscriptions modeling, then
  messaging). For each area: draft the migration(s) + a short schema note in
  from-sivan/worklog BEFORE merging, so Maor can veto shapes cheaply.
- Every change ships as a **migration through the pipeline** (merge to
  `momlee-web` → green checks → `db-deploy`); Migration Gate applies in
  full (rollback + RLS impact + affected tables/APIs); destructive changes
  need the in-file `-- destructive-approved:` marker as usual.
- **RLS by default** (owner-or-admin, explicit public-read carve-outs only),
  soft-delete by default, `user_display_info` pattern for any user-facing
  projection, **do NOT rename live tables** (`baby_meetups` stays; product
  naming lives in code/UI). Privacy: DATA_INVENTORY updated per area.

## ✅ RULING (Maor, 2026-07-15) — your ADR-016 gate fix is APPROVED as PERMANENT (+ your Windows papercut is fixed)

Answers to today's tasks:

1. **`isAdr016PlatformPair` is the rule now, not a temporary unblock.** Maor
   confirmed: the gate simply predated ADR-016. Your implementation was kept
   exactly as-is (one native + one web-primitives file, same name/node; any
   3rd claim or web-vs-web dup still fails). We removed the TEMPORARY/revert
   comments and pushed to `momlee-web` (`2b20520`). Nothing for you to change.
2. **The Windows path papercut is fixed in the same commit:** the three gates
   that compare relative paths against forward-slash literals
   (check-component-dupes, check-web-arch SANCTIONED, check-figma-refs
   grandfather list) now normalize `path.sep` — your local runs should match
   CI. Pull `momlee-web` before your next gate run.
3. **`Buttons/Social button` node ids — received from Maor (2026-07-16) and
   PARKED.** `kind='social'` is not needed by any web screen yet, and Plan C
   (above) is now the priority track — the node ids will be posted here when
   component work resumes, so nothing will block you then.
4. **`@svgr/webpack` + the icon-library strategy + the Figma plan/seat
   question — PARKED behind Plan C** (Maor is weighing a broader decision:
   adopting one large published icon library in Figma as canonical vs
   growing the hand-curated set). The inlined-paths workaround stays
   acceptable meanwhile; no web screen work depends on it right now.

## ✅ SOLVED (2026-07-15) — the Dev Changelog mystery: it was ARCHIVED, now restored INTO Momlee OS. Log to Notion again (same id!)

Your retraction was excellent detective work and correct on every check —
and there was one missing piece neither of us had: the Dev Changelog lived
under **"Momlee Old" → 08 · Delivery**, and when the old command center was
archived after the OS pivot, the changelog was archived WITH it. That
reconciles everything:

- Your side: Notion search does not index archived pages, and fetching an
  archived id 404s on your surface → "the DB does not exist". Correct
  observation, wrong conclusion.
- Claude's side: the id kept resolving read/write on Maor's session surface,
  so rows kept landing there daily → "it's alive". Also correct, and blind
  to the archived flag.
- Nobody deleted data: all rows are intact, including your two synced
  worklog entries.

**Fix applied (Maor's call, 2026-07-15):** the database was MOVED to
**Momlee OS → 07 - Engineering**, which un-archived it. The id is
**unchanged**: `ee6d4bbb-1444-479c-b818-36f7e3951988` — so the worklog
skill, the hook, and the standup skill all keep working as-is, no edits
needed. Your decision to keep logging to this file rather than create a
second DB was right; now please: **verify you can fetch the changelog, then
sync your pending Worklog entries below into it as rows and log directly
from now on** (git fallback stays for outages only).

## ✅ RULING (Maor, 2026-07-15) — Button UNBLOCKED: bg-brand-solid drift resolved at the source

Answers to your two questions from the web-primitives worklog entry:

1. **The light-pink retint was NOT final.** The Figma was mid-edit; Maor has
   reverted the color in the file. VERIFIED against live Figma (node
   `3287:428579`, 2026-07-15): `bg-brand-solid = #b05f64` and the label is
   `text-white` again — Figma, `design-system/tokens.md`, `@momlee/tokens`,
   and the native Button all agree. No token re-sync needed.
2. **Primary button text = white** (pairs with the mauve), same as native.

So: **build the Button now, figma-first, on the TOKEN classes**
(`bg-brand-solid` + `text-white`, never a hex), mirroring the native
contract per ADR-016. Maor is still running design-system experiments in
Figma — if you ever catch a drift like this again, same protocol: stop,
log, ask (this catch was exactly right). When his palette work closes for
real, we'll announce it and run `momlee-sync-tokens` as one deliberate
sync (web + native together) — do not chase intermediate Figma states.

## ✅ DONE (2026-07-15, per Sivan) — Notion MCP reconnected to Maor's workspace (was: ACTION REQUIRED 2026-07-14)

Sivan confirmed she reauthenticated and picked Maor's workspace, and task
reads now resolve. Definitive proof pending: the NEXT worklog entry should
write straight to the Dev Changelog (no from-sivan.md fallback). If it
404s again, the step-by-step guide is preserved below.

## ~~⛔ ACTION REQUIRED (Sivan, ~2 minutes) — reconnect the Notion MCP to MAOR's workspace (2026-07-14)~~

Your "Dev Changelog is gone" 404s are solved: the DB is alive, you ARE a full
member of Maor's workspace (verified) — **your Notion MCP is simply authorized
against the WRONG workspace** (your personal one), so every Momlee OS id 404s.
Per Notion's docs the MCP "acts with your full Notion permissions" in the
workspace you authorize — so authorize Maor's. Step by step:

1. In Claude Code run `/mcp` → select the **Notion** server → **Disconnect /
   Logout** (clear the current auth).
2. Same `/mcp` menu → **Authenticate / Reconnect** → a browser window opens
   with Notion's consent screen.
3. **THE CRITICAL STEP:** at the TOP of that consent dialog there is a
   workspace dropdown showing which workspace you are granting. It defaults
   to YOUR personal workspace — **switch it to Maor's workspace** (the one
   holding "Momlee OS"; you're a Member there as sivan@applee.dev).
4. Approve ("Allow access").
5. **Verify:** back in Claude Code, ask Claude: *"fetch the Momlee OS page"*
   or run a standup — if the Dev Changelog resolves, you're on the right
   workspace. If it still 404s, repeat step 2 and double-check the dropdown.

From then on the worklog writes straight to the Dev Changelog (no git
fallback needed), and the standup skill can read your open tasks.

## ✅ DONE (2026-07-14) — GitHub invitation accepted; from-sivan.md is flowing (was: ACTION REQUIRED 2026-07-13)

Root cause found: your push to the plugin repo failed because **the
collaborator invitation to `sivan@applee.dev` is still Pending — you never
accepted it** (Maor verified in the repo settings). Until you accept, every
push to `Maorsnaim/momlee-guide` is rejected, so your from-sivan.md entries
exist only on your machine.

1. **Accept the invite:** check the GitHub invitation email to
   `sivan@applee.dev`, or open
   **github.com/Maorsnaim/momlee-guide/invitations** while logged in to your
   GitHub account. (Invites expire after 7 days — if it expired, tell Maor to
   re-send.)
2. **Deliver the stranded entries:** from your momlee-guide clone (not the
   plugin cache folder):
   ```
   git add planning/from-sivan.md
   git commit -m "from-sivan: worklog + tasks for Maor"
   git push origin main
   ```
3. **Verify:** `git log origin/main -1 -- planning/from-sivan.md` shows your
   commit. From now on the worklog skill requires commit+push+verify in the
   same step, so a silent failure like this cannot recur.

## ⛔ ACTION REQUIRED (Sivan, one-time, ~2 minutes) — add the deploy secret (2026-07-08)

The new DB pipeline (below) cannot reach Supabase until you add the repo
secret — only you have the access:

1. supabase.com → avatar → **Account → Access Tokens → Generate new token**
   (name: `momlee-ci-deploy`; copy it — shown once).
2. GitHub `sivanhasson/MomLee` → **Settings → Secrets and variables →
   Actions → New repository secret** → Name: **`SUPABASE_ACCESS_TOKEN`**,
   Value: the token.

Until then the `db-deploy` job fails on auth and migrations stay pending
(harmless, but nothing deploys).

## 🆕 NEW skill: `/momlee-standup` — session-start updates from git + Notion (2026-07-08)

Per Sivan's request: opening a session and asking **"מה חדש?" / "תעדכן
אותי" / "standup"** now pulls open work from BOTH channels — this file
(git) AND live Notion (Dev Changelog open rows + the Momlee OS Tasks DB,
including Blocked tasks) — and returns one short prioritized Hebrew summary.
Refresh the plugin and it is active (restart the session so hooks/skills
load).

## 🌿 Web MVP work happens on the NEW `momlee-web` branch (2026-07-08)

**Sivan: `git fetch && git checkout momlee-web`.** The branch was cut from the
tip of `momlee-native` (so it carries the migration baseline, the security
fixes, the CI gates, and fresh generated types — branching from
`improved_momlee_by_claude` would have lost all of those). `momlee-native`
stays frozen as the intact native reference for the future iOS return.

**Fact-check (Sivan asked): the plugin does NOT block web development.** The
plugin itself ships only the worklog hook. The CLI gates live in the app repo,
and until today they only scanned `apps/mobile` + `packages/*` — web was
UNCOVERED, not blocked (CI even excludes web lint explicitly). What changed on
`momlee-web` (commit `032cb39`): the gates now cover web too —

- **check-naming** scans `apps/web/src` (glossary synonyms, auto Figma names).
- **check-deps** enforces `apps/web/package.json`; the existing web stack
  (Next.js / Radix / Mapbox / testing, 60 packages) is allowlisted as the
  approved web set; NEW dependencies still require the DEPENDENCY GATE.
- **check-figma-refs**: every NEW web `page.tsx` declares its Figma source
  node (`// figma: <nodeId>`); the 32 pre-pivot pages are grandfathered until
  rebuilt from Figma.
- **NEW check-rtl-web**: a logical-properties RATCHET — the count of physical
  Tailwind direction classes (`ml-/mr-/pl-/pr-/left-/right-/text-left/...`) in
  `apps/web/src` may only DECREASE (baseline 543). New code uses logical
  utilities only (`ms/me/ps/pe/start/end/text-start/text-end`). When you lower
  the count, lower the BASELINE in the same change.
- `ci.yml` runs on pushes/PRs to `momlee-web` as well.

**Web gates round 2 (2026-07-08, commit `53cad5f`) — three more mechanical
gates, closing the full contract on web:**

- **check-component-dupes** — the mechanical half of the REUSE AUDIT: no two
  component files may share a normalized base name (3 legacy dups
  grandfathered: button / input / serviceslist), and no NEW primitive may
  enter an existing synonym family (Sheet/Modal/Drawer/Dialog · Badge/Chip/Tag
  · Toast/Snackbar · Select/Dropdown/Picker · Avatar · Tooltip) in
  `components/ui`, `components/shared` or `packages/ui` — extend the existing
  primitive (13 stock shadcn primitives grandfathered).
- **check-web-arch** — Supabase lives ONLY in the data layer
  (`services/`, `lib/`, `app/api/`, `app/actions/`): files touching it
  elsewhere are ratcheted (baseline 28, may only go down — move queries into
  services as you touch them); `select('*')` ratcheted (baseline 25);
  **direct PostHog is hard-banned (0)** — analytics only via the
  `@momlee/core` wrapper.
- **check-tokens-web** — hardcoded color literals (arbitrary Tailwind color
  classes `bg-[#...]` + `style` hex) ratcheted (baseline 101, may only go
  down) — new code uses design-system token classes only.

Ratchet rule reminder: when your change LOWERS a count, lower the matching
BASELINE constant in the same commit (the gate prints the number).

## 🏛️ ADR-016 (2026-07-08, Maor decided): per-platform primitives with a SHARED CONTRACT

Three facts every UI session must know:

1. **The existing web UI is a DEMO, not the product.** Every screen gets
   rebuilt Figma-first (that is why the 32 pages are grandfathered in the
   figma-refs gate). Do not extend demo components as if they were canonical.
2. **One design system, one token source.** The real web runs on
   **`packages/tokens`** (Figma-sourced; the same package that feeds native)
   and **Noto Sans Hebrew** (Heebo is wrong and gets removed). A [GAP]
   Validator tracks wiring the preset into the web Tailwind — until that
   task closes, treat any stock-shadcn value as demo debt.
3. **native<->web reuse = a SHARED CONTRACT, not shared render code**
   (universal/react-native-web was rejected): each primitive has a native
   implementation (packages/ui, RN+NativeWind) and a web implementation
   (apps/web components/ui, DOM+Radix+Tailwind) with the **same name, same
   props API, same Figma node, same tokens** — one row in components.md
   covers both. Full rationale: ADR-016 in the OS; the work: Feature
   `platform.web_design_foundation` (Critical) + 4 tasks.

**Round 2.2 (2026-07-08, per Maor): Figma node identity — one node, one
component.** A Figma screen is composed of nested component instances; the
danger is re-implementing a nested sub-component that already exists. Now:

- Every NEW component file declares its source node (`// figma: 17297:8153`;
  non-visual files: `// figma: none - <reason>`) alongside the reuse-audit
  receipt — CI fails it otherwise.
- **One Figma node = one code component**: a second file claiming the same
  node FAILS CI (this catches re-implementations even under a new name —
  verified against the real inventory: the native `Input` already carries its
  node, and a synthetic second claim was caught).
- Before building a SCREEN, Claude must enumerate its component instances
  from the Figma metadata and resolve each node against the code
  (`grep "figma: <node>"` + components.md): mapped → IMPORT (inline
  re-implementation is forbidden); unmapped → full REUSE AUDIT → CREATE with
  both headers. See momlee-figma-first step 5.
- Code Connect mappings (Figma MCP) added as a task under
  `platform.web_design_foundation` — once mapped, the design context itself
  hands Claude the existing component.

**Round 2.1 (2026-07-08, per Maor): the reuse gate is now SEMANTIC, not just
name-level.** Every NEW component file (not in the pre-pivot inventory of
185) must carry a **reuse-audit receipt** header proving the semantic REUSE
AUDIT ran before creation:

```
// reuse-audit: searched="badge, chip, pill" checked="ui/badge" verdict=CREATE reason="animated numeric pill not covered"
```

Claude runs the three searches (components.md + Figma + code, names AND
synonyms, by FUNCTION not just name), decides REUSE -> EXTEND -> CREATE, and
records the receipt; CI fails any new component without it. Writing a receipt
without running the audit violates momlee-design-system.

**DB changes — the pipeline is LIVE (2026-07-08, Maor approved).** You CAN do
everything yourself now, end to end — **no local DB required**:

1. Author a migration file in `supabase/migrations/`, commit, push/merge to
   `momlee-web`. That's it.
2. **CI tests it on a fresh database FOR you** (the `rls-tests` job spins up
   a clean Postgres in Docker, applies ALL migrations from scratch, runs the
   pgTAP invariants). Running `supabase db start` locally is optional — a
   faster feedback loop if you want it, never a requirement.
3. After `checks` + `rls-tests` are green, the **`db-deploy` job applies the
   new migrations to the LIVE database automatically** and prints
   `migration list` for verification.
4. **Nobody runs `supabase db push` against live by hand — the pipeline is
   the only path.** Standard prod discipline; the same rule binds Maor and
   Claude.
5. **Destructive migrations** (DROP TABLE/COLUMN, TRUNCATE, DELETE FROM) are
   paused by a guard until the file carries
   `-- destructive-approved: <your-name> <date> <reason>`. **You approve it
   yourself** — it is not a permission request, it is a deliberate stop,
   because destructive SQL deletes data with NO undo. Before adding the
   marker ask: is there a backup? is the data truly unneeded? would a
   soft-delete or rename-and-deprecate be safer? When unsure, consult Maor —
   but the call is yours.

**One-time setup (Sivan — you have the repo-settings access):**
1. Create a Supabase personal access token: supabase.com dashboard → your
   avatar → **Account → Access Tokens → Generate new token** (name it e.g.
   `momlee-ci-deploy`).
2. In GitHub `sivanhasson/MomLee` → **Settings → Secrets and variables →
   Actions → New repository secret**: name **`SUPABASE_ACCESS_TOKEN`**,
   value = the token.
Until the secret exists, the `db-deploy` job fails on auth and migrations
simply stay pending — nothing breaks.

## ✅ RESOLVED 2026-07-02 — migration baseline squash + repair (was ⛔ since 2026-06-22)

**The `supabase migration repair` was EXECUTED and verified on 2026-07-02
(Maor + Claude).** Live bookkeeping now equals exactly the 2 baselines
(`migration list` local==remote, `db push --dry-run` = "Remote database is up
to date", direct table read = 2 rows, data untouched). **Sivan: nothing to run
anymore** — just `git pull` on `momlee-native`; future `db push` is safe. Full
detail + rollback backup: `docs/planning/MIGRATION_BASELINE_HANDOFF.md` in the
app repo. The original context is kept below for history.

## ~~⛔ ACTION REQUIRED~~ — migration baseline squash (2026-06-22)

The app-repo migrations were **squashed to a baseline** because the old 85
incremental migrations could not rebuild a clean DB from scratch (the
Lovable-origin tables `users` / `parent_profiles` / `provider_profiles` / the
`provider_groups` family had no `CREATE TABLE`, so `supabase db start` failed).
Old migrations are archived in `supabase/_archived_migrations/`; two baselines
(`20250101000000_baseline_public_schema.sql`,
`20250101000001_baseline_storage_policies.sql`) now equal the live schema.

**Sivan: before the next `supabase db push` you MUST run `supabase migration
repair`** (bookkeeping only — it does NOT change the live schema), otherwise the
push will try to re-create objects that already exist on live. Exact commands +
rollback + verification: **`docs/planning/MIGRATION_BASELINE_HANDOFF.md`** in the
app repo. No live database changes have been made.

> **Reaffirmed 2026-07-01 — STILL PENDING.** Sivan has not yet run
> `supabase migration repair`; it remains a **hard prerequisite before any
> `supabase db push`**. It also aligns with the direction we're leaning (a clean
> schema re-baseline — see the "Notion OS" note below): the squash already
> produced clean baselines that equal live, so the repair is the bookkeeping step
> that makes the clean baseline and the live DB agree.

> **PRE-FLIGHT DONE 2026-07-02 — the handoff commands were INCOMPLETE; use the
> UPDATED doc.** A read-only verification against live confirmed the baseline
> equals the live schema object-for-object (tables/policies/views/triggers/
> functions/enums/indexes all reconcile — safe to repair). BUT live bookkeeping
> holds 91 versions vs 85 archived files: **nine provider_groups versions
> (20260602100000..20260604120000) have no local file**, so the original repair
> loop leaves them dangling and `db push` would still complain. The app-repo
> handoff doc (`docs/planning/MIGRATION_BASELINE_HANDOFF.md`, commit `537a9a7`)
> now carries the corrected sequence: step 3b (revert those 9), a final
> `supabase db push --dry-run` gate ("Remote database is up to date" or STOP),
> and a rollback recipe from
> `docs/planning/live_schema_migrations_backup_2026-07-02.csv` (a backup of all
> 91 live bookkeeping rows). **Sivan: `git pull` on `momlee-native` first** (Maor
> pushes it), then follow the UPDATED section 3 end-to-end. Repair edits
> bookkeeping only — it cannot touch schema or data.

## 🧭 Notion OS — now the planning source of truth (2026-07-01)

The product is planned in a structured **Notion "Momlee OS"**, documented in this
plugin under **`knowledge/os/`** (`00-foundations` → `90-dispositions`). Read
`00-foundations.md` first.

**Structure (the spine):** **Epic → Feature → Story → Task**, with registries
hanging off Stories — Permissions, Roles, User Types, Eligibility, Events,
Automations, Product Rules, Schema Registry, Entities, Communication Channels,
Message Categories, Design Screens / Components. A Story carries a small
human-authored core; Claude proposes the wiring; the approved result compiles to
an unambiguous spec for dev.

**Recent structural work (2026-06-29 → 07-01), all reflected in the book:**

- Epic **boundary rules LOCKED** (one capability = one owning Epic; surface ≠
  ownership; actor ≠ ownership; lifecycle boundary).
- New Epics: **Profile & Settings** (ongoing profile + cross-actor settings
  layer), **Browse & Discovery** (home / index / discovery layer; renamed from
  "Home / Discover"), **Provider Profile & Services** (Post-MVP stub reserving the
  professional-profile domain).
- MVP decision: meetup discovery = a simple index scoped to a **single experiment
  neighborhood**, **no proximity ranking** (Post-MVP; per Sivan's experiment).
- Naming locked to the **live code terms** for keys: **`parent` / `provider`**
  (not mom / professional). Names stay Mom / Professional for UI.
- Conventions: Decision Log titles in English + bilingual bodies; one decision per
  row; canonical rows first; no emojis / em-dashes; Hebrew-lead everywhere except
  Decision Log titles.
- Cleanups **verified**: Permissions + Privacy Notes DBs are dedup-clean.
  **UI-only leftovers (do in the Notion UI):** delete the 4 deprecated
  Communication Channels rows (marked "- delete"), remove the stray empty data
  source inside the Stories DB, remove the `.is` auto-links in Eligibility.

**⚠️ OPEN architecture decision (pending Maor's final go) — "Plan C":** a broad
audit (live DB + governance docs + client code + the OS model) concluded the live
DB is **pre-launch** (seed data only) and **provider-heavy**: the
provider / verification / admin / audit / RLS subsystem is mature and worth
**KEEPING**, but the product core (meetups, organizations, subscriptions,
messaging, role / status enums) is barely modeled and effectively greenfield.
Recommendation: **not a full rewrite and not incremental patching, but a clean
schema RE-BASELINE** of the product core on top of the kept backend, driven by the
OS as the spec — done now while pre-launch (near-zero migration cost). **Not
started; awaiting Maor's decision.**

## 🔐 Security audit (2026-06-09) — STATUS

**Maor ran a security audit and FIXED the holes in code** (app repo, commit
`c432dad` on `momlee-native`, report: `docs/audit/SECURITY_FIXES_2026-06-09.md`
in the app repo). What was fixed:

1. Public storage buckets → **private + signed URLs**.
2. **CRITICAL:** a broad `users` RLS policy exposed every mom's email+phone →
   removed; safe view **`public.user_display_info`** (id, display_name,
   avatar_url ONLY) is now the only way to show users to other users.
   (Pattern codified in `../knowledge/security.md` §24.)
3. `.env` was tracked in git → untracked + `.gitignore` fixed.
4. 16 hardcoded Mapbox tokens → moved to env.
5. 6 legacy Edge Functions (service-role + open CORS) → **deleted from the repo**.
6. `/api/geocode` had no auth/rate-limit → now authenticated + 30 req/min.

### ✅ DEPLOYED TO LIVE (2026-06-12, via Maor's token)

The security migration (20260609200000) + onboarding_events (20260610120000)
+ phone-first signup (20260612210000) are APPLIED and VERIFIED on production:
buckets private, users RLS + user_display_info live, events table with RLS,
email/display_name nullable, phone-aware handle_new_user. A real OTP SMS
request returned 200. database.types.ts regenerated from live.

### ⚠️ Still PENDING from the original list

The code is fixed, but the fixes are not live until these run against
**production Supabase** (coordinate with Maor before touching live):

- [ ] Apply migration `20260609200000` to the **live** database
      (buckets + `users` RLS + `user_display_info` view).
- [ ] **Delete the 6 legacy Edge Functions from the Supabase dashboard**
      (they're gone from the repo, but still deployed and callable).
- [ ] **Rotate the exposed tokens** (Mapbox; any key that lived in the
      committed `.env`), update env/secret stores accordingly.

**Until then:** treat the live DB as if the holes are still open — don't build
anything that depends on the old public buckets or on querying `users` directly.
New code MUST follow the fixed patterns (signed URLs, `user_display_info`,
env tokens) so it's correct the moment the deployment lands.

## Machine enforcement: full map (2026-06-11, commits ce1147a/940a6eb/921f9b1)

Three layers now run on every push/PR in the app repo: **eslint** (12+ rule
families: architecture, RTL incl. raw-Text ban, tokens-only incl. font-weights
+ fontFamily + arbitrary px, permissions, analytics SDK + PII payloads + event
format, select('*'), dates) and **CI scripts** (deps allowlist, migration
RLS/retention/rollback, RTL manifest flags, naming auto-layer-names +
synonyms, secrets/.env/public-env, figma screen-node refs). All verified
positive + negative.

- [ ] **Backfill Figma node refs** on the 5 grandfathered screens (index,
      welcome, phone, otp, name): add `// figma: <nodeId>` with the REAL node
      ids (don't guess — pull from the Figma map), then empty GRANDFATHERED in
      scripts/momlee/check-figma-refs.mjs.
- [ ] **Consider eslint-plugin-react-native-a11y** (accessibility gate at lint
      level: roles/labels/touchables) — new devDependency, so it needs the
      DEPENDENCY GATE + allowlist + verification it works with the Expo flat
      config. Recommended next mechanical step alongside RLS tests in CI.

## Worklog is now MECHANICALLY enforced (2026-06-11) — hooks in the plugin

The plugin now ships Claude Code **hooks** (`hooks/hooks.json`, Node script):
commit in a MomLee repo → session marked worklog-pending → ending the turn is
blocked once until a Dev-Changelog row is logged (or the git fallback /
explicit "trivial" note). This is harness-level enforcement — independent of
the model's memory.

- [ ] **Sivan: after the next `/plugin install momlee-guide@momlee`, restart
      the session** so the hooks load (Claude Code asks to approve plugin
      hooks on install — approve). Node is required (already in your env).

## Ten resilience & boundary gates (2026-06-11, Maor) — NEW

A second wave of gates is live (plugin 0.13.0):

1. **Module Boundaries** — onboarding ↛ meetups ↛ subscriptions ↛ moms
   directly; cross-module = the other module's PUBLIC service only
   (architecture.md + momlee-react-native).
2. **Error Handling Standard** — every error: specific Hebrew user message +
   dev log (no PII) + taxonomy analytics event when relevant + recovery
   action. Never "Something went wrong" (new skill **momlee-resilience**).
3. **Four states** — Loading/Empty/Error/Success on every data screen, via the
   shared pattern; missing Empty/Error designs in Figma = blocked part.
4. **Device Permission Gate** — location/notifications/camera/photos/contacts
   ONLY via the `@momlee/core/permissions` wrapper; in-context requests;
   denial is a designed state (momlee-privacy).
5. **Feature Flags** — every large/sensitive feature behind `<feature>_enabled`
   (provider_subscriptions_enabled, mom_discovery_enabled,
   identity_verification_enabled); kill = flip flag, never delete code.
6. **Copy source of truth** — new channel `knowledge/copy-guidelines.md`
   (feminine second person, warm+specific, Figma word-for-word, sensitive
   areas need Maor); no invented user-facing text, ever.
7. **Authorization ≠ UI visibility** — hiding a button is not a permission;
   DB/RLS/service enforce even when the screen is hidden (momlee-security #5).
8. **Offline/Network failure** — OTP, meetup, message, upload, location all
   define failure behavior; input never lost; no auto-retry on writes;
   airplane-mode test before done (momlee-resilience).
9. **Upload Gate** — every upload declares size limit, type allowlist, privacy
   class, bucket, access policy, delete policy (+ EXIF/GPS stripping)
   (momlee-privacy).
10. **Deletion/Retention** — no new data model without retention/deletion
    behavior; SOFT delete by default; new mandatory Retention field in the
    Migration Gate (momlee-privacy + momlee-data-inventory + momlee-migration).

Pending decisions/tasks:

- [ ] **Maor: feature-flags kill-switch mechanism** (server-controllable —
      e.g. a Supabase config row vs build-time only).
- [ ] **Maor: account-deletion cascade map** — decide per data type (photos,
      messages, children records, meetups, verification refs): cascade /
      anonymize / retain — fill the retention column in data-inventory.md.
- [ ] **Maor: Figma designs for shared Empty/Error states** (the four-states
      pattern needs designed visuals before the shared component is built).
- [ ] **Sivan: scaffold `@momlee/core/permissions`** (wrapper like analytics)
      when the first permission feature lands.

## Naming Gate + Glossary + milestone audits (2026-06-11) — NEW

- New skill **momlee-naming**: fires before naming ANYTHING new (file, folder,
  component, hook, service, route, table, column, enum, analytics event) and
  before translating Figma layers to code. Glossary terms only, format per
  artifact kind, printed NAMING check for tables/components/events.
- New channel **`knowledge/glossary.md`**: the canonical terms (mom, provider,
  child, meetup, organization, subscription, verification) with the frozen DB
  mappings and FORBIDDEN synonyms. **No synonyms, ever** — a new entity term
  enters only by Maor adding it to the glossary.
- **Figma Layer Naming Guard** (inside momlee-naming): auto-generated layer
  names (`Frame 12`, `Group`, `Rectangle`, `Component 1`) are never copied to
  code — resolve via component set → annotation → parent context → glossary,
  or STOP and ask.
- **`/momlee-audit` is now MANDATORY before closing any milestone** (sprint /
  complete flow / release) — see momlee-worklog. Milestone isn't Done until
  the audit ran and High findings are fixed or explicitly accepted by Maor.
- Drift cleanup: the stale live reference to `UnderlineField` in momlee-rtl
  now reads `Input` (historic note only).

## Bind every session + retro-audit (2026-06-11) — NEW

Two new pieces close the enforcement loop:

- [x] **DONE (2026-06-11, commit `d60a34f` on momlee-native):** the session
      contract was MERGED into the app repo's existing `CLAUDE.md` as a
      "READ FIRST" section (the web-era guide below it was kept; the plugin
      wins on conflicts). Awaiting push by Maor. Every Claude session in the
      repo now reads the contract at session start.
- [ ] **Run `/momlee-audit` once** in an app-repo session (after updating the
      plugin) — full compliance review of everything built so far against all
      the new gates. Report only; fixes get approved by Maor first.

## Architecture & Naming Review Obligation (2026-06-11) — NEW skill

New skill **momlee-architecture-review**: during ANY task, existing
inconsistencies (naming conflicts, duplicate concepts, drift, legacy patterns,
architectural violations) must be surfaced — never silently ignored. Protocol:
continue on the canonical convention, no automatic refactors, no renames
without approval, document + recommend in the "Architecture Observations"
format (Issue / Impact / Recommendation / Priority), and log durable findings
to the worklog so Maor sees them. Consistency over perfection — but obvious
better directions get surfaced, not buried. No action needed beyond updating
the plugin.

## Core Technology Stack — Iron Law (2026-06-11, Maor) — READ stack.md

`knowledge/stack.md` now carries the full canonical stack rules: Expo Router
(file-based, never React Navigation directly), TanStack Query (centralized
query keys), RHF+Zod for every form, the backend flow (minimum acceptable:
Screen → Service → Repository; Screen → Supabase forbidden), the PostHog
wrapper (`analytics.track` ✅ / `posthog.capture` ❌), **expo-secure-store**,
Reanimated + Gesture Handler, Gorhom sheets, **Zustand (restricted list:
session/locale/direction/feature-flags/temp-onboarding/global-UI — never
server data)**, **date-fns**, **expo-location**, and the Dependency
Governance 4 questions (no approval = STOP).

- [ ] **Sivan: migrate the Supabase session storage to expo-secure-store.**
      The current client (`apps/mobile/src/lib/supabase.ts`) persists the
      session in AsyncStorage — now forbidden for tokens. Use a SecureStore
      storage adapter for the Supabase client. (Note: SecureStore values are
      capped at 2KB each — Supabase sessions can exceed it; the standard
      pattern is an AES-key-in-SecureStore + encrypted-payload-in-AsyncStorage
      adapter, e.g. Supabase's documented Expo recipe. Pick with Maor if
      unsure.)
- [ ] **Sivan: add `expo-secure-store`, `date-fns`, `expo-location`, `zustand`**
      as each becomes needed (all stack-approved now; Expo modules are Expo
      Go safe).

## State Management Guard + Dependency Budget (2026-06-11) — NEW, in momlee-react-native

**State has ONE home** (Maor's decision; TanStack Query + React Hook Form are
now stack-approved): UI state = local `useState`; server state = **TanStack
Query** in feature hooks (queryFn → service → repository); form state =
**React Hook Form** + Zod resolver; global client state = ONLY if nothing else
fits, per-store Maor approval (no Zustand-by-default). Never copy server data
into a client store.

**Dependency Budget:** don't add a dependency if the feature can be built in
under 100 LOC with the existing stack; every dependency prints a
`DEPENDENCY GATE` block (under-100-LOC test, justification, cost) and needs
Maor's approval + a `stack.md` row in the same change.

- [ ] **Sivan: add `@tanstack/react-query` + `react-hook-form` (+ Zod resolver)**
      when the first data-fetching/form screen needs them — both JS-only,
      Expo Go safe. Until then nothing to install.

## Analytics: PostHog via wrapper (2026-06-11, Maor's decision) + Analytics Gate

**Tool decided: PostHog** (supersedes the first-party-only plan from
2026-06-10). Everything goes through ONE abstraction —
`@momlee/core/analytics` (`analytics.track('otp_requested', {...})`) — with
`providers/posthog.provider.ts` as the only file importing the SDK. **Never
call PostHog (or any analytics SDK) directly from screens/components.** The
event taxonomy is STABLE (seed list in `knowledge/analytics.md`); payloads are
PII-free (`baby_age_range` ✅, `baby_birth_date` ❌).

Also live: skill **momlee-analytics** — every feature prints an
`ANALYTICS GATE` block at plan time (events / verification / success KPI
traced to WSMA) and isn't "done" until events are SEEN landing.

- [ ] **Sivan: scaffold the wrapper** — `packages/core/analytics/`
      (`analytics.ts`, `analytics.types.ts` typed taxonomy,
      `providers/posthog.provider.ts`), fail-soft without env key; migrate the
      existing `logEvent` console stub into it (same event names).
- [ ] **Sivan: add `posthog-react-native`** (now stack-approved — verify Expo
      Go compatibility on install; if it needs native code, defer the SDK to
      the EAS dev-client stage and keep the fail-soft console provider until then).
- [ ] **Maor: provide the PostHog project key + host** (`EXPO_PUBLIC_POSTHOG_KEY`,
      EU/US cloud choice) and update the privacy policy / store labels to name
      PostHog as a processor (data-inventory row already updated).

## Migration Gate (2026-06-11) — NEW, hard gate before any DB change

New skill **momlee-migration**: every database change (table/column, RLS
policy, enum, view, bucket, schema-touching Edge Function) must print a
`MIGRATION GATE` block BEFORE any SQL: migration file + rollback plan + RLS
impact + affected tables + affected APIs. New tables ship RLS in the same
migration; destructive changes need a backup step + Maor's approval; the live
DB stays Maor-coordinated; `database.types.ts` is regenerated before and after.
No action needed beyond updating the plugin.

## Component Reuse Audit (2026-06-11) — NEW, hard gate before any new component

"Reuse before create" is now a **proven audit**, not a guideline (upgraded in
**momlee-design-system**; figma-first rule #2 and /momlee-screen step 6 point at
it). Before creating ANY component, Claude must search `components.md` (both
tables) + the Figma inventory + the code, by name AND synonyms
(Sheet/Modal/Drawer, Badge/Chip/Tag…), print a `REUSE AUDIT` proof block, and
verdict REUSE → EXTEND → CREATE. Base primitives (Button, Input, Card, Avatar,
Badge, Sheet) are presumed to exist — a second one is a duplicate. A CREATE
verdict requires the component in Figma first + a row in `components.md` in the
same change. No action needed beyond updating the plugin.

## AI Prompt Guard (2026-06-11) — NEW, applies to everything

New skill **momlee-prompt-guard**: if information is not in an official source
(Figma, annotations, design-system/, knowledge/, planning/, or an explicit
instruction), it does not exist — STOP and ask; never invent. A missing
component (e.g. no Time Picker in the design system) means that part is
**blocked until Maor designs it** — build the rest, log the blocker. This is
also iron rule #4 in **momlee-figma-first**. No action needed beyond updating
the plugin — just know the rule is now binding for every Claude session.

## Architecture Gate (2026-06-11) — NEW, applies to all data wiring

The layered call chain **Screen → Hook → Service → Repository → Supabase** is now
a hard rule (see `../knowledge/architecture.md` → "The layered call chain" and the
updated **momlee-react-native** skill). Screens never import Supabase; business
logic lives only in services; only the repository layer touches the client.

- [x] **DONE (2026-06-11, commit `ce1147a` on momlee-native): machine
      enforcement is LIVE in the app repo.** eslint gates (Architecture, RTL
      physical classes/keys, tokens-only hex, Permission wrapper, Analytics
      wrapper, Dates) in `apps/mobile/eslint.config.js` — CI runs lint, so a
      violation fails the build. Plus `scripts/momlee/check-deps.mjs`
      (Dependency Governance vs `stack-allowlist.json` — adding a dep requires
      editing the allowlist = explicit approval) and `check-migrations.mjs`
      (post-2026-06-12 migrations: RLS with CREATE TABLE + `-- retention:`
      declaration), wired as a CI step. Verified: current code passes clean;
      a violating test file trips all five gate families.
- [x] No existing screen calls Supabase directly (verified by the lint run —
      0 errors).

## Dev-environment notes (already in effect)

- The app is pinned to **Expo SDK 54** so the App Store **Expo Go** runs it on a
  real iPhone via QR (SDK 56 was rejected by Expo Go). Don't bump the SDK
  without checking Expo Go support. See `../knowledge/dev-environment.md`.
- pnpm monorepo with `node-linker=hoisted` (required for Metro). After pulling
  dep changes: `pnpm install` from the repo root.

## /momlee-audit round 2 — fixes landed (2026-06-11, commit a5a024a)

DONE (Maor approved "fix everything"): H1 session→expo-secure-store ·
H2 not-configured bypass now __DEV__-only · H3 analytics wrapper in
@momlee/core (typed taxonomy + providers; screens use analytics.track) ·
G1 lint gates now cover packages/ui+core (caught & fixed physical offsets in
OtpInput) · R1 CTA Loading state wired · L2 fontByWeight token · import dups.

STILL OPEN:
- [ ] PostHog: Maor's project key + dependency gate → providers/posthog.provider.ts
- [ ] Taxonomy decision (Maor): adopt `onboarding_step_viewed` or migrate to seed events
- [ ] Component FILE naming: conventions say kebab-case; component files are
      PascalCase (RN standard) — Maor to rule
- [ ] R3 offline handling (with the shared error/empty states work)
- [ ] M4 regenerate database.types.ts after the live migration applies
- [ ] Live security deploy (see the security section above)

## Mechanical-enforcement additions (2026-06-12)

Per Maor ("mechanical checks over memory text"): the asset-color lesson is
now machine-enforced — `check-token-provenance.mjs` in CI (every token value
must cite its Figma variable on the same line; caught touchTarget on run #1)
+ an eslint rule banning hex literals in color-like JSX props. Also live:
the TextInput writingDirection rule in check-rtl.mjs (caught OtpInput run #1).

## Still open after the live deploy (2026-06-12)
- [ ] Delete the 6 legacy Edge Functions from the Supabase dashboard (still deployed)
- [ ] Rotate Mapbox tokens (old exposure) + ROTATE THE TWILIO AUTH TOKEN (pasted in chat)
- [ ] Maor: delete the temporary Supabase access token (used for db push, no longer needed)
- [ ] Configure Test Phone Numbers (972528547424=123456 format) to test without SMS costs

