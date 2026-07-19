# Exercise 3 — Build a Three-Scenario Model

**Goal:** Turn Lecture 3's base/bull/bear assumption tables into working code — SQL recursive CTE **and** pandas — that produces a quarterly comparison Finance could actually read.

**Estimated time:** 1 hour.

## Setup

You'll need December 2025's `ending_mrr` as the anchor for every scenario:

```sql
SELECT ending_mrr FROM mrr_bridge_actuals WHERE month = '2025-12-01';  -- 79270
```

Create `solutions.sql` and `solutions.py`.

## The three assumption sets

Use exactly these — don't tweak them yet (that's what Challenge 2 is for):

| Assumption | Bear | Base | Bull |
|---|---:|---:|---:|
| January `new_mrr` | $4,800 | $5,350 | $5,900 |
| `new_mrr` monthly growth | +$50 | +$150 | +$250 |
| Expansion rate | 2.40% | 3.20% | 3.80% |
| Contraction rate | 1.10% | 0.85% | 0.70% |
| Churn rate | 3.80% | 3.00% | 2.40% |

## Tasks

1. **One scenario in SQL.** Write a `WITH RECURSIVE` query (following Lecture 1 §4's pattern) that rolls the **base case** forward twelve months from the December 2025 anchor, and returns one row per month with `starting_mrr`, `new_mrr`, `expansion_mrr`, `contraction_mrr`, `churned_mrr`, `ending_mrr`. *(Expected: month 12 `ending_mrr` ≈$144,946.)*

2. **All three scenarios in SQL.** Extend Task 1 into three separate recursive CTEs (or one parameterized query run three times) for bear, base, and bull. Produce a single result set with a `scenario` column and one row per `(scenario, month_num)`. *(Expected: 36 rows total.)*

3. **Quarterly rollup.** From Task 2's output, produce a table with one row per quarter and three columns (`bear`, `base`, `bull`), showing each scenario's **ending MRR at the end of that quarter** (i.e., month 3, 6, 9, 12 within each scenario). *(Expected quarterly ending MRR — bear/base/bull: Q1 ≈$87,664/$94,129/$99,522; Q2 ≈$95,883/$110,042/$122,469; Q3 ≈$103,939/$126,987/$148,166; Q4 ≈$111,845/$144,946/$176,673.)*

4. **The spread.** For each quarter, compute `bull - bear` (the dollar width of the range) and `(bull - bear) / base` (the width as a percentage of the base case). *(Expected: the percentage spread grows every quarter — by how much, roughly, from Q1 to Q4?)*

5. **Reproduce in pandas.** Write a `run_scenario()` function (following Lecture 3 §4) and use it to build the same three scenarios and quarterly rollup as Tasks 2–3, entirely in pandas. Confirm your SQL and pandas numbers match to within rounding.

6. **Which assumption matters most?** Holding all other bear-case assumptions at their base-case values, swap in *only* the bear `new_mrr` assumptions (starting value and growth) and recompute Q4 ending MRR. Then reset and swap in *only* the bear churn/expansion/contraction rates instead, holding `new_mrr` at base. Which single swap moves Q4 ending MRR further from the base case? *(This isolates which lever the scenario spread is really coming from — write two sentences on what you find.)*

## Expected result (spot checks)

- Task 1 → base-case Q4 ending MRR ≈$144,946.
- Task 3 → matches the table in the task description above.
- Task 4 → Q4 spread ≈ $64,828, ≈45% of base.
- Task 6 → the `new_mrr` swap should move Q4 ending MRR substantially more than the rate-only swap — because Crunch Flow's NRR sits just under 100% (Lecture 1 §2), almost all of its net growth comes from new logos, not from the existing base compounding.

## Done when…

- [ ] `solutions.sql` has Tasks 1–4.
- [ ] `solutions.py` has Task 5 and Task 6.
- [ ] Your SQL and pandas Q4 numbers for all three scenarios match within a few dollars of each other.
- [ ] You can state, from memory, which single assumption drives most of the bull/bear spread — and why that follows from Crunch Flow's NRR being just under 100%.

## Stretch

- Add a fourth scenario, `bear_severe`, where churn jumps to 5% starting in Q2 2026 (simulating a major competitor launch) while everything else stays at the bear-case values. How much does Q4 ending MRR fall relative to the regular bear case?
- Instead of holding each rate constant for all 12 months, let the bull-case expansion rate **ramp** linearly from the base-case value (3.2%) in January to the full bull value (3.8%) by June, then hold there. How much does this softer ramp change the Q4 bull number versus assuming the higher rate from month 1?

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c38-week-11/exercise-03/`.
