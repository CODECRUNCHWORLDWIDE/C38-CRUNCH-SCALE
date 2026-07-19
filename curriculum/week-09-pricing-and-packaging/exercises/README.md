# Week 9 — Exercises

Three guided exercises, ~45–60 min each. **Write every query and every pandas snippet yourself** — don't copy-paste from the lectures. Re-deriving "build the four PSM curves" or "fit the demand curve" is the actual skill; reading someone else's working code isn't.

1. **[Exercise 1 — Estimate WTP from Survey Data](exercise-01-wtp-from-survey.md)** — build all four Van Westendorp curves from `wtp_survey` and read off PMC/PME/OPP/IDP.
2. **[Exercise 2 — Design a Three-Tier Package](exercise-02-design-three-tiers.md)** — map all 30 `accounts_usage` rows into Starter/Growth/Scale and compute the MRR impact.
3. **[Exercise 3 — Model a Price-Sensitivity Curve](exercise-03-price-sensitivity-curve.md)** — fit a demand curve to `price_experiment`, find the revenue-maximizing price, and compute elasticity.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and all four sanity checks return `30`, `30`, `5`, `3`.
- You have a SQL shell open (`psql crunch_scale_w9` or `sqlite3 crunch_scale_w9.db`) **and** a Python session/notebook with `pandas` and `numpy` importable.

## Suggested workflow

- Do the SQL part of each exercise first, in your shell. Get the counts and cumulative percentages right before you touch pandas.
- Then do the pandas part — either by hand-typing your SQL output into a small `DataFrame` (fine at these sizes), or by connecting pandas to your database with `pandas.read_sql` if you're already comfortable with that.
- Check every number against the "Expected" note before moving on. If yours doesn't match, work backward from the definition of the curve or the fit — nearly every miss in this set traces back to an inequality pointing the wrong way (`>=` vs `<=`) or a `GROUP BY` that pooled a segment it shouldn't have.
- Save your work as you go: one `.sql` file and one `.py` (or notebook) per exercise. You'll commit them.

## Note on rounding

Every "Expected" value in these exercises is rounded to 2 decimal places for dollars and prices, and 1 decimal place for percentages. A difference in the last digit from rounding order is fine; a difference of a full percentage point or more than a cent means a logic bug, not a rounding quirk.
