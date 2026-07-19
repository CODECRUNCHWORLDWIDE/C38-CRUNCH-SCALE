# Exercise 2 — Project Retention-Adjusted Revenue

**Goal:** Build a pooled cohort revenue-retention curve correctly (weighted, right-censoring-aware, segmented by curve), then use flat continuation to project how much of today's customer base will still be paying a year from now.

**Estimated time:** 1 hour.

## Setup

```sql
SELECT COUNT(*) FROM cohort_revenue_retention;   -- must print 57
SELECT COUNT(DISTINCT cohort_month) FROM cohort_revenue_retention;   -- must print 6
```

Create `solutions.sql` for the SQL tasks and `solutions.py` for the pandas tasks.

## Tasks

1. **The naive trap.** Write the naive `AVG(100.0 * active_cohort_mrr / starting_cohort_mrr)` grouped by `months_since_signup`, alongside `COUNT(*)` (number of cohorts contributing at that age). *(Expected: age 0–6 has all 6 cohorts; age 11 has exactly 1. Write one sentence on why the age-11 "average" is misleading.)*

2. **The weighted pooled curve.** Fix Task 1 by weighting: `100.0 * SUM(active_cohort_mrr) / SUM(starting_cohort_mrr)` grouped by `months_since_signup`, across **all six cohorts** (don't segment yet). *(Expected: age 0 = 100.0%, and the curve should show the "smile" — dip then partial recovery — same as the January cohort alone, just smoother.)*

3. **Segment by curve.** Split into two pooled curves: `legacy_onboarding` (`cohort_month < '2025-06-01'`) and `new_onboarding` (`cohort_month >= '2025-06-01'`), each weighted the same way as Task 2. Compare the two curves at every age they both cover (0–6). *(Expected: at age 3, legacy ≈90.5%, new-onboarding ≈93.5%; at age 6, legacy ≈89.7%, new-onboarding ≈94.0%. New-onboarding is ahead at every overlapping age.)*

4. **Flat-continuation value.** For each curve from Task 3, find its **last observed age** and the retention percentage at that age — this is the value you'll hold flat for any older age you haven't observed. *(Expected: legacy's last observed age is 11, value ≈98.2%; new-onboarding's last observed age is 6, value ≈94.0%.)*

5. **Project the six 2025 cohorts forward, in pandas.** Load each 2025 cohort's `starting_cohort_mrr` — for Jan–Jun, read it from `cohort_revenue_retention` at age 0; for Jul–Dec (which have **no** cohort-retention rows at all — they hadn't been observed at the December 2025 cutoff), read the cohort's starting MRR from `mrr_bridge_actuals.new_mrr` for that month instead. For each of the twelve 2025 cohorts, compute its age as of **December 2026**, pick the correct curve (legacy if signed up before June 2025, new-onboarding otherwise), apply the Task 4 flat-continuation value (every 2025 cohort's December-2026 age exceeds both curves' observed range), and compute `projected_mrr_at_target = starting_cohort_mrr * flat_value`. *(Expected total across all twelve cohorts: ≈$43,488 — matching Lecture 2. If your total is off by more than a few dollars, check you applied the *new-onboarding* flat value, not the legacy one, to the six Jul–Dec cohorts.)*

6. **Compare to the bridge forecast.** Lecture 1's base-case bridge forecast projects December 2026 ending MRR at ≈$144,946. What fraction of that is "already baked in" from customers who exist today (your Task 5 answer), and what fraction has to come from customers Crunch Flow hasn't sold yet? Write two sentences.

## Expected result (spot checks)

- Task 3 → new-onboarding curve leads legacy at every overlapping age.
- Task 4 → legacy flat ≈98.2% (age 11), new-onboarding flat ≈94.0% (age 6).
- Task 5 → total 2025-cohort contribution to Dec 2026 MRR ≈$43,488 (Jan–Jun legacy cohorts ≈$20,552; Jul–Dec new-onboarding cohorts ≈$22,936).
- Task 6 → roughly 30% baked in, ~70% still has to be sold.

## Done when…

- [ ] `solutions.sql` has Tasks 1–4.
- [ ] `solutions.py` has Tasks 5–6.
- [ ] You can explain, without looking it up, why the Jul–Dec 2025 cohorts use `new_mrr` from the bridge table instead of a row in `cohort_revenue_retention`.
- [ ] Your Task 5 total lands within a few dollars of $43,488.

## Stretch

- Redo Task 5 assuming Crunch Flow's onboarding improvement is *even better* than observed — i.e., assume the new-onboarding curve keeps rising past age 6 at half its age-4→6 rate of change, instead of flattening. How much higher does the Dec 2026 baked-in total get? Which assumption (flat vs. rising) would you actually put in a memo, and why?
- Build the same projection for **June 30, 2026** instead of December 2026 (a 6-month-out check instead of 12). Does the legacy curve need flat continuation for this shorter horizon, or does real observed data cover it?

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c38-week-11/exercise-02/`.
