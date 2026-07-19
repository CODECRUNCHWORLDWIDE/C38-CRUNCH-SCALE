# Exercise 1 — Engineer Churn Features from Events

**Goal:** Build the leakage-free feature table and label from `raw_events` and `raw_subscriptions`, exactly as of the `2025-09-01` cutoff, and confirm you get the same population and base rate as the lecture.

**Estimated time:** 90 minutes.

## Setup

`crunch_flow_scale.db` is seeded (260 users, 9095 events). Work in a `solutions_01.sql` file — one query per task, each preceded by a `-- Task N` comment.

## Tasks

1. **Confirm the raw counts.** Run the four sanity checks below. *(Expected: **260**, **9095**, **260**.)*

   ```sql
   SELECT COUNT(*) FROM raw_users;
   SELECT COUNT(*) FROM raw_events;
   SELECT COUNT(*) FROM raw_subscriptions;
   ```

2. **Eligible population.** Write the `eligible` CTE from Lecture 1, Section 3 (customers with `canceled_at IS NULL OR canceled_at > '2025-09-01'`). *(Expected: **236** rows.)*

3. **Already-churned count.** Separately, count customers who canceled **before** the cutoff (`canceled_at < '2025-09-01'`). *(Expected: **24**.)* Confirm `236 + 24 = 260` — every customer is accounted for exactly once.

4. **Full feature table.** Build the complete feature query from Lecture 1, Section 3 (`tenure_days`, `events_30d`, `events_31_60d`, `events_90d`, `support_tickets_90d`, `had_payment_failure`, `days_since_last_event`). *(Expected: **236** rows, matching Task 2.)*

5. **The label, joined separately.** Build the label query from Lecture 1, Section 4 and join it to your feature table **in pandas**, on `user_id`. Add `engagement_trend = events_30d - events_31_60d` after the join. *(Expected: **236** rows in the merged dataframe, no `NaN`s introduced by the merge.)*

6. **Base rate.** Compute `df["churned_in_window"].mean()`. *(Expected: **0.369** — 87 of 236 churn during the label window.)*

7. **Find the sentinel rows.** Query for customers where `days_since_last_event = 999` (no pre-cutoff activity at all). *(Expected: **2** customers, both with `tenure_days` under 50 — brand-new signups with no usage history yet.)* In a one-sentence comment, explain why a sentinel value here is a reasonable modeling choice rather than a bug to "fix" by imputing a number like 0 or the column mean.

8. **A leakage gut-check.** Write a query that would have (incorrectly) counted `feature_used` events for the **label window** (`event_ts >= '2025-09-01' AND event_ts <= '2025-12-31'`) into an `events_30d`-style feature. Run it for user `1` and compare the (wrong) count against the correct pre-cutoff `events_30d` you computed in Task 4. In a one-sentence comment, explain concretely what would go wrong if you shipped a model trained on the leaky version.

## Done when…

- [ ] Tasks 1–3 confirm the 260 / 24 / 236 split exactly.
- [ ] Task 4's feature table has 236 rows and every numeric column is non-negative.
- [ ] Task 6 prints a base rate of 36.9% (±0.1 pt for rounding).
- [ ] Task 7 identifies exactly 2 sentinel rows with a written explanation.
- [ ] Task 8 shows a concrete, non-trivial difference between the leaky and correct counts for user 1, with a written explanation of the consequence.

## Stretch

- Add a `payment_failed_30d` feature (a payment failure specifically in the last 30 days, as opposed to `had_payment_failure`'s "ever") and check whether it's a duplicate signal of `support_tickets_90d` or adds something new — cross-tabulate the two as binary flags.
- Rebuild the feature table with a `CUTOFF` one month earlier (`2025-08-01`) and compare the eligible population size and base rate. Does moving the cutoff earlier make the prediction problem easier or harder, and why?
- Add a `plan_tenure_interaction` feature (`tenure_days` divided into short/medium/long buckets, crossed with `plan`) and inspect whether short-tenure Starter customers look meaningfully different from long-tenure Starter customers on `events_90d`.

## Submission

Commit `solutions_01.sql` and the resulting `churn_features.csv` (Task 5's merged dataframe, saved with `df.to_csv(...)`) to your portfolio under `c38-week-10/exercise-01/`.
