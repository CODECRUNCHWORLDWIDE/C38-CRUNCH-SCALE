# Exercise 2 — Build the Core Metric Suite

**Goal:** Build `fct_acquisition`, `fct_mrr_monthly`, and `fct_unit_economics`, then pull a full retention curve out of the warehouse — the numbers Lecture 2 walked through, reproduced by your own hand.

**Estimated time:** 90 minutes.

## Setup

Exercise 1 complete: `stg_*` views, `dim_date`, and `dim_user` all built and verified. Continuing in `solutions.sql`.

## Tasks

1. **`fct_acquisition`.** Build it as in Lecture 2, Section 1 — one row per user, `activated`/`converted` flags from `stg_events`/`stg_subscriptions`. *(Expected: 72 rows. Roll up by channel and confirm you land exactly on `paid_search: 30/20/11`, `organic: 24/21/12`, `referral: 18/15/12` for signups/activated/converted.)*

2. **`fct_mrr_monthly`.** Build it with the corrected month-end join from Lecture 2, Section 2 (both the `created <=` and `canceled_at IS NULL OR canceled_at >` conditions). *(Expected: blended June MRR = **$1,878.00** across **22** active customers. If you get a bigger number, you forgot the cancellation condition and are counting churned customers as still paying.)*

3. **`fct_unit_economics` — CAC only (defer LTV to Task 5).** Build the `spend`/`paying` CTEs and the CAC column from Lecture 2, Section 3. *(Expected: `paid_search` CAC = **$1,963.64**, `organic` = **$1,200.00**, `referral` = **$225.00**.)*

4. **Full retention curve, pooled and per channel.** This is new — Lecture 2 gave you the finished numbers but not the query. Build it yourself, following the age-since-`created` pattern from Week 5's cohort-curve lecture, adapted to `stg_subscriptions`:

   ```sql
   -- starter shape for PostgreSQL — adapt the generate_series/interval math for SQLite
   -- with a recursive CTE if that's your engine, per Week 5 Lecture 1's note
   WITH cohort_ages AS (
       SELECT
           s.subscription_id,
           u.signup_channel,
           s.created,
           s.canceled_at,
           gs AS age
       FROM stg_subscriptions s
       JOIN dim_user u ON u.user_id = s.user_id,
            generate_series(0, 5) AS gs
       WHERE s.created + (gs || ' months')::interval <= DATE '2025-06-30'
   )
   -- TODO: flag is_active per row at each age, then aggregate by channel + age
   ```

   *(Expected pooled curve: age 0 → 100.0% (n=30), age 1 → 88.0% (n=25), age 2 → 71.4% (n=21), age 3 → 70.6% (n=17), age 4 → 58.3% (n=12), age 5 → 50.0% (n=4). Per channel, confirm `paid_search` drops to **44.4%** by age 2 — the leaky bucket Lecture 2 named — while `referral` stays at **100%** through every age it has data for.)*

5. **LTV — cohort sum, ages 0–4.** Using the retention curve from Task 4 and a **75% gross margin**, compute contribution-margin LTV per channel in pandas (or SQL if you prefer window functions), following Lecture 2, Section 3's method exactly. *(Expected: `paid_search` = **$165.93**, `organic` = **$206.43**, `referral` = **$555.60**. Add an `ltv_cohort_sum` and `ltv_to_cac` column to `fct_unit_economics`.)*

6. **Name the small-sample trap.** Compute the reciprocal-formula LTV (`margin ÷ avg monthly churn`, ages 1–4) for all three channels. One of the three should be undefined or absurd. In a comment, name which channel, why (cite the exact `n` at the relevant age), and which LTV number (`ltv_cohort_sum` or the reciprocal) belongs in `metrics.md`.

## Done when…

- [ ] `fct_acquisition`, `fct_mrr_monthly`, and `fct_unit_economics` all exist as real tables with the exact numbers above.
- [ ] Your retention-curve query reproduces Lecture 2's pooled and per-channel numbers without copy-pasting the summarized values from the lecture.
- [ ] `fct_unit_economics` has both `cac` and `ltv_cohort_sum` columns, plus a computed `ltv_to_cac` ratio.
- [ ] You can state, in one sentence, which channel's reciprocal LTV is untrustworthy and why — with the `n` to back it up.

## Stretch

- Add a `payback_months_naive` column (`cac / contribution_margin_per_month`) to `fct_unit_economics` and confirm `referral`'s naive payback (~2 months) is dramatically faster than `paid_search`'s (~39 months) — then explain in one sentence why "naive" is doing a lot of work in that column name (Week 5, Lecture 2, Section 3).
- Plot the three channels' retention curves on one chart (any plotting library) and visually confirm what the numbers already told you: `paid_search` bends down hard by age 2, `organic` bends down later and more gently, `referral`'s curve is a flat line with a warning label ("small sample") attached.

## Submission

Commit `solutions.sql` (and any plotting script) to your portfolio under `c38-week-12/exercise-02/`.
