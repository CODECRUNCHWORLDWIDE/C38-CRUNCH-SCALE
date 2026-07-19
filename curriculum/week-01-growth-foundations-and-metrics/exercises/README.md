# Week 1 — Exercises

Three exercises, ~45–90 min each. The first two are pen-and-paper judgment reps; the third is hands-on SQL against the seed data. Do them in order — each builds on the last.

1. **[Exercise 1 — Pick a north star](exercise-01-pick-a-north-star.md)** — apply the five-question test to three different products, not just StreakLab.
2. **[Exercise 2 — Classify vanity vs. actionable](exercise-02-classify-vanity-vs-actionable.md)** — sort a list of 15 real-sounding metrics and defend every call.
3. **[Exercise 3 — Query a metric from raw events](exercise-03-query-a-metric-from-events.md)** — write the SQL for six StreakLab metrics yourself, from a blank query editor.

## Before you start

- You've read all three lectures.
- You loaded the seed data from the [week README](../README.md) and `SELECT COUNT(*) FROM users; SELECT COUNT(*) FROM events;` return **15** and **149**.
- You have a `psql crunch_scale` (or `sqlite3 crunch_scale.db`) session open.

## Suggested workflow

- Exercises 1 and 2 are written reasoning — no database needed. Do them away from the keyboard if that helps you think.
- For Exercise 3, **write your query before running it**, then run it and check against the "Expected" note. If a number surprises you, trace one row by hand before moving on (see Lecture 3, section 5) — that's the whole skill.
- Save your work: `exercise-01.md`, `exercise-02.md`, and `exercise-03.sql` (queries with `-- Task N` comments and their results noted). You'll commit them to your portfolio.

## Data tooling reminder

Every number in every exercise this week is produced by a SQL query against `users`/`events` in PostgreSQL (or SQLite). If you catch yourself about to open a spreadsheet to "just sum this up quickly" — don't. Write the query instead. That instinct is exactly what Lecture 3 exists to retrain.
