# Week 2 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Required reading (this week's core)

- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG`, `LEAD`, `ROW_NUMBER`, `FIRST_VALUE` are the backbone of every query this week — funnel ordering, attribution ranking, and position-based weighting all lean on them.*
- **PostgreSQL — Window Functions reference:** <https://www.postgresql.org/docs/current/functions-window.html>
  *Why: exact signatures and frame-clause behavior for every window function used in the lectures.*
- **PostgreSQL — Aggregate expressions (`FILTER`):** <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
  *Why: the CAC report in Lecture 3 leans on `FILTER (WHERE ...)` to compute several conditional counts from one `GROUP BY`.*
- **SQLite — Window Functions:** <https://www.sqlite.org/windowfunctions.html>
  *Why: the same patterns, no `FILTER` — you'll rewrite it as `COUNT(DISTINCT CASE WHEN ... THEN user_id END)` instead.*
- **Modern SQL — Window Functions:** <https://modern-sql.com/use-case/subtotals-and-grand-totals>
  *Why: a clean, engine-neutral explanation of `PARTITION BY` vs. `GROUP BY`, which trips up nearly everyone the first time they mix the two in one query.*

## Growth & attribution concepts (vendor-neutral parts)

- **Google Analytics — Attribution models overview:** <https://support.google.com/analytics/answer/10596866>
  *Why: the standard vocabulary (first-touch, last-touch, linear, position-based, data-driven) this lecture builds from scratch in SQL — useful to see how a major platform frames the same ideas.*
- **CXL — "Marketing Attribution: A Marketer's Guide":** <https://cxl.com/blog/attribution-modelling/>
  *Why: a longer, practitioner-written treatment of the tradeoffs between models, including the "reshuffling organic demand" problem Lecture 3 covers with brand search.*
- **Reforge / growth-community writing on marginal CAC and channel saturation** (search "marginal CAC saturation curve growth" for current public write-ups — the concept is standard, the best explainer changes over time)
  *Why: the idea that CAC should be evaluated at the margin, not as a trailing average, is the single most useful lens in Lecture 3.*

## Reference (keep in tabs)

- **PostgreSQL — Date/Time Functions and Operators (interval arithmetic):** <https://www.postgresql.org/docs/current/functions-datetime.html>
  *Why: the `event_ts + INTERVAL '14 days'` pattern used throughout Lecture 1's conversion-window queries.*
- **SQLite — Date and Time Functions (`datetime`, `julianday`):** <https://www.sqlite.org/lang_datefunc.html>
  *Why: the SQLite equivalents for every interval computation in this week's Postgres examples.*
- **PostgreSQL — Conditional Expressions (`CASE`, `COALESCE`, `NULLIF`):** <https://www.postgresql.org/docs/current/functions-conditional.html>
  *Why: `CASE` builds the step-order mapping in every funnel query; `NULLIF` is what keeps CAC from crashing on a zero-conversion channel.*
- **PostgreSQL — Common Table Expressions (`WITH`):** <https://www.postgresql.org/docs/current/queries-with.html>
  *Why: every non-trivial query this week is 2–4 chained CTEs (anchor → rank/weight → aggregate) — the shape is worth internalizing.*

## Practice beyond the seed data

- **Mode Analytics — SQL tutorial, "Window Functions":** <https://mode.com/sql-tutorial/sql-window-functions/>
  *Why: more worked examples of `PARTITION BY` outside a marketing context, useful for building the pattern-recognition this week assumes.*
- **PostgreSQL Exercises:** <https://pgexercises.com/>
  *Why: general SQL reps if any specific clause (joins, aggregation) still feels shaky before tackling this week's compound queries.*

## Glossary

| Term | Definition |
|------|------------|
| **Funnel** | An ordered sequence of steps a user progresses through; measured as counts and conversion rates per step. |
| **Step-over-top rate** | A step's user count as a percentage of the very first step's count. |
| **Step-over-previous rate** | A step's user count as a percentage of the immediately preceding step's count. |
| **Conversion window** | A maximum time-lag after which a later event is no longer credited to an earlier one. |
| **Touch** | A single instance of a user encountering a marketing channel (a click, an impression, a visit) before converting. |
| **Attribution model** | A rule for splitting a conversion's credit among the touches that preceded it. |
| **First-touch attribution** | 100% credit to the earliest touch in a user's path. |
| **Last-touch attribution** | 100% credit to the latest touch in a user's path. |
| **Linear attribution** | Credit split evenly across every touch in a path. |
| **Position-based (U-shaped) attribution** | Credit weighted toward the first and last touch (commonly 40/40), with the remainder split among middle touches. |
| **CAC** | Customer Acquisition Cost — spend divided by conversions attributed to that spend. |
| **Marginal CAC** | The cost of the *next* customer at the current spend level, as opposed to the average cost across all spend to date. |
| **Saturation** | The point past which additional spend on a channel stops producing proportional additional results. |
| **Reshuffling organic demand** | When a channel (often branded search) captures credit for a conversion that another channel had already substantially caused, by being the last touch before a user acts on intent created elsewhere. |
| **Reconciliation** | Confirming that an attribution report's total credited revenue equals total revenue — the correctness check for any single-touch or fractional model. |

---

*Broken link? Open an issue or PR.*
