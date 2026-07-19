# Exercise 3 — Reconcile Revenue Across Two Sources

**Goal:** Find, quantify, and document the three real discrepancies between Stripe (`stg_subscriptions`) and the internal app database (`stg_app_subscriptions`) — the exact task a RevOps analyst does every time Finance and Product disagree.

**Estimated time:** 75 minutes.

## Setup

Exercises 1 and 2 complete: `stg_subscriptions` (18 rows), `stg_app_subscriptions` (15 rows), `fct_mrr_monthly` all built and correct. This exercise compares **current state** — "what is each system telling us *right now*" — so first build a current-only view of the Stripe side:

```sql
CREATE VIEW stg_subscriptions_current AS
SELECT *
FROM stg_subscriptions
WHERE status = 'active';
```

*(Expected: 12 rows — the 12 customers active as of the latest data, matching the May/June `active_customers` count from Exercise 2.)*

## Tasks

1. **Build the reconciliation query.** Join `stg_subscriptions_current` to `stg_app_subscriptions` on `user_id`, projecting both systems' `plan`, `mrr_cents`, and `status` side by side with clear aliases (`stripe_plan`, `app_plan`, `stripe_mrr_cents`, `app_mrr_cents`, `stripe_status`, `app_status`). *(Expected: 12 rows — one per currently-active Stripe customer, each with its app-DB counterpart alongside it.)*

2. **Flag every mismatch.** Add a boolean (or `CASE`-derived text) column that's `TRUE`/`'MISMATCH'` whenever `stripe_plan <> app_plan` **or** `stripe_mrr_cents <> app_mrr_cents`. Filter to only the mismatched rows. *(Expected: exactly **2** rows among the 12 active-in-both comparison — Ines (user 9) and Kira (user 11). Mira (user 13) won't show up here — see Task 3.)*

3. **Find the third discrepancy — the one Task 2 can't see.** Task 1's join used `stg_subscriptions_current`, which only has Stripe's *active* subscriptions. But the app database still thinks user 13 (Mira) is `active`. Write a query that finds users present in `stg_app_subscriptions` with `status = 'active'` but **absent** from `stg_subscriptions_current` (a `LEFT JOIN ... WHERE ... IS NULL`, or `NOT EXISTS`). *(Expected: **1** row — Mira.)*

4. **Name each discrepancy precisely.** For each of the three (Ines, Kira, Mira), write a one-line comment stating: which field disagrees, which system is stale/wrong, and your best guess at the real-world cause. Use the raw tables (`raw_stripe_subscriptions`, `raw_app_subscriptions`) to check your reasoning — don't guess blind.

   *(Expect your answers to land close to: **Ines** — the app DB still shows her old `Growth`/$99 plan; Stripe shows she downgraded to `Starter`/$29 on 2025-04-05. Cause: a downgrade webhook that never updated the internal record. **Kira** — the app DB has her `billing_cycle` wrong (`annual` instead of `monthly`), so her normalized `mrr_cents` comes out far too low ($24.92 vs. the real $299.00/mo). Cause: a data-entry error, not a sync failure — this one didn't come from Stripe at all. **Mira** — the app DB still shows `active`; Stripe shows she canceled on 2025-05-19. Cause: a cancellation webhook that never fired, or fired and failed silently.)*

5. **Pick, and justify, a winner.** For each of the three fields in conflict (plan+price for Ines, billing_cycle/price for Kira, status for Mira), state which system should win **and why**, citing Lecture 2 Section 4's design decision. *(Expected answer for all three: Stripe wins, because it is this warehouse's declared source of truth for money — the app database is useful for feature-gating, not billing. Write this down as if you were the RevOps analyst justifying the call in a Slack thread that Finance will read.)*

6. **Quantify the blast radius.** If nobody had caught these three discrepancies and the company had reported MRR from `stg_app_subscriptions` instead of `stg_subscriptions_current` for the current month, how far off would the reported number have been? Compute both totals and the dollar (and percentage) difference. *(Work it out precisely from the numbers in Task 2/3 — don't estimate. Expected: the app database reports **$1,683.09 across 13 "active" customers**; Stripe reports the true **$1,858.17 across 12 active customers**. The app-side number understates true MRR by **$175.08 (≈9.4%)**, and it does so while showing a phantom *extra* active customer — Kira's understated $274.08 and Ines's overstated $70.00 and Mira's phantom $29.00 net out to −$175.08. Note how the customer-count error and the dollar error don't even point the same direction — exactly why you reconcile both, never just one.)*

## Done when…

- [ ] Task 2 finds exactly 2 mismatched rows via the join-based comparison.
- [ ] Task 3 finds the 1 additional discrepancy the join-based comparison structurally cannot see, and you can explain *why* it can't see it.
- [ ] All three discrepancies are named with field, direction, and likely cause (Task 4).
- [ ] Task 5's "Stripe wins" justification is written down, not just understood.
- [ ] Task 6's dollar and percentage blast-radius numbers are computed, not guessed.

## Stretch

- Write a single query that produces **all three** discrepancies in one result set (a `UNION` of the join-mismatch query and the missing-from-Stripe query), each row labeled with a `discrepancy_type` column (`'plan_or_price_mismatch'` / `'stale_status'`). This is the shape of a real reconciliation report you'd hand to Finance.
- Given that these three discrepancies exist *today*, what would you add to this week's data-test suite (Lecture 3, Section 2) to catch the *next* one automatically, before a human has to go looking? Sketch the test query — you'll build it for real in Challenge 2.
- The reconciliation above only checked *currently active* subscriptions. Is it possible for a **canceled** subscription to also disagree between systems in a way that matters (e.g., disagreement on *when* it was canceled, affecting last month's MRR)? Write one query that checks agreement on `end_date`/`canceled_at` between the two systems for canceled subscriptions.

## Submission

Commit `solutions.sql` (with all six tasks' written comments included) to your portfolio under `c38-week-06/exercise-03/`.
