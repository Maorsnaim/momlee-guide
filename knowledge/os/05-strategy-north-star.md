# Momlee OS — North Star & Activation (Strategy)

> The single metric that captures realized product value. Anchored 2026-06-28
> (confirmed by Maor). Belongs in the live Notion `01 - Strategy` section too
> (add when aligning). Status: LOCKED.

## North Star — WSMA (Weekly Successful Meetup Attendances)

**Definition:** the count of distinct (mom, meetup) pairs in a rolling 7-day
window where the mom **attended** the meetup (reached the confirmed-attendance
terminal state), for meetups whose date falls in the window.

**Why this one:** the product promise is taking an isolated new mother to a real,
in-person gathering of peers. Signups / profiles / even "joins" are leading
indicators; value is only *realized* when she actually attends. WSMA sits at the
confluence of activation, retention, marketplace, and revenue — and it goes to
zero the instant the app feels empty, so it forces us to face the real problem.

## What counts as "successful" — D5 (locked)

A **successful meetup = ≥2 attendees marked `attended`** (host + ≥1 confirmed).
Two sub-streams, one metric: free meetups (`meetup_type = free`) and pro meetups
(`pro`). Sum into WSMA, but always be able to split them.

- **Per-mom variant** (for cohorts/retention): a mom is a "successful attendee
  this week" if she has ≥1 attendance in the window.

## Interim proxy (until the attendance lifecycle ships)

`attended` needs the meetup attendance lifecycle, which is not built yet. Until
then, run on **Weekly Meetup Joins** (DB-derivable from join rows), **explicitly
labeled a proxy** — joining ≠ attending; over-counting joins masks churn.

## Activation

**Mom activation = % of new moms who attend a first meetup within 14 days of
signup.** Report the funnel M1 completed profile → M2 joined first meetup →
M3 attended first meetup, so "can't get them to join" is distinguishable from
"they join but the meetup never happens."

## Dependencies

- The **meetup attendance lifecycle** (`attended` state) — Story/meetups work;
  until then, the joins proxy.
- The **Events** registry (analytics) — `meetup_joined`, `meetup_attended`.
- MVP note: pro-meetups are admin-curated (Sivan) for now, but WSMA counts both
  free and pro attendances the same way.
