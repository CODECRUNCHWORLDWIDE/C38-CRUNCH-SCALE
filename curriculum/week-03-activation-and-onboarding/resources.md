# Week 3 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first (if you skipped Week 1)

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.9+** — for `generate_seed.py`. The script uses only the standard library (`random`, `csv`, `datetime`) — no `pip install` needed.

## Required reading (this week's core)

- **Amplitude — "What Is User Activation?"**: <https://amplitude.com/blog/activation-metric>
  *Why: the clearest short treatment of activation-event criteria you'll find, and it's what Lecture 1's four-criteria table is built on.*
- **Amplitude — "How to Build a Funnel Analysis"**: <https://amplitude.com/blog/funnel-analysis>
  *Why: the industry-standard framing of step funnels — read alongside Lecture 3.*
- **First Round Review — Rahul Vohra, "How Superhuman Built an Engine to Find Product/Market Fit"**: <https://review.firstround.com/how-superhuman-built-an-engine-to-find-product-market-fit>
  *Why: a real, well-documented case study of a team empirically discovering what drives retention — the same instinct behind Lecture 2's lift table, applied with a survey instead of SQL.*
- **PostgreSQL — Window Functions tutorial**: <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG()` does the heavy lifting in every funnel query this week; understand it once, reuse it forever.*

## Reference (keep in tabs)

- **PostgreSQL — Window Function reference (`LAG`, `PERCENTILE_CONT`)**: <https://www.postgresql.org/docs/current/functions-window.html>
  *Why: exact signatures for the percentile and lag functions used throughout the lectures.*
- **PostgreSQL — Aggregate Expressions (`FILTER`)**: <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
  *Why: `COUNT(*) FILTER (WHERE ...)` is how the lift table computes conditional counts without multiple passes over the data.*
- **PostgreSQL — Subquery Expressions (`EXISTS`)**: <https://www.postgresql.org/docs/current/functions-subquery.html>
  *Why: the `flags` CTE leans on `EXISTS` for every candidate-event boolean.*
- **PostgreSQL — Date/Time Functions and `INTERVAL`**: <https://www.postgresql.org/docs/current/functions-datetime.html>
  *Why: every windowed query (7-day reach, week-4 retention) depends on `INTERVAL` arithmetic.*
- **SQLite — Window Functions**: <https://www.sqlite.org/windowfunctions.html>
  *Why: confirms `LAG()` support and syntax differences on SQLite.*
- **SQLite — Date and Time Functions**: <https://www.sqlite.org/lang_datefunc.html>
  *Why: the `julianday()`/`datetime()` reference you need for every Postgres `INTERVAL`/`EXTRACT` translation this week.*

## Practice beyond the seed data

- **Reforge — public blog index**: <https://www.reforge.com/blog>
  *Why: deeper, practitioner-written pieces on activation and onboarding from teams who've run this analysis at scale.*
- **Andrew Chen — growth writing**: <https://andrewchen.com/>
  *Why: long-running, well-known independent writing on growth mechanics, including activation and the "aha moment" framing this week borrows from.*
- **Lenny's Newsletter**: <https://www.lennysnewsletter.com/>
  *Why: frequent, concrete write-ups on onboarding and activation from operators at a range of companies — good for pressure-testing your Problem 3 homework answer against real-world examples.*

## Background (optional this week)

- **"Correlation does not imply causation"**: <https://en.wikipedia.org/wiki/Correlation_does_not_imply_causation>
  *Why: the exact trap Lecture 2 §4–5 and Challenge 1 are built to make you avoid — worth a careful read, not just a skim, before you write your Challenge 1 memo.*

## Glossary

| Term | Definition |
|------|------------|
| **Activation event** | A specific, early, observable event that marks a user's first experience of a product's core value, chosen because it predicts retention. |
| **Aha moment** | Informal name for the behavior an activation event captures — the moment a user "gets it." |
| **Reach** | The percentage of a cohort (usually all signups) that ever fires a given event. |
| **Activation rate** | Reach of the activation event specifically, measured within a fixed window (e.g. 7 days) so cohorts are comparable. |
| **Lift** | The percentage-point gap in a retention outcome between users who did and didn't take some early action. |
| **Endogeneity / reverse causation** | When a metric appears predictive because it's a downstream symptom of the same underlying cause, not an independent early lever. |
| **Window leakage** | Measuring a candidate event without an early time cap, letting very-late occurrences count as if they were known early. |
| **Time-to-value (TTV)** | The elapsed time between signup and a user's activation event. |
| **Step funnel** | An ordered sequence of events with per-step counts and step-over-step conversion rates. |
| **Step-over-step conversion** | The percentage of users at one funnel step who go on to reach the next step (as opposed to cumulative reach from the top). |
| **`LAG()`** | A window function returning a prior row's value within an `ORDER BY`, used here to compute step-over-step conversion without a self-join. |
| **`PERCENTILE_CONT`** | A PostgreSQL ordered-set aggregate computing an interpolated percentile; unavailable in SQLite. |

---

*Broken link? Open an issue or PR.*
