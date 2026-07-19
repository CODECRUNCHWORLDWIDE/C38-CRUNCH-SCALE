# Exercise 1 — Build an MRR Bridge

**Goal:** Prove the bridge reconciles before you trust anything downstream of it, then extract the rates that Lecture 1's forecast is built from. By the end, `LAG`/window-function rate math over a bridge table feels automatic.

**Estimated time:** 1 hour.

## Setup

Confirm you're connected and both tables are there:

```sql
SELECT COUNT(*) FROM mrr_bridge_actuals;       -- must print 12
SELECT COUNT(*) FROM cohort_revenue_retention; -- must print 57
```

Create a file `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Reconcile the bridge.** Write a query that returns, for every month, `starting_mrr + new_mrr + expansion_mrr - contraction_mrr - churned_mrr` alongside the stored `ending_mrr`, and a boolean/flag column showing whether they match. *(Expected: all 12 rows match — the seed reconciles by construction. If yours don't, you mistyped a number; fix it before continuing.)*

2. **Verify the chain.** Write a query using `LAG(ending_mrr) OVER (ORDER BY month)` that checks every month's `starting_mrr` equals the *previous* month's `ending_mrr` (the first month, January, has no previous row and should be excluded or shown as `NULL`). *(Expected: all 11 comparable rows match.)*

3. **Monthly rates.** For every month, compute `expansion_rate_pct`, `contraction_rate_pct`, `churn_rate_pct` (each as a percentage of that month's `starting_mrr`, rounded to 2 decimals) and `net_revenue_retention_pct` (`100 * (starting_mrr + expansion_mrr - contraction_mrr - churned_mrr) / starting_mrr`). *(Expected: December's NRR ≈ 99.40%.)*

4. **Find the worst month.** Using the query from Task 3, find the single month with the **lowest** `net_revenue_retention_pct`. *(Expected: August 2025, ≈98.41% — cross-check this against the `new_mrr` dip you were told about in the README; is the worst-NRR month the same as the worst-new-MRR month? Write one sentence on whether that's a coincidence.)*

5. **Trailing-3-month averages.** Using a window frame (`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`), compute the trailing-3-month average of each rate from Task 3, for every month that has at least 3 months of history (October, November, December). *(Expected: the December trailing-3-month averages are the base-case rates used throughout the lectures — expansion ≈3.19%, contraction ≈0.85%, churn ≈2.98%.)*

6. **Logo churn vs. dollar churn.** Compute the **logo (customer) churn rate** per month — `churned_customers / starting_customers` — alongside the **dollar churn rate** (`churned_mrr / starting_mrr`) from Task 3. Are they always close to each other? *(Expected: they track loosely but not exactly — a month can lose proportionally more/fewer dollars than customers if the customers who churn skew toward higher- or lower-tier plans. Find the month where the two rates diverge the most and note which direction.)*

7. **The new_mrr trend, in pandas.** Load `mrr_bridge_actuals` into a DataFrame, fit a one-variable linear trend (`numpy.polyfit`) to `new_mrr` against month index (1–12), and print the slope and intercept. *(Expected: slope ≈ +140/month.)*

## Expected result (spot checks)

- Task 1 → all 12 months reconcile exactly.
- Task 4 → August 2025, NRR ≈98.41%.
- Task 5 → December trailing-3mo: expansion ≈3.19%, contraction ≈0.85%, churn ≈2.98%.
- Task 7 → slope ≈140 (accept 130–150).

## Done when…

- [ ] `solutions.sql` has Tasks 1–6, each under a `-- Task N` comment.
- [ ] `solutions.py` (or a notebook cell) has Task 7.
- [ ] Task 1's reconciliation check shows all 12 rows matching — if it doesn't, you've found and fixed an arithmetic slip before moving on.
- [ ] You can explain, in one sentence, why Task 5's trailing average is a better forecasting input than either the single-month rate or the full-year rate.

## Stretch

- Compute **gross** dollar churn rate as a percentage of *ending* MRR instead of *starting* MRR (a metric some finance teams prefer). Does it change which month looks "worst"?
- Add a column `arpu` = `ending_mrr / ending_customers` for every month, and describe in one sentence what its trend across the year suggests about Crunch Flow's plan mix shifting.

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c38-week-11/exercise-01/`.
