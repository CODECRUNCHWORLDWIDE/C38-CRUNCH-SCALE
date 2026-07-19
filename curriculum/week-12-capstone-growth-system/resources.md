# Week 12 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific capstone task calls for it. This week deliberately reuses **plain PostgreSQL/SQLite SQL and pandas** for everything — nothing here requires a paid tool, and nothing in this week's deliverables depends on one.

## Required reading (this week's core)

- **PostgreSQL — `CREATE VIEW`:** <https://www.postgresql.org/docs/current/sql-createview.html>
  *Why: every `stg_*` model this week is a view — review exactly what it does and doesn't materialize before Exercise 1.*
- **PostgreSQL — `CREATE TABLE AS`:** <https://www.postgresql.org/docs/current/sql-createtableas.html>
  *Why: `dim_user`, `fct_acquisition`, and every other mart this week is built this way.*
- **PostgreSQL — Date/Time Functions and Operators:** <https://www.postgresql.org/docs/current/functions-datetime.html>
  *Why: the month-end MRR join (Lecture 2, Section 2) and the retention age-window CTE (Exercise 2, Task 4) both depend on getting this exactly right.*
- **NumPy — `polyfit`:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
  *Why: Lecture 3's linear-regression forecast method uses this directly.*
- **Investopedia — "Statistical Significance":** <https://www.investopedia.com/terms/s/statistically_significant.asp>
  *Why: a clean, free refresher before Lecture 2, Section 5's experiment read — especially the p-value misinterpretation Quiz Q8 tests directly.*

## SQLite equivalents (keep these open alongside the Postgres docs)

- **SQLite — Date and Time Functions:** <https://www.sqlite.org/lang_datefunc.html>
- **SQLite — `CREATE VIEW`:** <https://www.sqlite.org/lang_createview.html>
- **SQLite — Recursive Common Table Expressions** (an alternative to Postgres `generate_series` for building age-window tables): <https://www.sqlite.org/lang_with.html#recursive_common_table_expressions>

## Concept references (tooling-agnostic — read for the idea, not the product)

- **dbt — "About staging models":** <https://docs.getdbt.com/best-practices/how-we-structure/2-staging>
  *Why: the raw → staging → marts pattern this capstone reuses from Week 6, explained cleanly by the tool most associated with popularizing it — this course implements the same idea in plain SQL, no dbt required.*
- **Kimball Group — Dimensional modeling techniques:** <https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/>
  *Why: the original source for the dimension/fact/grain vocabulary this entire course leans on — worth one last skim before you design `vw_retention_cohorts` in Challenge 1.*
- **Evan Miller — "How Not To Run an A/B Test":** <https://www.evanmiller.org/how-not-to-run-an-ab-test.html>
  *Why: the classic, free write-up of exactly the underpowered-experiment trap Exercise 3 and Homework Problem 4 walk you through hands-on.*
- **statsmodels — `NormalIndPower`:** <https://www.statsmodels.org/stable/generated/statsmodels.stats.power.NormalIndPower.html>
  *Why: the tool Exercise 3, Task 4 asks you to use (or a documented equivalent) for the minimum-detectable-sample-size estimate.*

## Growth and RevOps background

- **Bessemer Venture Partners — "State of the Cloud":** <https://www.bvp.com/atlas/state-of-the-cloud>
  *Why: a widely cited, freely available industry reference for LTV:CAC and payback benchmarks — useful for sanity-checking your own numbers against real-world ranges in the mini-project's decision memo.*
- **OpenView Partners — SaaS metrics glossary:** <https://openviewpartners.com/glossary/>
  *Why: free glossary covering logo churn, net revenue retention, and other terms this week's homework and metric-definitions work use.*
- **Andrew Chen — "Growth" archive:** <https://andrewchen.com/category/growth/>
  *Why: free, widely referenced essays on channel economics and growth-team judgment calls — good background reading for Challenge 2's decision memo.*

## Practice beyond the seed database

- **pandas — Time series / date functionality:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
  *Why: more reps if the forecast comparison in Lecture 3 / Challenge 2 felt unfamiliar.*
- **PostgreSQL Exercises:** <https://pgexercises.com/>
  *Why: more join/aggregation/window-function reps if any part of building the marts felt slow.*

## Glossary

| Term | Definition |
|------|------------|
| **Analysis cutoff** | The explicit "as of" date a warehouse's marts are built against — a business decision encoded in a `WHERE` clause, not a fact the raw data hands you for free. |
| **Scope statement** | The one sentence defining exactly what question a capstone (or any analysis) is answering — everything that doesn't serve it is scope creep. |
| **Fully-loaded CAC** | Total channel cost (media, team, or program spend) divided by paying customers that channel produced — as opposed to media-only CAC, which undercounts. |
| **Cohort-sum LTV** | Contribution margin × observed retention, summed across the ages you actually have evidence for — a conservative floor, not a projection. |
| **Reciprocal-formula LTV** | `margin ÷ average monthly churn rate` — a projection that assumes constant hazard forever; breaks (undefined) when observed churn is exactly zero. |
| **Right-censoring** | The fact that a customer who hasn't churned *yet* isn't the same as a customer who will never churn — they've simply only been observed for a limited window. |
| **Thin sample** | A retention, churn, or experiment data point built on too few observations to trust at face value — flagged, not silently reported as fact. |
| **Two-proportion z-test** | The significance test used to compare two conversion/activation rates (e.g., experiment control vs. treatment) and produce a p-value. |
| **Statistical power** | The probability a test correctly detects a real effect of a given size at a given sample size — used to estimate the minimum sample an experiment needs. |
| **Decision narrative** | A recommendation structured as ask → evidence → uncertainty → checkpoint, with every evidentiary claim traceable to a specific query or table. |
| **Forecast reconciliation** | Presenting multiple forecasting methods side by side, quantifying their disagreement, and stating which one you'd act on and why — instead of hiding the disagreement behind a single number. |
| **Idempotent build** | A warehouse build script that produces byte-for-byte identical results whether run once or run many times in a row. |

---

*Broken link? Open an issue or PR.*
