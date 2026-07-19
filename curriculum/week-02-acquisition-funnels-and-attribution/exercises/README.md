# Week 2 — Exercises

Three guided exercises, ~45–90 min each. **Type every query yourself** — don't copy-paste from the lectures. The dataset is small enough to sanity-check by hand; use that to catch your own mistakes before you check the expected values.

1. **[Exercise 1 — Build a funnel conversion query](exercise-01-build-a-funnel-query.md)** — step counts, ordering, and both kinds of conversion rate.
2. **[Exercise 2 — First- vs. last-touch attribution](exercise-02-first-vs-last-touch.md)** — build both models, compare them channel by channel.
3. **[Exercise 3 — Per-channel CAC report](exercise-03-channel-cac-report.md)** — join spend to attributed conversions under two models.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and the five sanity-check counts all match (20 users, 31 touches, 59 funnel events, 10 conversions, $4,500 total spend).
- You have a shell open: `psql crunch_growth` (Postgres) or `sqlite3 crunch_growth.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal.
- Write your query before running it, then check the output against the "Expected" note. If it doesn't match, that gap is the lesson — find it before moving on.
- Save your answers in a `solutions.sql` file (one query per task, with the task number in a `-- comment`). You'll commit it.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. Where the two differ — interval arithmetic, `FILTER`, integer division — the task calls it out and gives both spellings. On SQLite, replace `x + INTERVAL 'n days'` with `datetime(x, '+n days')`, and replace `FILTER (WHERE ...)` with `COUNT(DISTINCT CASE WHEN ... THEN user_id END)`.
