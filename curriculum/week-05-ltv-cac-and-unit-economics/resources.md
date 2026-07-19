# Week 5 — Resources

Official references and the few links worth your time. You don't need to read all of these before starting — they're here for when a lecture, exercise, or challenge sends you looking for more depth.

## Install (if you haven't already)

- **PostgreSQL 16+:** <https://www.postgresql.org/download/> — this course's primary engine. This week's retention-curve query uses `generate_series` and `FILTER`, both native to Postgres.
- **SQLite 3.35+:** <https://www.sqlite.org/download.html> — the zero-setup fallback. Every Postgres-specific construct this week has a noted SQLite equivalent (a recursive CTE in place of `generate_series`, `SUM(CASE WHEN...)` in place of `FILTER`).
- **Python 3.10+ with pandas:** <https://pandas.pydata.org/docs/getting_started/install.html> — this course keeps data in SQL and pandas, never a spreadsheet.

## LTV, retention curves, and censoring (Lecture 1)

- **Investopedia — "Customer Lifetime Value (LTV/CLV)":** <https://www.investopedia.com/terms/c/customer-lifetime-value-clv.asp>
- **PostgreSQL — Set Returning Functions (`generate_series`):** <https://www.postgresql.org/docs/current/functions-srf.html>
- **PostgreSQL — Date/Time Functions and Operators:** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **SQLite — Recursive Common Table Expressions:** <https://www.sqlite.org/lang_with.html#recursive_common_table_expressions>
- **pandas — `groupby` user guide:** <https://pandas.pydata.org/docs/user_guide/groupby.html>
- **Wikipedia — "Censoring (statistics)"** (for the right-censoring concept behind `NULL` churn dates): <https://en.wikipedia.org/wiki/Censoring_(statistical)>

## CAC, payback, and the LTV:CAC ratio (Lecture 2)

- **Investopedia — "Customer Acquisition Cost (CAC)":** <https://www.investopedia.com/terms/c/cac.asp>
- **Bessemer Venture Partners — State of the Cloud** (industry benchmark reference for LTV/CAC and payback): <https://www.bvp.com/atlas/state-of-the-cloud>
- **PostgreSQL — `CASE` expressions:** <https://www.postgresql.org/docs/current/functions-conditional.html>
- **PostgreSQL — Aggregate Functions (`FILTER` clause):** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **pandas — `merge`, `join`, and `concat`:** <https://pandas.pydata.org/docs/user_guide/merging.html>

## Contribution margin and modeling in SQL/pandas (Lecture 3)

- **Investopedia — "Contribution Margin":** <https://www.investopedia.com/terms/c/contributionmargin.asp>
- **Y Combinator Startup Library — "A Founder's Guide to Unit Economics":** <https://www.ycombinator.com/library>
- **PostgreSQL — `CREATE VIEW`:** <https://www.postgresql.org/docs/current/sql-createview.html>
- **pandas — `DataFrame` basics and vectorized column math:** <https://pandas.pydata.org/docs/user_guide/10min.html>
- **pandas — writing functions that operate on `DataFrame`s (`apply`):** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.apply.html>

## Broader context (optional, for deeper reading)

- **Christoph Janz (Point Nine Capital) — SaaS metrics essays on LTV/CAC and cohort thinking** (search his blog "The SaaS Napkin"): widely cited independent writing on unit economics for subscription businesses.
- **Amplitude — "Cohort Analysis: A Complete Guide":** <https://amplitude.com/explore/growth/cohort-analysis>
- **NNGroup / general SaaS churn-rate benchmarking articles** are useful for sanity-checking whether a computed churn rate is in a plausible industry range — search "SaaS benchmark monthly churn rate" for current-year figures rather than relying on any single source.

---

*Next: [Week 6 — RevOps & the Customer Data Stack](../week-06-revops-and-the-customer-data-stack/).*
