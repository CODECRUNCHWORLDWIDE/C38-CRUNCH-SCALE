# Week 4 — Exercises

Three guided exercises, ~60–90 min each. **Type every query yourself** — don't copy from the lecture notes. You already saw the correct triangle and growth-accounting numbers in the lectures; the point of these exercises is building the muscle to derive them again from a blank cursor.

1. **[Exercise 1 — Build a cohort retention triangle](exercise-01-build-a-cohort-triangle.md)** — the full triangle from raw `users`/`events`, from scratch.
2. **[Exercise 2 — Growth-accounting split](exercise-02-growth-accounting-split.md)** — classify every user-month into new/retained/resurrected/churned and check the identity.
3. **[Exercise 3 — Stickiness ratio](exercise-03-stickiness-ratio.md)** — DAU/MAU by month and by segment.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM users;` / `SELECT COUNT(*) FROM events;` return **48** and **366**.
- You have a shell open: `psql crunch_scale` (Postgres) or `sqlite3 crunch_scale.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal.
- Write the query yourself before checking the "Expected" note — the lecture already showed you the destination; the exercise is the drive there.
- If your number doesn't match, don't just copy the lecture's query — find the specific step where your logic diverges (usually: month-0 counted where it shouldn't be, censoring not handled, or a `JOIN` silently dropping users with zero events).
- Save your answers in a `solutions.sql` file (one query per task, with the task number in a `-- Task N` comment). You'll commit it.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. Where the two differ (date arithmetic, `generate_series` vs. a recursive CTE), the task gives both spellings. If you're on SQLite, run `.headers on` and `.mode column` first so output is readable.
