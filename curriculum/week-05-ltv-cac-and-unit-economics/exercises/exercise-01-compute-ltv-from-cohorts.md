# Exercise 1 — Compute LTV from Cohort Retention

**Goal:** Build the pooled, age-indexed retention curve from raw `customers` rows in SQL, then turn it into a margin-based LTV two different ways, exactly as Lecture 1 did — but this time you write every query.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM customers;   -- must print 64
```

Create `solutions.sql` for the SQL tasks and `analysis.py` (or a notebook) for the pandas tasks. Label each with a `-- Task N` / `# Task N` comment.

## Tasks

### SQL (Tasks 1–4)

1. **Still active.** Count customers per channel where `churn_month IS NULL`. *(Expected: `paid_search` 19, `organic_content` 23.)*

2. **The retention curve.** Write the pooled age-indexed retention query from Lecture 1 §3 yourself — don't copy it verbatim, re-derive it. Produce `channel, age, n_observed, retention` for ages 0 through 6. *(Expected, spot check: `paid_search` age 3 → 59.3% retention on n=27; `organic_content` age 3 → 88.2% retention on n=17.)*

3. **Flag the noisy tail.** Add a column to your Task 2 query, `reliable`, that's `TRUE` when `n_observed >= 15` and `FALSE` otherwise. *(Expected: both channels are `reliable = FALSE` by age 6 — `paid_search` n=18 is borderline, `organic_content` n=9 is not reliable.)*

4. **Contribution margin per customer.** Select `channel`, `mrr`, and a computed `contribution_margin_per_month` = `mrr * 0.80`, one row per distinct channel/mrr pair. *(Expected: `paid_search` → 119.20; `organic_content` → 79.20.)*

### pandas (Tasks 5–7)

Bring your Task 2 retention curve into a `DataFrame` (typed in by hand from your SQL output, or loaded via `pandas.read_sql` if you're set up for it) for the rest of this exercise.

5. **Cohort-sum LTV (the floor).** Compute `Σ retention(age) × margin_per_month` for ages 0–6, per channel. *(Expected: `paid_search` ≈ $574.87; `organic_content` ≈ $471.09.)*

6. **Reciprocal-formula LTV (the projection).** Compute the average monthly churn rate from your ages-1-through-6 retention numbers (`1 - retention[i]/retention[i-1]`, averaged), then `LTV = margin ÷ avg_churn`. *(Expected: `paid_search` ≈ $1,342.73; `organic_content` ≈ $2,061.64.)*

7. **The overstatement check.** Recompute Task 6 using raw `mrr` instead of `contribution_margin_per_month`. Confirm the revenue-based LTV is exactly 25% higher than the margin-based one for both channels, and explain *why* 25% specifically (tie it to the gross margin).

## Expected result (spot checks)

- Task 1 → 19 / 23.
- Task 2 → `paid_search` age 0 = 100%, age 6 = 55.6% (n=18).
- Task 5 → `paid_search` ≈ $574.87, `organic_content` ≈ $471.09.
- Task 6 → `paid_search` ≈ $1,342.73, `organic_content` ≈ $2,061.64.
- Task 7 → both channels' revenue-based LTV is exactly 1.25× the margin-based LTV.

## Done when…

- [ ] `solutions.sql` has all 4 SQL tasks, each returning the expected row counts and values.
- [ ] `analysis.py` has all 3 pandas tasks with printed output matching the expected values within rounding.
- [ ] You can explain, out loud, why the Task 5 number is smaller than the Task 6 number for the *same* channel.
- [ ] You can explain why the Task 7 overstatement factor is 1.25 and not some other number.

## Stretch

- Add ages 7–11 to your Task 2 curve and note where `n_observed` drops below 10. At what age would you personally stop trusting the curve for `organic_content`? For `paid_search`?
- Recompute Task 6's LTV assuming gross margin were 65% instead of 80% (a more typical margin for a services-heavy SaaS product). How much does LTV drop, in dollars and in percent?

## Submission

Commit `solutions.sql` and `analysis.py` to your portfolio under `c38-week-05/exercise-01/`.
