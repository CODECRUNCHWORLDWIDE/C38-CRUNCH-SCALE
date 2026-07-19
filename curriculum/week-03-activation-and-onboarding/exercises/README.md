# Week 3 — Exercises

Three guided exercises, ~45–60 min each. **Type every query yourself** — don't copy-paste from the lectures. Reading SQL and writing SQL are different skills, and only writing sticks.

1. **[Exercise 1 — Define an activation event](exercise-01-define-an-activation-event.md)** — measure reach for every candidate event and write a justified activation-event definition.
2. **[Exercise 2 — Time-to-value distribution](exercise-02-time-to-value-distribution.md)** — compute the full TTV distribution on both engines.
3. **[Exercise 3 — Onboarding step drop-off](exercise-03-onboarding-step-dropoff.md)** — build the step funnel and find the single biggest drop.

## Before you start

- You've completed all three lectures.
- You ran `generate_seed.py` from the [week README](../README.md) and loaded `users.csv` / `events.csv`.
- `SELECT COUNT(*) FROM users;` returns **400**; `SELECT COUNT(*) FROM events;` returns **5017**.
- You have a shell open: `psql crunch_scale` (Postgres) or `sqlite3 crunch_scale.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal.
- For each task, **write your query before running it**, then run it and check the output against the "Expected" note.
- If a number surprises you, stop and figure out *why* before moving on — that confusion is the lesson.
- Save your work in a `solutions.sql` file (one query per task, with the task number in a `-- comment`) plus any written deliverables the task asks for. You'll commit all of it.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. Where the two differ (`INTERVAL` vs `datetime()`, `EXTRACT` vs `julianday()`, `PERCENTILE_CONT` vs manual `OFFSET`), the task says so and gives both spellings.
