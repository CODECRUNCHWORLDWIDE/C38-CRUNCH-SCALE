# Exercise 2 — Build a Behavioral Cohort

**Goal:** Write the feature-usage profile and behavior-tier pipeline from Lecture 2 yourself, then cross-tab it against Exercise 1's RFM tiers to find where the two segmentations agree and where they disagree.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM product_events;   -- must print 393
```

"Today" is still **2025-12-31**. Create `exercise-02.sql`. You'll need your Exercise 1 `rfm_segment` query again for Task 5 — either paste it in as a CTE or save it as a view (`CREATE VIEW rfm_tier AS ...`) so you're not retyping it.

## Tasks

1. **Feature-usage profile.** For every customer: `total_events`, `distinct_features` (`COUNT(DISTINCT event_name)`), `power_events` (count of `automation_triggered`, `integration_connected`, `api_call`), and `days_since_last_event`. Use `LEFT JOIN` from `customers`. *(Expected: 30 rows. Spot-check customer `3` — Fjord Tech — should show `total_events = 24`, `power_events = 14`, `days_since_last_event = 2`.)*

2. **The zero-order customers, behaviorally.** Customers `22` and `24` had zero orders in Exercise 1. Do they *also* have zero product events? Run your Task 1 query filtered to just those two customers and report their `total_events`. *(Expected: customer `22` has `total_events = 7`; customer `24` has `total_events = 9` — both nonzero. Write one sentence: what does it mean that these two customers use the product but have never bought an add-on?)*

3. **Behavior tier.** Add the `behavior_segment` `CASE` expression from Lecture 2 section 4 (`Power User` / `Core User` / `Cooling Off` / `Dormant`), checking recency **before** power-event depth. *(Expected 4 segments with sizes: Power User 7, Core User 9, Cooling Off 4, Dormant 10 — sums to 30.)*

4. **Feature-breadth leaderboard.** Return the top 5 customers by `distinct_features` descending, ties broken by `total_events` descending. *(Expected: the top group is tied at `distinct_features = 8` — this should include customers `1` through `5`, the Champions from Exercise 1.)*

5. **RFM × behavior cross-tab.** `JOIN` your Task 3 result against Exercise 1's `rfm_segment` result on `customer_id`, `GROUP BY rfm_segment, behavior_segment`, and count. *(Expected: the `At Risk` × `Dormant` cell should be `6` — every single `At Risk` customer is behaviorally dormant. The `Champions` × `Power User` cell should also be `6`... check this one carefully against the lecture's number and explain any discrepancy you find.)*

## Expected result (spot checks)

- Task 1 → customer 3: `total_events=24, power_events=14, days_since_last_event=2`.
- Task 2 → customers 22 and 24 have 7 and 9 events respectively, despite zero orders.
- Task 3 → 4 segments summing to 30: Power User 7 / Core User 9 / Cooling Off 4 / Dormant 10.
- Task 4 → top breadth tier (8 distinct features) includes customers 1–5.
- Task 5 → `At Risk` × `Dormant` = 6.

## Done when…

- [ ] `exercise-02.sql` has all 5 queries under `-- Task N` comments.
- [ ] Task 3's `CASE` checks `days_since_last_event` **before** `power_events` — verify by finding one customer in your data with high `power_events` but a `Dormant` classification, and confirm that's correct given the recency-first priority.
- [ ] Task 5 actually joins to Exercise 1's segmentation, not a re-derived approximation of it.
- [ ] You caught and explained the deliberate discrepancy planted in Task 5's expected result (see the Stretch section if you're stuck).

## Stretch

- Task 5's prompt claims `Champions` × `Power User` should be `6`. **It is not — verify this yourself and find the actual number.** One Champion customer, by the strict behavior-tier definition, does not clear the `power_events >= 5` bar even though their RFM tier is `Champions`. Find that customer, report their `power_events` count, and write two sentences: is this a data problem, a threshold problem, or a genuinely interesting edge case worth a human's attention?
- For customers in `New/Promising` (Exercise 1), what's the distribution of `behavior_segment`? Are new customers behaviorally uniform, or already splitting into different engagement patterns this early? What would you want to know, operationally, if a brand-new customer's very first weeks already show `Cooling Off`?

## Submission

Commit `exercise-02.sql` to your portfolio under `c38-week-07/exercise-02/`.
