# Week 7 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 8. A mix of multiple-choice and short "what would this query return?" — the answer key explains the *why*, not just the letter.

---

**Q1.** What are the three axes RFM scores a customer on, in order of priority?

- A) Revenue, Frequency, Margin
- B) Recency, Frequency, Monetary
- C) Retention, Frequency, MRR
- D) Recency, Feature-usage, Monetary

---

**Q2.** In Crunch Flow's seed data, customers `22` and `24` have never placed an order. In the `rfm_base` CTE, what should `frequency` and `monetary` show for them, and why?

- A) `NULL` and `NULL` — there's nothing to compute
- B) `0` and `0` — `COUNT` over zero matching rows is naturally `0`, but `SUM` needs `COALESCE(..., 0)` because `SUM` over zero rows returns `NULL`
- C) `1` and `0` — every customer gets credit for one implicit "signup" order
- D) The rows should be excluded from the result entirely

---

**Q3.** Why is `r_score` computed as `6 - NTILE(5) OVER (ORDER BY recency_days)` instead of just `NTILE(5) OVER (ORDER BY recency_days)`?

- A) It's a stylistic choice with no functional effect
- B) `NTILE` puts the *smallest* `recency_days` (best, most recent) into bucket 1, but RFM convention says a score of 5 means "best" — the `6 -` flip corrects the direction
- C) SQLite's `NTILE` counts backwards compared to PostgreSQL's
- D) It converts days into weeks before scoring

---

**Q4.** A customer has never placed an order and gets a `recency_days` sentinel of `9999` in the RFM pipeline. Why does Lecture 3 `.clip(upper=365)` that same column before running k-means, instead of feeding `9999` straight into `StandardScaler`?

- A) `StandardScaler` errors on any value above `1000`
- B) A `9999` outlier would dominate the mean/standard-deviation computation, compressing every other customer's real variation into a rounding error — the sentinel is right for sorting but wrong for a distance-based scale
- C) k-means requires all inputs to be under 365
- D) It has no effect either way; the clip is decorative

---

**Q5.** Given the seed data, what is the `rfm_segment` classification for a customer with `r_score = 5, f_score = 5, m_score = 5`?

- A) At Risk
- B) Loyal Core
- C) Champions
- D) New/Promising

---

**Q6.** Per Lecture 1's segment-value rollup, which `rfm_segment` has the **highest total `monetary`** across its members, despite not being the largest segment by customer count?

- A) Loyal Core (8 customers)
- B) Champions (5 customers)
- C) At Risk (6 customers)
- D) Hibernating (6 customers)

---

**Q7.** In the behavioral tier `CASE` expression, why is `days_since_last_event` checked **before** `power_events`?

- A) SQL requires numeric comparisons before string comparisons in a `CASE`
- B) So a customer with a historically high power-event count but no recent activity is correctly classified `Dormant`, not `Power User` — recency gates everything else
- C) It has no effect on the result either way
- D) `power_events` is only defined for customers who are still active

---

**Q8.** What three `event_name` values make up "power events" in this week's schema?

- A) `task_created`, `comment_added`, `report_created`
- B) `automation_triggered`, `integration_connected`, `api_call`
- C) `invite_sent`, `template_used`, `report_created`
- D) `signup`, `activated`, `upgraded`

---

**Q9.** In the RFM × behavioral cross-tab, every single `At Risk` customer (all 6) landed in which `behavior_segment`?

- A) Power User
- B) Core User
- C) Cooling Off
- D) Dormant

---

**Q10.** What does that 100% agreement in Q9 demonstrate?

- A) The two segmentations are redundant and one should be dropped
- B) Purchase-history and product-usage signals independently confirm the same finding for this segment, which makes it a high-confidence target rather than a coincidence from one data source
- C) It's a bug — `At Risk` should be spread across multiple behavior tiers
- D) `At Risk` customers always place at least one order per month

---

**Q11.** Why must every numeric feature be run through `StandardScaler` before k-means clustering, when RFM's `NTILE` scoring in Lecture 1 needed no such step?

- A) `NTILE` and k-means are mathematically identical, so scaling is redundant either way
- B) `NTILE` already produces same-scale 1–5 outputs by ranking within each column separately; k-means computes raw Euclidean *distance* across columns simultaneously, so a column measured in bigger raw units (like dollars) would silently dominate the distance calculation unless every column is rescaled to a comparable range first
- C) Scaling is only needed when using PostgreSQL, not SQLite
- D) `StandardScaler` is required by scikit-learn's API and has no mathematical purpose

---

**Q12.** In the elbow/silhouette sweep over Crunch Flow's 30 customers (`k = 2` through `7`), at which `k` does the silhouette score peak?

- A) `k = 2`
- B) `k = 4`
- C) `k = 5`
- D) `k = 7`

---

**Q13.** The k=5 clustering solution recovers `Champions` and `New/Promising` as clean, separate clusters almost exactly matching the RFM tiers — but `Loyal Core` and `At Risk` largely merge into one cluster. What's the most defensible explanation?

- A) The clustering code has a bug in how it handles the `Loyal Core` segment specifically
- B) `Loyal Core` and `At Risk` customers have very similar *snapshot* values across the five clustering features; the real difference between them is a change over time (declining vs. steady), which a single point-in-time feature matrix cannot represent
- C) `k=5` is simply the wrong number of clusters and `k=8` would fix it
- D) `Loyal Core` and `At Risk` are not real segments and should be merged in the RFM tiering too

---

**Q14.** Why is "sort all customers by lifetime spend and call the top N Champions" a weaker approach than the full RFM tiering this week builds?

- A) Lifetime spend is impossible to compute in SQL
- B) A single high-value historical purchase can rank a customer above people who buy smaller amounts regularly and recently — spend-only sorting ignores whether the customer is still engaged at all
- C) SQL's `ORDER BY` cannot sort by a `SUM` aggregate
- D) There is no meaningful difference — RFM and spend-sorting always produce the same ranking

---

**Q15.** A cluster from a k-means run is labeled `cluster == 2` with no further interpretation. Per Lecture 3 section 6, why is that label, by itself, not yet useful to a CS or sales team?

- A) Cluster numbers are randomly reassigned every time the model runs, so `2` means nothing consistent
- B) A cluster index is not a persona — it has to be translated by a human, grounded in the cluster's actual feature means and a sample of real customer rows, into a name and story a non-technical team member can act on
- C) k-means clusters cannot be joined back to `customer_id`
- D) Only `k=5` solutions can be interpreted; other values of `k` produce meaningless clusters

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — Recency, Frequency, Monetary, in that priority order; Monetary is scored last on purpose because a single big purchase can mislead if recency and frequency both say the customer hasn't returned.
2. **B** — `COUNT` naturally returns `0` over zero matching rows, but `SUM` returns `NULL` over zero matching rows unless wrapped in `COALESCE(..., 0)` — a classic aggregate-`NULL` trap.
3. **B** — `NTILE`'s bucket 1 always holds the lowest sort-key values; since low `recency_days` is the *good* outcome, the score has to be inverted so "most recent" maps to score 5, matching RFM convention.
4. **B** — `StandardScaler` computes a mean and standard deviation across the column; leaving the `9999` sentinel in place would let two extreme outliers dominate that computation and compress every other customer's real signal.
5. **C** — `r_score=5, f_score=5, m_score=5` clears the `>= 4` bar on all three axes, which is exactly the `Champions` definition.
6. **B** — Champions, despite being the smallest of the five segments (5 customers), has the highest total `monetary` (~$4,584) — a small group carrying outsized value is the headline finding of Lecture 1.
7. **B** — checking recency first ensures a formerly-heavy user who's gone quiet is correctly flagged `Dormant`; checking `power_events` first would misclassify a churned power user as still engaged.
8. **B** — `automation_triggered`, `integration_connected`, and `api_call` are the three deep-adoption "power" actions; the other five events are core actions any plan/skill level can perform.
9. **D** — all 6 `At Risk` customers are behaviorally `Dormant`.
10. **B** — two independent data sources (purchase history and product usage) agreeing is a stronger, higher-confidence signal than either alone, and is exactly the kind of finding worth acting on with confidence.
11. **B** — `NTILE` ranks within a single column at a time, so its outputs are already comparable across columns; k-means computes distance across all columns *simultaneously*, so unscaled columns with bigger raw units would dominate the distance metric unless rescaled first.
12. **C** — silhouette peaks at `k=5` (≈0.41) across the 2–7 sweep on this dataset.
13. **B** — the two groups look similar in a single snapshot; separating "steady" from "declining" needs a trend feature (e.g., recent vs. prior-period activity), which the five snapshot features don't encode.
14. **B** — spend-only ranking can put a customer with one big historical purchase above someone who buys smaller amounts often and recently, which is exactly the trap RFM's three-axis design exists to avoid.
15. **B** — a bare cluster index means nothing to a non-technical stakeholder until a human interprets its feature means and sample rows into a named persona with a concrete story.

</details>

**Scoring:** 13+ → start Week 8. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; Week 8 (pricing and packaging experiments) assumes you can name and defend a customer segment without hesitating.
