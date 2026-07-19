# Exercise 2 — Classify Vanity vs. Actionable Metrics

**Goal:** Build the reflex of catching a vanity metric on sight, and — harder — explaining precisely *why* it's vanity rather than just asserting it.

**Estimated time:** 45 minutes. No database needed — this is written reasoning.

## What makes a metric "vanity"

A metric is **vanity** if it can go up while the business gets no healthier — usually because it's a raw cumulative count, easily inflated by spending, or disconnected from repeat behavior. A metric is **actionable** if a team can change it on purpose and the change reliably signals something real about customer value. (Re-read Lecture 1, section 2, questions 1 and 4, before starting — this exercise is those two questions applied fifteen times.)

Some metrics are genuinely **borderline** — actionable in one context, vanity in another, depending on what surrounds it. Recognizing borderline cases well is worth more than getting every clear-cut one right.

Create a file `exercise-02.md`.

## The 15 metrics

For each, write: **Vanity**, **Actionable**, or **Borderline** — plus **one sentence** explaining your call by name-checking one of the five questions from Lecture 1.

1. Total registered users, all time (cumulative, never decreases)
2. Weekly Engaged Users (StreakLab's north star — 3+ check-ins in trailing 7 days)
3. Number of push notifications sent this week
4. Number of push notifications **opened** this week, as a % of sent
5. App Store star rating (average, all-time)
6. Monthly Recurring Revenue (MRR)
7. Net New MRR this month (new + expansion − churned − contraction)
8. Total pageviews across the marketing site
9. Signup-to-activation conversion rate (Lecture 2's activation rate)
10. Number of press mentions this quarter
11. Number of features shipped this quarter
12. Week-4 retention rate for a signup cohort
13. Total dollars spent on paid ads this month
14. Customer Acquisition Cost (CAC) — ad spend ÷ new customers acquired
15. Number of `invite_sent` events (StreakLab referrals), counted alone, with no `invite_accepted` data

## Constraints

- You must use the word **"vanity," "actionable," or "borderline"** exactly once per item — no hedging with "it depends" as the entire answer. If it's borderline, say *what it depends on*.
- At least 3 of your 15 answers must be **Borderline**, and each must state the specific condition that would flip it one way or the other. (Metric 3 is a strong candidate — think about what it depends on.)
- Two of your answers should reference **question 4** (hard to game) specifically — pick the two metrics on this list most trivially inflated by spending money alone, and say so.

## A worked example (metric 15, partly done for you)

**Metric 15 — `invite_sent` count alone:** *Borderline, leaning vanity.* It fails question 4 hard — a growth team under pressure can trivially inflate "invites sent" by auto-prompting every user aggressively, producing a big number that reflects nagging, not enthusiasm. It only becomes actionable when paired with `invite_accepted` as a **rate** (Lecture 2, section 6) — the moment you're measuring conversion, not raw volume, question 4 mostly resolves. Finish this one: is `invites_sent ÷ signups` (average invites per user) more actionable than the raw count? Why or why not?

## Done when…

- [ ] All 15 metrics classified with a one-sentence justification each.
- [ ] At least 3 marked **Borderline** with the flip condition stated.
- [ ] Two answers explicitly invoke question 4 (game-resistance).
- [ ] Metric 15's worked example is completed in your own words.

## Stretch

Pick the metric on this list you personally would have gotten wrong before this course, and write two sentences on what specifically changed your mind.

## Submission

Commit `exercise-02.md` to your portfolio under `c38-week-01/exercise-02/`.
