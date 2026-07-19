# Week 6 — Homework

Five problems, ~4.5 hours total, spread across the week. These extend the mini-project's warehouse with two metrics every RevOps team eventually needs (ARR, logo churn), stress-test the idempotency claim, and make you invent — then catch — your own reconciliation bug.

All queries run against the `crunch_flow` seed database and the `stg_*`/`dim_*`/`fct_*` objects you built in the exercises, unless a problem says to add something new.

---

## Problem 1 — ARR, three ways (45 min)

"What's our ARR?" sounds like it should just be `MRR × 12`, and mostly it is — but there's more than one defensible way to compute it, and a RevOps analyst should know all three exist.

1. **Method A — simple annualization.** `SUM(mrr_cents) * 12` from the latest month of `fct_mrr_monthly`.
2. **Method B — true contract value.** Sum each active customer's *actual* contract terms from `stg_subscriptions` directly (a monthly customer's contribution is `mrr_cents * 12`; an annual customer's contribution is their real annual `amount_cents`, not the normalized-then-reinflated figure).
3. **Method C — logo-weighted.** `dim_plan`'s list price × active customer count per plan (you'll need to build a tiny 3-row `dim_plan` table for this one: Starter $29, Growth $99, Scale $299 monthly-equivalent).

Compute all three for the May 2025 snapshot. **Deliver** `arr.sql` (the three queries) and a two-paragraph `arr-writeup.md`: do all three agree exactly? If not, explain the cent-level discrepancy — it should trace back to Opal's annual plan and `ROUND()` behavior. Which method would you recommend as the warehouse's official ARR definition, and why?

---

## Problem 2 — Logo churn, month over month (75 min)

**Logo churn** (as opposed to revenue churn) counts *customers* lost, regardless of their plan size: `customers active at the start of a period who are inactive by the end of it, divided by customers active at the start`.

1. Using `fct_mrr_monthly` and a window function (`LAG` over `user_id` ordered by `month_date`), build a query that flags, for each user and month, whether they were active last month but are **not** active this month (a churn event).
2. Aggregate into a monthly logo-churn-rate table for Feb–Jun 2025 (Jan has no "prior month" in this dataset, so it's excluded).
3. Compare your output to the table below — it must match exactly:

   | month | churned_logos | active_start_of_month | logo_churn_rate |
   |---|---:|---:|---:|
   | Feb | 0 | 3 | 0.0% |
   | Mar | 0 | 5 | 0.0% |
   | Apr | 1 | 9 | 11.11% |
   | May | 1 | 11 | 9.09% |
   | Jun | 0 | 12 | 0.0% |

4. Name the two customers who generated the only two churn events in this six-month window, and cross-check each against `raw_events` — does an explicit `canceled` event exist for both, on a date consistent with your SQL result?

**Deliver** `logo-churn.sql` and a `logo-churn.md` explaining, in one paragraph, why logo churn and *revenue* churn (dollars lost, not customers lost) can disagree — give a hypothetical (not from this dataset) where a company could have healthy logo churn but terrible revenue churn, or vice versa.

---

## Problem 3 — Invent, then catch, a fourth discrepancy (60 min)

Exercise 3 found three real Stripe-vs-app-database discrepancies already baked into the seed data. This problem asks you to add a **fourth, of your own design**, and prove your Exercise 3/Challenge 2 tooling catches it without modification.

1. Pick a new fictional Crunch Flow customer (`user_id = 16`), and `INSERT` a `raw_users` row, a `raw_stripe_subscriptions` row, and a `raw_app_subscriptions` row for them — but make the two subscription rows **disagree** in some way not already covered by Ines, Kira, or Mira (e.g., a currency mismatch, a proration-related price difference, a plan that's active in the app DB but was never created in Stripe at all — genuinely invent one).
2. Re-run your staging views (they're views, so this is automatic) and re-run the reconciliation queries from Exercise 3 and/or the test suite from Challenge 2.
3. Confirm your new discrepancy is caught **without editing any of your existing SQL** — if it isn't, that's a real gap in your reconciliation logic, and you should note what you'd add to close it.

**Deliver** `invented-discrepancy.sql` (the inserts) and a short `invented-discrepancy.md`: what you invented, why it's realistic, and whether your existing tooling caught it on the first try or needed a fix.

---

## Problem 4 — Prove idempotency under stress (45 min)

Lecture 3 claims truncate-and-reload is idempotent. Prove it under conditions closer to a real accidental rerun.

1. Run your `03_marts.sql` (or equivalent) build script **three times in a row**. Confirm `SELECT COUNT(*) FROM fct_mrr_monthly;` is 90 every time (or 90 + however many rows Problem 3's new customer added, if you did that problem first).
2. Now simulate a late-arriving duplicate webhook: `INSERT` a byte-for-byte duplicate of an existing `raw_stripe_subscriptions` row (same `subscription_id` is disallowed by the primary key — instead duplicate the *content* under a new `subscription_id`, exactly like Stripe accidentally double-firing a webhook with two different event IDs for the same underlying charge would look). Rebuild the marts.
3. Does `fct_mrr_monthly`'s total MRR silently double for that customer? If so, **that's the real-world scenario Challenge 2's uniqueness/reconciliation tests exist to catch** — run your test suite and confirm it does. If your test suite doesn't catch it, explain in writing what test you'd add.

**Deliver** `idempotency-stress.sql` and a short `idempotency-stress.md` write-up of what happened at each step.

---

## Problem 5 — Write the missing metric definitions (45 min)

`metrics.md` from the mini-project covered MRR, ARR, and active customer. This problem extends it.

Write full entries, in the exact format from Lecture 3, Section 3, for:

1. **Logo churn rate** (from Problem 2) — definition, computed-in table, source of truth, rules, owner.
2. **Net-new MRR** (from Exercise 2, Task 6) — same format.
3. One metric of your own choosing that a later week of this course will need (pick from: activation rate, LTV, CAC payback, experiment lift) — you don't need to be able to compute it yet with this week's tables; the exercise is writing a precise definition *before* the data exists, which is exactly what a RevOps lead does when a new metric gets requested.

**Deliver**: append all three to your `metrics.md` from the mini-project.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 75 min |
| 3 | 60 min |
| 4 | 45 min |
| 5 | 45 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md) if you haven't already.
