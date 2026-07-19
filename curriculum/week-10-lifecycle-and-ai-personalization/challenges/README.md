# Week 10 — Challenges

Two open-ended problems, ~90 minutes each. **No single right answer for the design work** — but Challenge 2's measurement has an exact numeric answer, because a lift measurement that can't be pinned down to a number isn't a measurement.

1. **[Challenge 1 — Build a win-back lifecycle journey](challenge-01-build-a-winback-journey.md)** — design the trigger, the journey steps, and a randomized holdout split for the `winback_email_discount` segment.
2. **[Challenge 2 — Measure campaign lift](challenge-02-measure-campaign-lift.md)** — compute the observed lift, run the significance check, and size the test that would actually detect it.

## Before you start

- Exercises 1–3 are complete: `churn_features.csv`, your trained model, and `churn_scored.csv` (or a `scored_customers` table) with `churn_score`, `risk_band`, `value_tier`, and `action` all populated.
- You've read Lecture 3 (Lifecycle Campaigns) in full — both challenges lean directly on it.

## How these differ from the exercises

Challenge 1 has design-decision latitude (journey timing, message sequencing, discount sizing) — document your reasoning the way a real growth/RevOps hire would: propose a shape, defend it in writing, be honest about tradeoffs you didn't choose. Challenge 2 does **not** have that latitude — its numbers (segment size, split, churn rates, p-value) are fully determined by this week's seed and a documented random split, so your output should match the lecture's numbers closely. The judgment in Challenge 2 is in the *interpretation*, not the arithmetic.

## Submission

Each challenge asks for a `challenge-0N.md` (design doc / write-up) plus the SQL/Python that implements it. Commit both to your portfolio under `c38-week-10/challenge-0N/`.
