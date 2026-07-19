# Week 7 — Homework

Five problems, ~5 hours total, spread across the week. A mix of hands-on SQL, hands-on pandas, and written analysis against the Crunch Flow seed. Commit each.

All queries run against the `customers`/`orders`/`product_events` seed from the [README](./README.md) unless a problem says otherwise. "Today" is **2025-12-31**.

---

## Problem 1 — Reproduce every headline number (60 min)

Without looking back at the lecture files, write fresh SQL for each of these from memory (peek only if truly stuck, then note which ones you peeked at):

1. Full RFM scoring query (recency, frequency, monetary, `NTILE(5)` scores, `rfm_segment` tier), all 30 customers.
2. Full behavioral profile query (total events, distinct features, power events, days since last active, `behavior_segment` tier), all 30 customers.
3. Segment-value rollup: for each `rfm_segment`, `n` and total `monetary`.
4. Behavior-tier rollup: for each `behavior_segment`, `n` and average `total_events`.

**Deliver** `reproduce.sql` with all 4 queries and a one-line note on which (if any) you needed to check a lecture for.

---

## Problem 2 — Segment by plan and industry (75 min)

This week's lectures never broke segments down by `customers.plan` or `customers.industry`. This problem does.

1. Cross-tab `rfm_segment` against `plan` — for each `(rfm_segment, plan)` pair, a count. Which plan tier is most concentrated in `Champions`? Is that surprising, given `plan` directly sets `mrr` and `mrr` doesn't factor into the RFM calculation at all?
2. Cross-tab `behavior_segment` against `industry`. Is there an industry that stands out as unusually `Dormant` or unusually `Power User`-heavy? (With only 1–3 customers per industry in this seed, be honest in your write-up about whether a pattern with `n=2` is a real finding or just noise — this is a real judgment call analysts make constantly with small samples.)
3. Write three sentences: does `plan` (i.e., how much a customer already pays) predict RFM tier, behavioral tier, neither, or both? What would that finding mean for how Crunch Flow's sales team should — or shouldn't — use plan tier as a proxy for "this customer is healthy"?

**Deliver** `by-plan-industry.sql` (the 2 queries) plus `by-plan-industry-notes.md` (the three sentences plus your honesty check on Question 2).

---

## Problem 3 — Explain the segmentation, precisely (45 min)

In `metric-writeup.md`, answer in prose (no more than 450 words total):

1. Write the `rfm_segment = 'Champions'` definition precisely enough that a new analyst, with no other context, could implement the exact `CASE` branch from your sentence alone.
2. Explain, in your own words, why `NTILE(5)`'s quintile boundaries are sample-relative rather than absolute — use a concrete "what would change if 10 new low-spend customers signed up tomorrow" example.
3. Explain why `StandardScaler` is necessary before k-means but is *not* something you'd ever apply before an `NTILE`-based RFM score — what's structurally different about how the two techniques use the numbers?
4. Name one customer from this week's seed data whose segment assignment (by any of the three techniques) you personally disagree with, and defend your alternative call using at least one number.

---

## Problem 4 — A second clustering pass (75 min)

Lecture 3 clustered on 5 features: `recency_days`, `frequency`, `monetary`, `total_events`, `power_events`. This problem asks whether a *smaller* feature set tells a meaningfully different story.

1. Re-run the k-means pipeline using **only the two RFM-purchase features** (`recency_days`, `monetary` — drop `frequency`, `total_events`, `power_events` entirely). Sweep `k=2..6`, report the best `k` by silhouette.
2. Re-run using **only the two behavioral features** (`total_events`, `power_events` — drop everything else). Sweep the same range.
3. Compare both 2-feature solutions' cluster assignments against Lecture 3's full 5-feature `k=5` solution. Does dropping down to 2 features change which customers cluster together, or does the story hold up? Put your answer, with a short comparison table, in `cluster-comparison.md`.
4. One paragraph: when would a simpler, 2-feature clustering be the *right* engineering choice over the full 5-feature version, even if it's slightly less rich? (Think about interpretability, maintenance, and who has to explain the model to a non-technical stakeholder.)

**Deliver** `cluster-comparison.py` (both re-runs) and `cluster-comparison.md` (the comparison + paragraph).

---

## Problem 5 — Extend the dataset (75 min)

Make the data your own and query it — same spirit as prior weeks' extension homework, applied to a customer-segmentation schema.

1. Invent **3 new Crunch Flow customers** (`customer_id` 31, 32, 33), one that should clearly land in `Champions`, one that should clearly land in `Hibernating`, and one that's a genuinely ambiguous edge case between two tiers.
2. Write realistic `orders` rows for each (at least 2 orders for the Champion, 0–1 for the Hibernating customer, and enough for the edge case that its tier assignment is a real judgment call).
3. Write realistic `product_events` rows for each, consistent with the behavioral story you intend (a Champion should show power-event usage; a Hibernating customer's last event should be well over 60 days old).
4. Load them (you now have 33 customers) and recompute Problem 1's four headline queries. Confirm your Champion and Hibernating customers land where you intended — if they don't, that's useful information about how sensitive the tier boundaries are, not a bug to hide.
5. Report, in `extend-notes.md`, which tier your "ambiguous" customer actually landed in, and whether that matches your own gut call before you ran the query.
6. Then **undo** it cleanly: delete your 3 customers' orders and events first (foreign keys), then the customers, and confirm you're back to 30 customers / 125 orders / 393 events.

**Deliver** `extend.sql` (inserts, the 4 recomputed queries, and the cleanup) plus `extend-notes.md` (what happened with the ambiguous customer).

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 75 min |
| 5 | 75 min |
| **Total** | **~5.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
