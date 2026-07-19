# Week 8 — Exercises

Three guided exercises, ~60–90 min each. **Write every formula and every query yourself** — don't copy-paste from the lectures. The sizing formula, the z-test, and the chi-square check all look intimidating the first time and completely mechanical the fifth time; the only way to get to the fifth time is typing them out four times first.

1. **[Exercise 1 — Size an A/B test](exercise-01-size-a-test.md)** — turn a baseline rate and an MDE into a required sample size and test duration.
2. **[Exercise 2 — Analyze an A/B result](exercise-02-analyze-a-result.md)** — full significance + confidence interval + guardrail read of the `checkout_sessions` pilot.
3. **[Exercise 3 — Detect sample-ratio mismatch](exercise-03-detect-srm.md)** — run a chi-square goodness-of-fit test to catch a broken randomizer, and confirm a healthy one.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM checkout_sessions;` returns **100**.
- You have a SQL shell open (`psql crunch_scale_w8` or `sqlite3 crunch_scale_w8.db`) **and** a Python session/notebook with `pandas` and `numpy` importable (`pip install pandas numpy` if you haven't yet).

## Suggested workflow

- Do the sizing math (Exercise 1) with pen-and-paper-style Python first — no SQL needed, it's pure arithmetic on four numbers you're given.
- For Exercises 2 and 3, get the raw counts out of SQL first (a `GROUP BY` query), then do the statistical test itself in Python. This mirrors exactly how the lectures did it, and it keeps the two skills — querying and computing — cleanly separated so a mistake in one doesn't hide inside the other.
- Check every number against the "Expected" note before moving on. If yours doesn't match, the bug is almost always a pooled-vs-unpooled standard error mix-up (Lecture 2, section 5) or a `NULL`-handling mistake in the SQL (remember: `order_value_usd`/`refunded` are `NULL` for non-converted sessions, not 0/false).
- Save your work as you go: one `.sql` file and one `.py` (or notebook) per exercise. You'll commit them.

## Note on precision

Every "Expected" value in these exercises is rounded to 3 decimal places for rates and z-scores, 4 for p-values, and whole numbers for sample sizes (always rounded **up**). A difference in the last digit from rounding order is fine; a bigger difference means a logic bug, not a rounding quirk.
