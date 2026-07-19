# Week 4 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Required reading (this week's core)

- **PostgreSQL — Date/Time Functions and Operators:** <https://www.postgresql.org/docs/current/functions-datetime.html>
  *Why: every query this week leans on date arithmetic (`month_number`, day-offsets, calendar spines) — this is the canonical reference for both engines' date math.*
- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG`, `MAX() OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING)` are the backbone of the growth-accounting classification in Lecture 2.*
- **PostgreSQL — Set Returning Functions (`generate_series`):** <https://www.postgresql.org/docs/current/functions-srf.html>
  *Why: `generate_series` is how you build the calendar spine on Postgres — one line instead of a recursive CTE.*
- **SQLite — Recursive Common Table Expressions:** <https://www.sqlite.org/lang_with.html>
  *Why: SQLite's answer to `generate_series` for calendar spines — `WITH RECURSIVE`, used throughout Lecture 2 and Exercise 2.*
- **SQLite — Date and Time Functions:** <https://www.sqlite.org/lang_datefunc.html>
  *Why: `strftime`, `julianday`, and `date(..., '-27 day')` — the exact functions used in this week's SQLite queries.*

## Reference (keep in tabs)

- **PostgreSQL — Aggregate Functions:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: `COUNT(DISTINCT ...)` inside a `GROUP BY` is the workhorse of every triangle, survival, and stickiness query this week.*
- **PostgreSQL — `date_trunc`:** <https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-TRUNC>
  *Why: the Postgres-native way to bucket a timestamp to its containing month or week, an alternative to the `DATE_PART` arithmetic used in the lectures.*
- **SQLite — the `generate_series` table-valued function (ships in the CLI shell):** <https://www.sqlite.org/series.html>
  *Why: if your `sqlite3` build supports it (check with `SELECT * FROM generate_series(1,5);`), it's a lighter-weight alternative to the recursive-CTE calendar spine.*

## Background on the concepts (not SQL-specific)

- **Wikipedia — Survival analysis:** <https://en.wikipedia.org/wiki/Survival_analysis>
  *Why: the origin of the censoring discipline Lecture 3 borrows for the retention "survival curve." Read the "Censoring" section specifically.*
- **Wikipedia — Kaplan–Meier estimator:** <https://en.wikipedia.org/wiki/Kaplan%E2%80%93Meier_estimator>
  *Why: the rigorous version of what Lecture 3's eligible-adjusted curve approximates. Useful for understanding exactly where the growth-analytics convention diverges from the original method (absorbing vs. reversible events).*
- **Wikipedia — Cohort study:** <https://en.wikipedia.org/wiki/Cohort_study>
  *Why: cohort analysis in product analytics is a direct borrow from epidemiology's cohort-study methodology — same idea (track a defined group forward in time), different subject matter.*

## Practice beyond the seed dataset

- **Mode Analytics — SQL tutorial, window functions section:** <https://mode.com/sql-tutorial/sql-window-functions/>
  *Why: more `LAG`/`LEAD`/`ROWS BETWEEN` reps beyond this week's growth-accounting query — the same technique shows up constantly in time-series business SQL.*
- **PostgreSQL Exercises:** <https://pgexercises.com/>
  *Why: more aggregation and window-function drills if Exercise 2's classification query felt shaky.*

## Glossary

| Term | Definition |
|---|---|
| **Cohort** | A group of users sharing a start event (usually signup), bucketed to a period. |
| **Retention triangle / cohort grid** | Cohorts down the rows, periods-since-signup across the columns; each cell is % of that cohort active in that period. |
| **`month_number`** | Calendar periods elapsed between a user's signup and a given event — the x-axis of a cohort curve. |
| **Right-censoring** | A cohort hasn't existed long enough to have reached a given period yet — "not observed," not "observed and zero." |
| **N-day retention** | Whether a user was active on exactly day N after signup (e.g., D1, D7, D30). |
| **Unbounded retention** | Whether a user was active *at any point* during a given period after signup (the metric behind the triangle). |
| **Rolling retention** | Whether a user has been active at all within a trailing N-day window, measured as of today — a present-tense, whole-base snapshot. |
| **Growth accounting** | Splitting a period's active users into new, retained, resurrected, and churned. |
| **New** | Active for the first time, in their own signup period. |
| **Retained** | Active this period and the immediately preceding period. |
| **Resurrected** | Active this period, not the preceding period, but active further back — a comeback. |
| **Churned** | Active the preceding period, not this one. Measured against the *previous* period's active total. |
| **Survival curve** | The eligible-adjusted retention-by-period curve, blended across cohorts old enough to have reached each period. |
| **Stickiness (DAU/MAU)** | Average daily active users divided by monthly active users — a frequency-of-use ratio, distinct from whether users stick around at all. |
| **MAU / DAU** | Monthly / daily active users — distinct users with at least one qualifying event in that window. |
| **Leaky bucket** | A product where retention/survival trends toward zero with no stable floor — acquisition spend leaks out as fast as it comes in. |

---

*Broken link? Open an issue or PR.*
