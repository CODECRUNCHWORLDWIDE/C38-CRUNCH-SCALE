# Week 6 — Exercises

Three guided exercises, ~60–90 min each. **Type every query yourself** — don't copy-paste out of the lectures, even when a task builds directly on a worked example there. Reading SQL and writing SQL are different skills, and only writing sticks.

1. **[Exercise 1 — Build a staging layer](exercise-01-build-a-staging-layer.md)** — clean, type, dedupe, and conform all four raw tables into `stg_*` models.
2. **[Exercise 2 — Model an MRR fact](exercise-02-model-an-mrr-fact.md)** — build `dim_date`, `dim_user`, and `fct_mrr_monthly` from scratch and read a six-month revenue trend out of it.
3. **[Exercise 3 — Reconcile two sources](exercise-03-reconcile-two-sources.md)** — find and resolve the three real discrepancies between Stripe and the internal app database.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and the four sanity-check counts (`15`, `58`, `18`, `15`) all matched.
- You have a shell open: `psql crunch_flow` (Postgres) or `sqlite3 crunch_flow.db` (SQLite).
- All SQL in this week's files is written to run **unchanged on PostgreSQL**, with SQLite alternatives called out explicitly wherever the two engines diverge (mainly: `generate_series` vs. a recursive CTE, `TRUNCATE` vs. `DELETE`, `ON CONFLICT` vs. `INSERT OR REPLACE`, `FILTER` vs. `CASE WHEN`).

## Suggested workflow

- Open the exercise file beside your terminal.
- Build each `stg_*`/`dim_*`/`fct_*` object as a real database object (`CREATE VIEW` or `CREATE TABLE`) — don't just eyeball the query result. Later exercises and the mini-project query the objects you build here.
- For each task, **write your query before running it**, then run it and check the output against the "Expected" note.
- If a result surprises you, stop and figure out *why* before moving on — in a RevOps pipeline, the surprising result is usually the whole lesson.
- Save your work in a `solutions.sql` file: one `CREATE` or `SELECT` per task, each preceded by a `-- Task N` comment. You'll build on this file across all three exercises and the mini-project.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. Where the two differ, the task calls it out. SQLite users: run `.headers on` and `.mode column` first so output is readable, and remember SQLite has no native `BOOLEAN` (it stores `0`/`1`) and no `TIMESTAMP` type enforcement (it stores text/real — comparisons still work correctly on the `'YYYY-MM-DD HH:MM:SS'` format we used when seeding).
