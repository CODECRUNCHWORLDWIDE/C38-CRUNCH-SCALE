# Exercise 1 — Build a Funnel Conversion Query

**Goal:** Turn `funnel_events` into an ordered, rated funnel from scratch — counting correctly, ordering the steps, and computing both step-over-top and step-over-previous conversion rates.

**Estimated time:** 60 minutes.

## Setup

Connected, 20 users and 59 rows in `funnel_events`. New `solutions.sql` under `-- Task N` comments.

## Tasks

1. **Raw step counts.** `GROUP BY step_name`, counting **distinct users**, not rows. *(Expected: site_visit 20, signup 16, activated 13, paid_conversion 10.)*

2. **Order the steps.** Add the `CASE step_name WHEN ... THEN n END AS step_order` mapping from Lecture 1 to Task 1's query and `ORDER BY` it. *(Expected: same four numbers, now in funnel order.)*

3. **Step-over-top rate.** Add a column showing each step's users as a percentage of `site_visit`'s count, using `FIRST_VALUE(...) OVER (ORDER BY step_order)` as the denominator. *(Expected: 100.0, 80.0, 65.0, 50.0.)*

4. **Step-over-previous rate.** Add a second rate column using `LAG(...) OVER (ORDER BY step_order)` as the denominator instead. *(Expected: —, 80.0, 81.3, 76.9 — round to 1 decimal.)*

5. **Name the biggest drop-off.** Using Task 4's numbers, which single step-to-step transition loses the highest *percentage* of users? Which one loses the highest *absolute count*? Are they the same step? Write one sentence answering both, in a comment above your Task 5 query (you can reuse Task 4's query here).

6. **Per-user step timing.** For user 9 only, write a query using `LAG() OVER (PARTITION BY user_id ORDER BY event_ts)` that shows each step next to the step before it and the time elapsed. *(Expected: 4 rows — site_visit, signup, activated, paid_conversion — with `prev_step` NULL on the first row.)*

7. **The conversion window.** Using the pattern from Lecture 1 §5, write a query that checks, for every user who had a `site_visit`, whether they reached `paid_conversion` within **7 days** of that visit (not 14). *(Expected: still all 10 payers pass — every payer in this dataset converts in under 7 days. Confirm this by checking the *slowest* payer's elapsed time.)*

8. **The fan-out warning.** Write (but do not "fix") a query that joins the Task 1 step-count summary to the `touches` table on `user_id`, with no aggregation afterward, and run `SELECT step_name, COUNT(*) FROM (...) GROUP BY step_name`. Compare the counts to Task 1's. In a comment, explain in one sentence *why* they differ and which one (Task 1's or this one's) is correct for "how many users reached this step."

## Done when…

- [ ] Task 2's step_order column sorts site_visit → signup → activated → paid_conversion.
- [ ] Task 3 and Task 4's rate columns match the Expected values above (rounded to 1 decimal).
- [ ] Task 5 identifies the correct transition for both percentage and absolute-count framing, and states whether they're the same step.
- [ ] Task 6 shows `prev_step` as `NULL` on user 9's first row, not an error.
- [ ] Task 8's comment names the fan-out bug specifically — not just "the numbers are different."

## Stretch

- Rebuild the whole funnel using a **3-day** conversion window instead of 14. Does any payer drop out? (Compare each payer's `visit_ts` to `paid_ts`; find the slowest one first.)
- Compute the funnel **separately for users whose first touch was a paid channel** (`channel_type = 'paid'`) versus everyone else. Does paid traffic convert better or worse through the funnel than non-paid traffic, once it lands?
- `LAG` gives you the previous row. Rewrite Task 6 using `LEAD` instead so each row shows the *next* step and the time until it, and confirm the two approaches produce the same elapsed times shifted by one row.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-02/exercise-01/`.
