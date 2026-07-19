# Week 12 — Exercises

Three guided exercises, ~90 min each, that build every table this week's mini-project ships on. **Type every query yourself** — don't copy-paste out of the lectures, even when a task mirrors a worked example there.

1. **[Exercise 1 — Stand up the warehouse](exercise-01-stand-up-the-warehouse.md)** — build all five `stg_*` views and both `dim_*` tables on `crunch_boards`.
2. **[Exercise 2 — Build the metric suite](exercise-02-build-the-metric-suite.md)** — build `fct_acquisition`, `fct_mrr_monthly`, and `fct_unit_economics`, and read a full retention curve out of them.
3. **[Exercise 3 — Run the capstone experiment](exercise-03-run-the-capstone-experiment.md)** — analyze `onboarding_checklist_v2` end to end and write the ship/hold/extend recommendation.

## Before you start

- You've read all three lectures.
- You ran the seed from the [week README](../README.md) and every sanity-check count matched (`72`, `56`, `18`, `35`, `24`).
- You have a shell open: `psql crunch_boards` (Postgres) or `sqlite3 crunch_boards.db` (SQLite).
- All SQL in this week's files is written to run **unchanged on PostgreSQL**, with SQLite alternatives called out explicitly wherever the two engines diverge (mainly: date-truncation syntax and month-end arithmetic).

## Suggested workflow

- Open the exercise file beside your terminal.
- Build every `stg_*`/`dim_*`/`fct_*` object as a real database object (`CREATE VIEW` or `CREATE TABLE`) — don't just eyeball a `SELECT`. Later exercises, the challenges, and the mini-project all query the objects you build here.
- For each task, **write your query before running it**, then check the output against the "Expected" note. If a number surprises you, stop and figure out why before moving on — a surprising number is usually the whole lesson, not a mistake to paper over.
- Save your work in a `solutions.sql` file: one `CREATE` or `SELECT` per task, each preceded by a `-- Task N` comment. You'll build on this file across all three exercises, both challenges, and the mini-project — by Saturday it should be the single source of truth for your entire capstone build.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. SQLite users: run `.headers on` and `.mode column` first so output is readable. Month-end date arithmetic uses `DATE(x, '+1 month', '-1 day')` on SQLite versus `(x + INTERVAL '1 month' - INTERVAL '1 day')::date` on Postgres — both spellings are given inline wherever it comes up.
