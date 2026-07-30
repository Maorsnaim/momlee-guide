# Maor ↔ Sivan call log

> Dated summaries of working calls, newest first. Maintained by Claude from
> Maor's debriefs (and Sivan's, when she debriefs her side). Decisions land in
> the Dev Changelog / open-tasks as usual — this file is the conversational
> record that ties them together.

## 2026-07-30

1. **Status correction on the two Notion design tasks** (phone-OTP login +
   verification screens): per Maor they are largely NOT open work — the
   phone/OTP screens already exist in Momlee 2.0 (the 02_-06_ onboarding
   frames, reworked with keyboards and states), and the verification INTRO
   exists too ("שומרות על קהילה בטוחה"). What actually remains on Maor's
   plate: **finish + polish the `Birth List` component** (node 511:4576) and
   **design the Didit FAILURE scenarios** — states/screens for unsuccessful
   verification etc. (which covers the five-state return screen Sivan asked
   for).
2. **NEW joint task agreed — onboarding session persistence** (assigned to
   Sivan, tracked in open-tasks): keep the session alive as long as possible;
   a mom who starts onboarding and stops mid-way must resume from the step
   where she stopped, not from scratch.
3. **Working principle recorded** (now codified in design-system/figma.md):
   the Figma SCREENS are comprehensive but do not cover every scenario — the
   COMPONENTS inside them DO. When a scenario seems missing from a screen,
   the answer is almost always in the component set's variants.

## 2026-07-28

1. Family-status step becomes 4 options (בהריון / בחופשת לידה / אמא מנוסה /
   אחר-מעדיפה לא לומר) — per Sivan's request.
2. Location ("Sweet Spot") step CONFIRMED in onboarding.
3. Verification provider = **Didit** (closes Didit-vs-Persona); the intro
   screen carries a reassurance note (no photo/sensitive data kept; returned
   to Momlee at the end).
4. The DS handoff begins — Maor transfers ready Variables / Text Styles /
   screen components piece by piece.

## 2026-07-21

1. Web login model: **phone + SMS OTP is the ONLY initial signup/auth**;
   email collected post-onboarding (magic-link verify, separate marketing
   consent), eventually required. Facebook dropped; Google delegated to
   Sivan — she later decided no social at all (2026-07-28, the
   Sign-in-with-Apple obligation argument).
