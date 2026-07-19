# Week 11 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on SQL/pandas, a written explanation, and an extend-the-dataset task. Commit each.

All queries run against the `mrr_bridge_actuals` and `cohort_revenue_retention` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Rebuild the bridge from scratch, twice (60 min)

Get comfortable with the recursive-CTE pattern by writing it cold, without the lecture open.

1. In `bridge_cold.sql`, write a `WITH RECURSIVE` query that rolls the base-case bridge forward from December 2025's actual `ending_mrr`, using the trailing-3-month rates you compute yourself (don't hardcode Lecture 1's numbers — recompute them from `mrr_bridge_actuals`).
2. In `bridge_cold.py`, write the equivalent pandas loop, computing the same trailing rates independently in pandas (don't copy the SQL results across).
3. Confirm both give the same Q4 2026 ending MRR (within a few dollars).

**Deliver** both files, plus one sentence on which approach (SQL recursion vs. pandas loop) you found easier to reason about, and why.

---

## Problem 2 — Twenty rate and rollup queries (75 min)

Write and run each against `mrr_bridge_actuals`. Put them in `warmups.sql` with a `-- N` comment and the answer beneath each.

1. Total `new_mrr` added across all of 2025.
2. Total `churned_mrr` across all of 2025.
3. The month with the single highest `new_mrr`.
4. The month with the single highest `churned_mrr`.
5. Full-year average `expansion_mrr` per month.
6. `ending_mrr` growth from January to December, in dollars and as a percentage.
7. The three-month period (any three consecutive months) with the highest combined `new_mrr`.
8. Every month where `contraction_mrr` exceeded `expansion_mrr` (a "net downgrade" month, if any exist).
9. Cumulative `new_mrr` by month, running total (`SUM() OVER` with an `ORDER BY` frame).
10. The month-over-month percentage change in `ending_mrr` for every month (use `LAG`).
11. Average `starting_customers` across the year.
12. Net new customers added in Q3 2025 (Jul–Sep) combined.
13. The logo churn rate (`churned_customers/starting_customers`) for the single worst month.
14. ARPU (`ending_mrr / ending_customers`) for every month.
15. The month where ARPU grew the fastest month-over-month.
16. Total dollars of "net revenue movement" (`expansion_mrr - contraction_mrr - churned_mrr`) for every month — is it ever positive (i.e., would the base have grown even with zero new sales)?
17. A quarterly rollup: `SUM(new_mrr)`, `SUM(churned_mrr)`, and `ending_mrr` at quarter-end, for all four 2025 quarters.
18. The single quarter with the best net revenue retention (aggregate the whole quarter, not an average of monthly NRRs).
19. Every cohort's `starting_cohort_mrr` at age 0, pulled from `cohort_revenue_retention` — confirm each matches the corresponding month's `new_mrr` in `mrr_bridge_actuals`.
20. The cohort with the single lowest revenue retention at age 4 — which one, and what's the percentage?

---

## Problem 3 — Explain net revenue retention (45 min)

In `nrr-writeup.md`, answer in prose (no more than 400 words total):

1. In your own words, explain the difference between **logo churn** and **dollar (revenue) churn**, and give a concrete reason the two numbers can move in different directions in the same month.
2. Crunch Flow's NRR sits just under 100% all year. Explain what that means for where the company's growth is actually coming from, and what would have to change for NRR to cross 100% and stay there.
3. Explain, to someone who's never heard the term, what "right-censored" means in the context of `cohort_revenue_retention`, using the June 2025 cohort as your example.
4. Why is "flat continuation" (Lecture 2 §3) a *conservative* choice rather than an aggressive one, when projecting a cohort's retention curve past the age you've actually observed?

---

## Problem 4 — Cross-check three ways, on a different target date (60 min)

Repeat the cross-check from the mini-project's Part D, but for **June 30, 2026** instead of December 31, 2026.

1. Rebuild the base-case bridge forecast's `ending_mrr` for June 2026 (month 6 of the forecast).
2. Rebuild the retention-adjusted cohort projection's total for June 2026 (note: several 2025 cohorts will actually have *observed* data at this shorter horizon rather than needing flat continuation — check which ones).
3. Rebuild the linear-trend point forecast (and ±1-RMSE interval) for June 2026.
4. In `cross-check-june.md`, report all three numbers and one paragraph: does a 6-month-out cross-check agree more closely across the three methods than the 12-month-out one did? Does that match your intuition about how forecast uncertainty should behave with a shorter horizon?

**Deliver** `cross-check-june.sql`/`.py` plus `cross-check-june.md`.

---

## Problem 5 — Extend the dataset with Q1 2026 "actuals" (60 min)

Make the forecast falsifiable — practice what happens when real numbers finally arrive.

1. Invent three months of "actual" data for January, February, and March 2026 (your own numbers — but keep them plausible: `ending_mrr` should chain from December 2025's $79,270, and the components should reconcile per the bridge identity). Write the `INSERT` statements into a **new** table, `mrr_bridge_actuals_2026_q1`, matching the original table's shape.
2. Compare your invented Q1 2026 actuals against the base-case forecast from Part A (Lecture 1 / Exercise 3). Compute the dollar and percentage variance for each month.
3. Write two versions of your invented data: one where actuals come in **within** your bear/bull range for those months, and one where they come in **outside** it (below bear or above bull).
4. For the "outside the range" version, write two sentences: per the invalidation condition you'd state in a real forecast memo (Lecture 3 §5, Part 5), would this trigger a rebuild? What's the first thing you'd investigate?

**Deliver** `extend_2026.sql` (the inserts, both versions) and `extend_2026.md` (the variance analysis and the two sentences).

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 60 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
