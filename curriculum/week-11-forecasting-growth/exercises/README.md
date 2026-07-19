# Week 11 — Exercises

Three guided exercises, ~1 hour each. **Type every query and every line of pandas yourself** — don't copy-paste from the lectures. Forecasting is a skill you build by doing the arithmetic, not by reading someone else do it.

1. **[Exercise 1 — Build an MRR bridge](exercise-01-build-an-mrr-bridge.md)** — validate the bridge reconciles, then compute expansion/contraction/churn rates and net revenue retention per month.
2. **[Exercise 2 — Project retention-adjusted revenue](exercise-02-project-retained-revenue.md)** — build the pooled retention-by-age curve and project unobserved cohorts forward with it.
3. **[Exercise 3 — Build a three-scenario model](exercise-03-three-scenario-model.md)** — build the base/bull/bear bridge forecast for Q1–Q4 2026 from given assumption deltas.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM mrr_bridge_actuals;` returns **12**, `SELECT COUNT(*) FROM cohort_revenue_retention;` returns **57**.
- You have a shell open — `psql crunch_scale_w11` (Postgres) or `sqlite3 crunch_scale_w11.db` (SQLite) — **and** a Python environment with `pandas` and `numpy` installed.

## Suggested workflow

- Open the exercise file beside your terminal/editor.
- Do the SQL parts in your database shell first, then reproduce the same result in pandas — the repetition is what makes the bridge mechanics automatic.
- Save your SQL in a `solutions.sql` and your pandas work in `solutions.py` (or a notebook), one task per file, with the task number in a comment.
- If a number surprises you, stop and trace it back to the seed data before moving on — every number in this week's seed is deliberate.

## Note on the two engines

Postgres's `regr_slope`/`regr_intercept` aggregates (used in Lecture 1) have no SQLite equivalent — if you're on SQLite, do the linear-trend step in pandas instead (`numpy.polyfit`), which every exercise also asks you to reproduce anyway. Everything else in this week's SQL runs unchanged on both engines, including `WITH RECURSIVE`.
