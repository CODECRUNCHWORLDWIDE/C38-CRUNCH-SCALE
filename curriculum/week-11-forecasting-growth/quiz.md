# Week 11 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before moving on. A mix of multiple-choice and short "what does this compute?" — the answer key explains the *why*, not just the letter.

---

**Q1.** In the MRR bridge identity `ending_mrr = starting_mrr + new_mrr + expansion_mrr − contraction_mrr − churned_mrr`, what must be true for month *n+1*'s `starting_mrr`?

- A) It's independently assumed each month.
- B) It equals month *n*'s `ending_mrr`.
- C) It equals the year's average `ending_mrr`.
- D) It resets to the January value every quarter.

---

**Q2.** Crunch Flow's net revenue retention (NRR) sits just under 100% through all of 2025. What does that tell you?

- A) The company is shrinking overall.
- B) Existing-base expansion alone isn't enough to offset contraction and churn — net growth depends on new logos.
- C) NRR under 100% means the forecast should be discarded.
- D) The company has no churn at all.

---

**Q3.** Why does Lecture 1 express expansion, contraction, and churn as **rates against starting MRR** rather than fixed dollar amounts when projecting forward?

- A) Rates are always smaller numbers and easier to type.
- B) Dollar amounts don't scale as the MRR base compounds; rates stay roughly stable while dollar amounts naturally grow with the base.
- C) SQL cannot store dollar amounts larger than $100,000.
- D) Rates eliminate the need for a recursive CTE.

---

**Q4.** Which trailing window does Lecture 1's base case use for expansion/contraction/churn rates, and why not the full-year average instead?

- A) Trailing 1 month — most recent is always most accurate.
- B) Trailing 12 months — more data is always better.
- C) Trailing 3 months — recent enough to reflect the current business, smooth enough not to be whipsawed by any single volatile month.
- D) There is no trailing window; rates are assumed constant from a textbook.

---

**Q5.** `new_mrr` is projected differently from expansion/contraction/churn. Why?

- A) It isn't a rate against anything — it's driven by sales/marketing capacity, so it's projected on its own dollar trend instead.
- B) `new_mrr` is always zero after month 1.
- C) It's the only bridge component that can be negative.
- D) It's computed as a percentage of `churned_mrr`.

---

**Q6.** In `cohort_revenue_retention`, the January 2025 cohort has 12 observed ages while the June 2025 cohort has only 7. This situation is called:

- A) Overfitting
- B) Right-censoring
- C) Survivorship bias
- D) Seasonal adjustment

---

**Q7.** Why is a naive `AVG(pct_retained)` grouped by `months_since_signup`, across all six cohorts, misleading at the older ages (9, 10, 11)?

- A) `AVG()` is syntactically invalid in a `GROUP BY` query.
- B) At those ages, only the January cohort (sometimes the only one) has data — the "average" is really just one cohort's number.
- C) Older ages always have higher retention by definition.
- D) `AVG()` ignores `NULL` values, which corrupts the whole calculation.

---

**Q8.** The fix for Q7 is to weight the pooled curve by:

- A) The number of cohorts contributing at each age.
- B) Each cohort's `starting_cohort_mrr` (i.e., `SUM(active_cohort_mrr) / SUM(starting_cohort_mrr)`), so bigger cohorts count proportionally more.
- C) Alphabetical order of the cohort month.
- D) A fixed 1/6th weight per cohort regardless of size.

---

**Q9.** Why does Lecture 2 build **two separate** pooled curves (legacy vs. new-onboarding) instead of one blended curve across all six cohorts?

- A) SQL cannot pool more than five cohorts in one query.
- B) The June 2025 onboarding change produced a measurably different retention curve; blending it into one average would understate the improvement and misrepresent future (post-June) cohorts.
- C) Two curves are always more accurate than one, regardless of the data.
- D) The seed data has a bug that requires a workaround.

---

**Q10.** When projecting a cohort's retention forward past the last age you've actually observed, Lecture 2 recommends:

- A) Extrapolating the observed trend indefinitely, since more data is always better.
- B) Assuming retention drops to zero immediately after the last observed age.
- C) Holding the curve flat at its last observed value — a conservative choice that doesn't bet on an unconfirmed trend.
- D) Using the average of every cohort's very first observed value (age 0).

---

**Q11.** Roughly how much of December 2026's forecasted MRR does the retention-adjusted cohort projection say is "already baked in" from customers who exist today (as of the Week 11 seed data)?

- A) Almost all of it (~95%)
- B) About 30%
- C) Essentially none (~2%)
- D) Exactly half

---

**Q12.** In Lecture 3's simple linear-trend fit on `ending_mrr`, the residual RMSE is used to:

- A) Replace the point forecast entirely.
- B) Build a prediction interval that widens with the forecast horizon (via `rmse * sqrt(horizon)`), instead of presenting one number as if it were certain.
- C) Determine the slope of the trend line.
- D) Decide which month had the highest `new_mrr`.

---

**Q13.** The base-case bottoms-up bridge forecast (Lecture 1) and the naive linear-trend forecast (Lecture 3) disagree substantially by Q4 2026, with the bridge forecast higher. What's the most likely reason?

- A) One of the two methods must contain an arithmetic bug.
- B) The linear trend is a straight line and structurally cannot represent compounding growth on an expanding MRR base, which the bridge's percentage-of-base rates naturally capture.
- C) The linear trend always overestimates growth.
- D) The bridge forecast ignores churn entirely.

---

**Q14.** In the base/bull/bear scenario model, which single assumption drives the largest share of the Q4 spread between bull and bear, and why?

- A) The expansion rate — because expansion is the largest dollar line in the bridge.
- B) The `new_mrr` starting value and growth rate — because Crunch Flow's NRR sits just under 100%, so almost all net growth comes from new logos rather than the existing base compounding.
- C) The contraction rate — because contraction is the most volatile line historically.
- D) All four assumptions contribute exactly equally by construction.

---

**Q15.** A forecast memo should include a stated "invalidation condition." What is that, and why does Lecture 3 insist on it?

- A) A legal disclaimer protecting the analyst from blame.
- B) A concrete, checkable condition (e.g., "if January actual new_mrr falls below the bear-case floor") that, if it occurs, means the forecast should be rebuilt — without it, a forecast can never be shown wrong, which means it was never a real forecast.
- C) The date the forecast expires and must be deleted.
- D) A requirement that the forecast be reviewed by two people before publishing.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — the bridge only chains correctly if each month's starting point is exactly the prior month's ending point; break that and every downstream month is wrong.
2. **B** — NRR just under 100% means the existing base would shrink slightly with zero new sales; Crunch Flow's growth is sales-led, not expansion-led. This is a fact about the business, not a reason to distrust the forecast.
3. **B** — dollar amounts of expansion/contraction/churn would need re-guessing every month as the base grows; rates scale naturally with a compounding base and stay comparatively stable.
4. **C** — trailing 3 months balances recency against being whipsawed by any one volatile month (like August's dip).
5. **A** — `new_mrr` has no natural base to be a percentage of; it reflects sales/marketing output and gets its own dollar trend.
6. **B** — right-censoring: younger cohorts haven't had time to reach the older ages yet, so their later-age data simply doesn't exist yet (not "is low" — doesn't exist).
7. **B** — at age 11, only January has a data point; an "average" of one cohort is not an average, it's that cohort's number relabeled.
8. **B** — weighting by `starting_cohort_mrr` ensures bigger cohorts (in dollar terms) count proportionally more, which is what "pooled" should mean for a revenue metric.
9. **B** — a real, measured shift in the underlying process (the onboarding change) means legacy and new cohorts are genuinely different populations; blending erases the very signal you're trying to measure and use going forward.
10. **C** — flat continuation is the conservative default; extrapolating a rising trend indefinitely without confirming evidence risks materially overstating the forecast.
11. **B** — about 30% (≈$43,488 of the ≈$144,946 base-case December 2026 forecast) comes from customers who already exist; the rest has to be sold in 2026.
12. **B** — RMSE quantifies historical model error and is used to build a widening interval around the point forecast, replacing false precision with an honest range.
13. **B** — the bridge's rates apply to a growing starting-MRR base each month (compounding), while a straight line by construction adds the same fixed dollar amount every period and can't represent that acceleration.
14. **B** — because NRR sits under 100%, expansion/contraction/churn changes move the needle much less than changes to new-logo acquisition; the new-MRR assumption dominates the scenario spread.
15. **B** — a real invalidation condition makes the forecast falsifiable and accountable; without one, "the forecast was wrong" can never actually be checked against anything concrete.

</details>

**Scoring:** 13+ → start the mini-project. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; this week's methods build directly on each other.
