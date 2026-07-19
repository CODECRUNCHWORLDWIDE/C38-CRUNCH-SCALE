# Week 11 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first (if you haven't already)

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/>. macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback, ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with `pandas` and `numpy`** — `pip install pandas numpy` (or `conda install pandas numpy`). This week's `numpy.polyfit` calls need `numpy` specifically, not just `pandas`.

## Required reading (this week's core)

- **PostgreSQL — "Recursive Queries" (`WITH RECURSIVE`):** <https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE>
  *Why: the exact mechanism Lecture 1 uses to roll a bridge forward month by month. Read the "worked example" section closely — it's the same pattern.*
- **PostgreSQL — Window Functions:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG`, `SUM() OVER`, `AVG() OVER` with a `ROWS BETWEEN` frame are how you compute trailing rates and verify the bridge chains correctly (Exercise 1).*
- **NumPy — `polyfit` reference:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
  *Why: the one-line linear-regression tool this week uses instead of a heavier forecasting library — understand what `slope` and `intercept` mean before you use them.*
- **SQLite — "The WITH Clause" (recursive CTEs):** <https://www.sqlite.org/lang_with.html>
  *Why: confirms the same recursive-CTE syntax works unchanged on SQLite, with worked Fibonacci/date-spine examples.*

## Reference (keep in tabs)

- **PostgreSQL — Aggregate Functions incl. `regr_slope`/`regr_intercept`:** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE>
  *Why: the built-in linear-regression aggregates used in Lecture 1 §3 (Postgres-only — SQLite users do this step in pandas instead).*
- **pandas — `groupby`, `pivot_table`, and window/rolling docs:** <https://pandas.pydata.org/docs/user_guide/groupby.html> and <https://pandas.pydata.org/docs/user_guide/window.html>
  *Why: the quarterly rollups and trailing-average computations this week reach for constantly.*
- **pandas — `to_datetime` and date arithmetic:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
  *Why: computing a cohort's "age" as of a target date (Lecture 2 §4, Exercise 2) needs correct month-difference arithmetic.*

## Concept background (SaaS metrics, in plain terms)

- **ChartMogul — "MRR Movements explained":** <https://chartmogul.com/blog/mrr-movements-explained/>
  *Why: a widely-used, vendor-neutral explanation of the exact bridge components (new/expansion/contraction/churned) this week builds forecasts from.*
- **ChartMogul — "Net Revenue Retention (NRR)":** <https://chartmogul.com/blog/net-revenue-retention/>
  *Why: connects the NRR formula this week uses directly to industry benchmarks — useful context for judging whether Crunch Flow's sub-100% NRR is unusual for its stage.*
- **OpenView — "SaaS Benchmarks" (annual report, free):** <https://openviewpartners.com/saas-benchmarks/>
  *Why: real-company reference ranges for churn, expansion, and growth rates — a sanity check for whether a scenario's assumptions (Lecture 3) are in a plausible range or fantasy territory.*

## Deeper background on forecasting methodology (optional this week)

- **Rob J. Hyndman & George Athanasopoulos, *Forecasting: Principles and Practice*, 3rd ed. — free online textbook, Ch. 3 & 5:** <https://otexts.com/fpp3/>
  *Why: the standard open reference on time-series forecasting, including prediction intervals and why simple methods often beat complex ones on short series — directly relevant to Lecture 3's "why not ARIMA" choice.*
- **a16z — "The Playbook for Highly Effective B2B SaaS Sales Compensation" and related growth-metrics writeups:** search "a16z SaaS metrics" for the current post
  *Why: industry framing for why `new_mrr` is projected differently from the "rate" lines — it's the output of a sales/marketing system, not a property of the existing base.*

## Practice beyond the seed data

- **Kaggle — "Store Sales - Time Series Forecasting" (free competition/dataset):** <https://www.kaggle.com/competitions/store-sales-time-series-forecasting>
  *Why: a real multi-year dataset with genuine seasonality — a good next step after Challenge 1's "only one year of data" constraint, once you have more than one cycle to work with.*

## Glossary

| Term | Definition |
|------|------------|
| **MRR bridge** | The identity `ending = starting + new + expansion − contraction − churned`, applied per period. |
| **New MRR** | Revenue added by brand-new customers in a period. |
| **Expansion MRR** | Revenue added by existing customers upgrading/buying more. |
| **Contraction MRR** | Revenue lost to existing customers downgrading (but not leaving). |
| **Churned MRR** | Revenue lost to customers canceling entirely. |
| **Net revenue retention (NRR)** | `(starting + expansion − contraction − churned) / starting`; NRR > 100% means the existing base alone grows. |
| **Logo churn** | The rate of *customers* (not dollars) canceling in a period. |
| **Cohort** | A group of customers who signed up in the same period, tracked together over time. |
| **Right-censoring** | A cohort not yet old enough to have data at a given age — absence of data, not low retention. |
| **Revenue retention curve** | A cohort's active MRR at each age as a percentage of its starting MRR; can rise above 100% via expansion. |
| **Flat continuation** | Holding a retention curve at its last observed value for ages beyond what's been measured, instead of extrapolating a trend. |
| **OLS / linear trend** | Ordinary least squares — the straight line minimizing squared distance to a set of points; `numpy.polyfit(x, y, 1)`. |
| **Residual** | The difference between an actual value and a model's fitted value for that same point. |
| **RMSE** | Root-mean-square error — the typical size of a model's residuals, used to size a prediction interval. |
| **Prediction interval** | A range around a point forecast reflecting historical model uncertainty, typically widening with the forecast horizon. |
| **Scenario (base/bull/bear)** | A forecast variant built from an explicit, named set of alternative assumptions. |
| **Invalidation condition** | A concrete, checkable event that, if observed, means a forecast should be rebuilt rather than trusted as-is. |

---

*Broken link? Open an issue or PR.*
