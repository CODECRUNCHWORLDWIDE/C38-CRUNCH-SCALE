# Exercise 2 — Model an MRR Fact Table

**Goal:** Build the conformed `dim_date` and `dim_user` dimensions and the `fct_mrr_monthly` fact table from Lecture 2, then read a real six-month revenue trend out of it.

**Estimated time:** 75 minutes.

## Setup

Exercise 1's four `stg_*` objects exist and pass their checks. New tasks continue in `solutions.sql`.

## Tasks

1. **`dim_date`.** Build it exactly as in Lecture 2, Section 3 — a monthly spine from `2025-01-01` through `2026-12-01`. `SELECT COUNT(*) FROM dim_date;` *(Expected: **24** rows.)*

2. **`dim_user`.** Build it as in Lecture 2, Section 2, off `stg_users`. `SELECT COUNT(*) FROM dim_user;` *(Expected: **15** rows.)*

3. **`fct_mrr_monthly`.** Build it as in Lecture 2, Section 5, restricted to `2025-01-01` through `2025-06-01` (six months). `SELECT COUNT(*) FROM fct_mrr_monthly;` *(Expected: **90** rows — 6 months × 15 users, including the many `$0` rows for customers not yet signed up or already churned in a given month. If you get fewer than 90, your `CROSS JOIN` or date filter dropped rows it shouldn't have.)*

4. **Monthly trend.** Run the query from Lecture 2, Section 6 (`GROUP BY month_date`). Compare your output to the table below — it must match **exactly**, cent for cent:

   | month_date | mrr_usd | active_customers |
   |---|---:|---:|
   | 2025-01-01 | 157.00 | 3 |
   | 2025-02-01 | 555.00 | 5 |
   | 2025-03-01 | 1211.00 | 9 |
   | 2025-04-01 | 1539.00 | 11 |
   | 2025-05-01 | 1858.17 | 12 |
   | 2025-06-01 | 1858.17 | 12 |

   If any row doesn't match, the bug is almost always in the correlated subquery's date-range condition (`started_on <= month_end AND (canceled_at IS NULL OR canceled_at > month_end)`) — re-derive one mismatched user's months by hand against `stg_subscriptions` before touching the query again.

5. **Explain the June plateau.** MRR is flat from May to June even though the company presumably kept signing up customers. Looking at `raw_users.signup_date`, explain in a one-sentence comment why June shows no growth in *this specific dataset* — is it a bug, or a property of the data you seeded?

6. **Net-new MRR by month.** Using `fct_mrr_monthly` and a window function (`LAG`), compute month-over-month **net-new MRR** (this month's total minus last month's total) for each of the 6 months. *(Expected deltas: Jan → n/a, Feb +398.00, Mar +656.00, Apr +328.00, May +319.17, Jun +0.00.)*

7. **The customer who vanishes.** Query `fct_mrr_monthly` for user 5 (Elin) across all six months. Every row shows `mrr_cents = 0`, even though she was a real, paying customer for two weeks in February. In a one-sentence comment, explain *why* — tie your answer to the exact metric definition from Lecture 3, Section 3 ("as of the last day of the given month"). This is not a bug in your SQL; it's a real, known limitation of month-end MRR snapshots.

## Done when…

- [ ] `dim_date`, `dim_user`, and `fct_mrr_monthly` all exist and match the expected row counts.
- [ ] Task 4's six-row table matches exactly, including cents.
- [ ] Task 6's net-new deltas match.
- [ ] Tasks 5 and 7 each have a one-sentence written explanation — not just a query.

## Stretch

- Rebuild `fct_mrr_monthly` for the full 24-month `dim_date` spine (not just Jan–Jun 2025) and confirm every month from July 2025 onward shows the same totals as June (since the seed data has no activity past May) — a good sanity check that your date-range logic degrades correctly into the future.
- Add an `mrr_band` column to a query over `fct_mrr_monthly` (`'0-2900'`, `'2901-9900'`, `'9901+'`, however you want to bucket it) and count active customers per band per month — a first taste of the segmentation work coming in Week 7.
- Compute **ARR** (Annual Recurring Revenue = MRR × 12) for the May snapshot and sanity-check it against `SUM` of each active customer's *actual* annualized contract value from `stg_subscriptions` (remembering Opal's contract genuinely *is* annual). Do the two methods agree, and should they?

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-06/exercise-02/`.
