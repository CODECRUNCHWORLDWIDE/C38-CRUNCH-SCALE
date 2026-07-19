# Exercise 1 — Build RFM Scores in SQL

**Goal:** Write the complete recency/frequency/monetary/`NTILE`/tier pipeline yourself, from a blank query editor, against the live `customers`/`orders` seed data. By the end, "build an RFM segmentation" stops meaning "read Lecture 1" and starts meaning "write the CTE chain from memory."

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded (see the [week README](../README.md)):

```sql
SELECT COUNT(*) FROM customers;   -- must print 30
SELECT COUNT(*) FROM orders;      -- must print 125
```

Remember: **"today" is 2025-12-31** everywhere this week. Create `exercise-01.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Raw R, F, M per customer.** Write the `rfm_base` CTE from Lecture 1: for every customer, `recency_days` (with the `9999` sentinel for never-purchased customers), `frequency` (order count), and `monetary` (total spend, `COALESCE`d to `0`). *(Expected: 30 rows. Customers `22` and `24` should show `recency_days = 9999`, `frequency = 0`, `monetary = 0`.)*

2. **Score each axis with `NTILE(5)`.** Add `r_score`, `f_score`, `m_score` columns using `NTILE(5) OVER (...)`, remembering the recency inversion. *(Expected: every score is an integer 1–5. Spot-check customer `1` — Northwind Traders — should show `r_score = 4`, `f_score = 5`, `m_score = 5`.)*

3. **`rfm_total` and a sort.** Add `rfm_total` (the sum of the three scores) and return all 30 rows sorted highest total first, ties broken by `customer_id` ascending. *(Expected: four customers tie for the top score at `rfm_total = 14` — `1`, `2`, `3`, `4` — so with the `customer_id ASC` tiebreak the top row is `customer_id = 1`, Northwind Traders. The bottom two rows are customers `22` and `24`, both `rfm_total = 3`.)*

4. **Tier classification.** Add the `rfm_segment` `CASE` expression from Lecture 1 section 4 (`Champions` / `Loyal Core` / `At Risk` / `New/Promising` / `Hibernating`). *(Expected 5 segments with sizes: Champions 5, Loyal Core 8, At Risk 6, New/Promising 5, Hibernating 6 — sums to 30.)*

5. **Segment value rollup.** From your Task 4 result, `GROUP BY rfm_segment` and compute `COUNT(*)`, `ROUND(AVG(monetary), 2)`, and `ROUND(SUM(monetary), 2)`, sorted by total value descending. *(Expected top row: `Champions`, n=5, avg ≈ `916.74`, total ≈ `4583.71`.)*

6. **The `NULL`-handling check.** Write a query proving your Task 1 CTE handles the two never-purchased customers correctly: it should return exactly `2` rows where `frequency = 0`, and both should show `recency_days = 9999` (not `NULL`, not `0`, not missing from the result entirely). *(Expected: 2 rows — customer `22` Halcyon Health and customer `24` Bluepeak Media.)*

## Expected result (spot checks)

- Task 1 → 30 rows; customers 22 and 24 both show `recency_days=9999, frequency=0, monetary=0`.
- Task 2 → customer 1: `r_score=4, f_score=5, m_score=5`.
- Task 3 → top row `customer_id=1` at `rfm_total=14` (tied with 2, 3, 4); bottom two rows `customer_id 22, 24` at `rfm_total=3`.
- Task 4 → 5 distinct `rfm_segment` values, counts summing to 30.
- Task 5 → `Champions` is the highest-total-value segment.
- Task 6 → exactly 2 rows, both with the `9999` sentinel intact.

## Done when…

- [ ] `exercise-01.sql` has all 6 queries under `-- Task N` comments.
- [ ] Every result matches the spot checks above — if one doesn't, trace it back to a specific `CASE`/`NTILE` clause before moving on.
- [ ] Your Task 1 CTE uses `LEFT JOIN`, not `JOIN` — confirm by checking your Task 1 result really does have 30 rows, not 28.
- [ ] You did not open a spreadsheet at any point.

## Stretch

- Recompute the whole pipeline using `NTILE(4)` (quartiles) instead of `NTILE(5)` (quintiles). How does the `Champions` segment's membership change? Is 5 buckets clearly better than 4 for a 30-row dataset, or is that itself a judgment call?
- Write a single query that returns, for each `rfm_segment`, the **single customer with the lowest `monetary`** in that segment — the "weakest member" of each tier. Are any of those weakest members borderline enough that you'd argue for moving them to a different tier by hand? Note your reasoning in a comment.

## Submission

Commit `exercise-01.sql` to your portfolio under `c38-week-07/exercise-01/`.
