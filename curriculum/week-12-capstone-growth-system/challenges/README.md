# Week 12 — Challenges

Two open-ended problems, ~2 hours and ~1.5 hours respectively. **No single right answer** — these are judged on the reasoning and the documentation, not just a query that runs.

1. **[Challenge 1 — Build the end-to-end growth system](challenge-01-end-to-end-growth-system.md)** — wire every mart from Exercises 1–3 into one idempotent build script, plus a retention-cohort view none of the exercises built directly.
2. **[Challenge 2 — Defend the growth plan](challenge-02-defend-the-growth-plan.md)** — forecast with two-plus methods, reconcile them, and write the channel-budget recommendation a founder would actually act on.

## Before you start

- Exercises 1–3 are complete: all `stg_*` views, both `dim_*` tables, `fct_acquisition`, `fct_mrr_monthly`, `fct_unit_economics` (with `ltv_cohort_sum` and `ltv_to_cac`), and `fct_experiment_results`.
- You've read all three lectures in full — Challenge 1 leans on Lecture 1 (idempotency, warehouse shape) and Lecture 2 (the marts); Challenge 2 leans entirely on Lecture 3.

## How these differ from the exercises

The exercises had exact expected row counts and dollar figures. These challenges don't — they ask you to make and justify **design and judgment decisions** the way a real growth analyst or RevOps hire would: propose a build, defend a forecast choice, own a recommendation, and be honest about what you didn't have enough evidence to conclude. A reviewer should be able to read your `challenge-0N.md` and understand not just *what* you built but *why* you built it that way instead of two other reasonable ways.

## Submission

Each challenge asks for a `challenge-0N.md` (design doc / decision memo) plus the SQL and/or Python that implements it. Commit both to your portfolio under `c38-week-12/challenge-0N/`.
