# Week 2 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with hands-on queries, a written explanation, and a build-your-own-scenario task. Commit each.

All queries run against the seed tables from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Funnel variants (60 min)

The week's funnel used `site_visit → signup → activated → paid_conversion`. Practice re-slicing it.

1. Rebuild the full funnel (Exercise 1's query), but filter to only users whose **first touch** was a `paid` channel type. Report the same four step counts and step-over-top rates.
2. Rebuild it again for users whose first touch was **not** paid (organic, owned, or referral). Compare the two funnels' `signup`-stage conversion rate. Which group converts from visit to signup at a higher rate?
3. In `funnel-variants.md`, write 150 words on what this comparison would (and wouldn't) tell a growth team about whether to invest more in paid acquisition, given this is only a 10-day, 20-user sample.

---

## Problem 2 — Twenty attribution warm-ups (75 min)

Write and run each against `touches`/`conversions`/`channels`. Put them in `warmups.sql` with a `-- N` comment and the result beneath each.

1. The channel with the single largest first-touch revenue credit.
2. The channel with the single largest last-touch revenue credit.
3. Every user with exactly 1 touch before converting (single-touch converters).
4. Every user with 3 or more touches before converting.
5. The average number of touches per converting user (`AVG` over `COUNT(*) GROUP BY user_id`, restricted to converters).
6. The channel that appears as a **first** touch for the most *total* users (converted or not).
7. The channel that never appears as anyone's first touch.
8. For user 18, list all three touches with their `ROW_NUMBER()` position.
9. The total revenue from users whose path included **Email** anywhere (not just last-touch).
10. The total revenue from users whose path did **not** include Email anywhere.
11. Using first-touch, the two channels tied or closest in credited revenue.
12. The `channel_type` (paid/organic/owned/referral) that earns the most total revenue under last-touch.
13. Every touch that happened on `2025-01-05` across all channels, with the user and channel name.
14. The user with the longest gap, in hours, between their first and second touch.
15. Every paid-channel touch (`channel_type = 'paid'`) that belongs to a user who never signed up.
16. The number of distinct channels touched across all 20 users combined (not per-user — the union).
17. Every user whose first and last touch are the **same** channel but who had more than 1 touch (i.e., they left and came back to the same channel).
18. Total ad spend on the single most expensive day across all channels combined.
19. The day with the highest **Meta Ads** spend, and that day's spend amount.
20. Every channel with total spend under $500.

---

## Problem 3 — Explain the reconciliation rule (45 min)

In `reconciliation-writeup.md`, answer in prose (400 words max):

1. Run first-touch and last-touch attribution (Exercise 2) and confirm both sum to $1,590.
2. Now write a query that attributes revenue by joining `conversions` to `touches` **without** any `ROW_NUMBER()` filter or split — just a plain join and `GROUP BY channel_name`. Sum its `revenue_credited` column.
3. Explain, in your own words, why a "real" attribution model (first/last/linear/position-based) is *mathematically guaranteed* to reconcile to total revenue, while a plain join is not. What property does the query need to have to guarantee reconciliation?
4. Name one situation where you would deliberately choose **not** to reconcile a report to total revenue (hint: Challenge 2's Fix A), and explain why that's acceptable there but wouldn't be for a board-deck revenue number.

---

## Problem 4 — CAC under a stricter attribution rule (45 min)

The week used first-touch and last-touch CAC. Try a third:

1. Compute CAC for each paid channel using **linear** attribution instead — i.e., a channel's "conversions" for CAC purposes is the sum of its fractional linear credit across all converters (not a whole-number count), divided by spend inverted (spend ÷ fractional credit, in revenue-equivalent terms — or, simpler, treat linear *revenue* credited to the channel divided by spend as a "revenue-per-dollar" ratio instead of a CAC in dollars-per-customer).
2. Compare that ratio across the four paid channels. Does the ranking match first-touch CAC, last-touch CAC, neither, or something in between?
3. **Deliver** `linear-cac.sql` plus a two-sentence note on whether you'd trust a linear-based efficiency ratio more or less than first/last-touch CAC for a budget decision, and why.

---

## Problem 5 — Extend the dataset (60 min)

Make the data your own and query it.

1. Write `INSERT` statements adding **3 new users** (21, 22, 23) with their own `touches` and `funnel_events` rows. Include: one user with a **4-touch** path across four different channels who converts, one user who bounces after `site_visit` on a channel of your choice, and one user with a 2-touch path who signs up and activates but never pays.
2. Add a matching row to `conversions` for the one who pays (pick a plan and revenue).
3. Re-run the full funnel query and the first-touch/last-touch attribution queries. Report the new step counts and confirm attribution still reconciles to the new total revenue.
4. Then **undo** it cleanly: delete your new rows from `conversions`, `funnel_events`, `touches`, and `users` (in that order, respecting the foreign keys), and confirm you're back to the original 20 users / $1,590.

**Deliver** `extend.sql` (inserts, the two re-run queries with output, and the cleanup) plus one sentence on how much a single 4-touch user changed the attribution comparison table.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.75 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
