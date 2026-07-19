# Week 5 — Exercises

Three guided exercises, ~45–60 min each. **Write every query and every pandas snippet yourself** — don't copy-paste from the lectures. Translating "compute LTV" into working code is the actual skill; reading someone else's working code isn't.

1. **[Exercise 1 — Compute LTV from cohort retention](exercise-01-compute-ltv-from-cohorts.md)** — build the pooled retention curve in SQL and turn it into LTV two ways.
2. **[Exercise 2 — Compute fully-loaded CAC per channel](exercise-02-loaded-cac-per-channel.md)** — media-only vs. fully-loaded CAC, blended vs. trailing.
3. **[Exercise 3 — Compute payback and LTV:CAC](exercise-03-payback-and-ratio.md)** — naive vs. retention-adjusted payback, and the ratio, in pandas.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and both sanity checks return `64` and `24`.
- You have a SQL shell open (`psql crunch_scale_w5` or `sqlite3 crunch_scale_w5.db`) **and** a Python session/notebook with `pandas` importable.

## Suggested workflow

- Do the SQL part of each exercise first, in your shell. Get the numbers right before you touch pandas.
- Then do the pandas part — either by hand-typing the SQL result as a small `DataFrame` (fine for these exercise sizes), or by connecting pandas to your database with `pandas.read_sql` if you're comfortable with that already.
- Check every number against the "Expected" note before moving on. If yours doesn't match, the bug is almost always in how `NULL`/censored customers are being counted — re-read Lecture 1 §2–3 before anything else.
- Save your work as you go: one `.sql` file and one `.py` (or notebook) per exercise. You'll commit them.

## Note on rounding

Every "Expected" value in these exercises is rounded to 2 decimal places for dollars and 3 decimal places for rates/ratios. Small differences in the last digit from rounding order are fine; a difference bigger than a cent or a thousandth means a logic bug, not a rounding quirk.
