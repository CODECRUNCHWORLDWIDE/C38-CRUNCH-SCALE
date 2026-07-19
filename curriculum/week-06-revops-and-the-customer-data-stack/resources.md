# Week 6 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up. This week deliberately uses **plain PostgreSQL/SQLite SQL** for everything — none of these links require installing dbt or any other paid/hosted tool, and none of this week's deliverables depend on one.

## Required reading (this week's core)

- **PostgreSQL — `CREATE VIEW`:** <https://www.postgresql.org/docs/current/sql-createview.html>
  *Why: staging models in this week are views — understand exactly what a view does and doesn't materialize.*
- **PostgreSQL — `generate_series`:** <https://www.postgresql.org/docs/current/functions-srf.html>
  *Why: the `dim_date` spine (Lecture 2) depends on it; know its signature cold.*
- **PostgreSQL — window functions tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `ROW_NUMBER`/`LAG` show up in staging dedup logic and in the homework's logo-churn calculation.*
- **PostgreSQL — `INSERT ... ON CONFLICT`:** <https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT>
  *Why: the upsert half of Lecture 3's idempotent-load pattern.*
- **PostgreSQL — Transactions:** <https://www.postgresql.org/docs/current/tutorial-transactions.html>
  *Why: `BEGIN`/`COMMIT` is what makes truncate-and-reload safe against a mid-load crash.*

## SQLite equivalents (keep these open alongside the Postgres docs)

- **SQLite — date and time functions** (`strftime`, `date`, recursive-CTE spine): <https://www.sqlite.org/lang_datefunc.html>
- **SQLite — `INSERT OR REPLACE` / conflict resolution:** <https://www.sqlite.org/lang_conflict.html>
- **SQLite — `UNIQUE` constraints:** <https://www.sqlite.org/lang_createtable.html#uniqueconst>
- **SQLite — `CREATE VIEW`:** <https://www.sqlite.org/lang_createview.html>
- **SQLite — recursive common table expressions** (the `WITH RECURSIVE` date-spine pattern): <https://www.sqlite.org/lang_with.html>

## Concept references (tooling-agnostic — read for the idea, not the product)

- **dbt — "About staging models":** <https://docs.getdbt.com/best-practices/how-we-structure/2-staging>
  *Why: dbt didn't invent the raw/staging/marts pattern, but its docs are the clearest public write-up of the convention. This course implements the same idea in plain SQL views, no dbt required.*
- **dbt — "What are data tests?":** <https://docs.getdbt.com/docs/build/data-tests>
  *Why: the not-null/unique/accepted-values/relationships vocabulary Lecture 3 uses comes from here — again, concept only, implemented as plain `SELECT`s this week.*
- **Kimball Group — dimensional modeling techniques:** <https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/>
  *Why: the original, decades-old source for the dimension/fact vocabulary this whole course leans on.*
- **Stripe — Subscriptions API reference:** <https://stripe.com/docs/api/subscriptions>
  *Why: `raw_stripe_subscriptions` is deliberately shaped like a real Stripe export — skim this to see how close (and where it simplifies).*

## RevOps and metrics background

- **Bessemer Venture Partners — "State of the Cloud" MRR/ARR definitions (public, no signup):** <https://www.bvp.com/atlas/state-of-the-cloud>
  *Why: a widely cited, freely available reference for how the industry actually defines and benchmarks SaaS revenue metrics — useful cross-check for your `metrics.md`.*
- **OpenView Partners — SaaS metrics glossary:** <https://openviewpartners.com/glossary/>
  *Why: free glossary covering logo churn, net revenue retention, and other terms this week and later weeks use — good for Problem 5 of the homework.*

## Practice beyond the seed database

- **PostgreSQL Exercises:** <https://pgexercises.com/>
  *Why: more window-function and aggregation reps if `LAG`/`ROW_NUMBER` still feel unfamiliar after the exercises.*
- **Use The Index, Luke:** <https://use-the-index-luke.com/>
  *Why: once your marts are larger than this week's seed data, this free book explains why some of your `CROSS JOIN`-based fact builds will need indexes — worth a first skim now, a real read in Weeks 6–7 of C33 Crunch SQL if you haven't already.*

## Glossary

| Term | Definition |
|------|------------|
| **Raw layer** | Source data landed exactly as the source system provided it; never modified. |
| **Staging layer** | Cleaned, typed, deduped, conformed models — one per raw source, no cross-source business logic (with the documented exception of `stg_subscriptions`). |
| **Marts layer** | Business-ready dimension and fact tables, joinable and documented, queried by everyone downstream. |
| **ELT** | Extract, Load, Transform — raw data lands first; transformation happens afterward, inside the warehouse, in SQL. |
| **Semantic layer** | The single place — a mart's SQL plus a written glossary — where a metric's definition lives exactly once. |
| **Dimension** | A mart table describing who/what/when — slow-changing, filtered and grouped by (`dim_user`, `dim_date`, `dim_plan`). |
| **Fact** | A mart table recording something measurable — a numeric event or snapshot, summed/counted/averaged (`fct_mrr_monthly`, `fct_events`). |
| **Grain** | The precise definition of "one row of this table represents ___" — the single most important design decision for any fact table. |
| **Conformed dimension** | A dimension built exactly once and reused, unmodified, across every fact table in the warehouse. |
| **MRR** | Monthly Recurring Revenue — the normalized monthly value of active subscriptions, annual plans divided by 12. |
| **ARR** | Annual Recurring Revenue — MRR × 12 (with more than one defensible computation method; see Homework Problem 1). |
| **Logo churn** | The rate at which distinct customers (not dollars) stop being active in a period. |
| **Idempotent load** | A load where running it once and running it many times produce the identical result. |
| **Upsert** | Insert new rows, update existing rows matching a key — `ON CONFLICT ... DO UPDATE` (Postgres) / `INSERT OR REPLACE` (SQLite). |
| **Data test** | A `SELECT` designed to return zero rows on healthy data and one-or-more rows the moment something's wrong. |
| **Source of truth** | The one system explicitly designated to win when two systems disagree on a given field — a RevOps decision, not a technical default. |

---

*Broken link? Open an issue or PR.*
