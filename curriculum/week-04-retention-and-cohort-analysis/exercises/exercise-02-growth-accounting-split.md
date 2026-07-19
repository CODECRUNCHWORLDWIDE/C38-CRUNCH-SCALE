# Exercise 2 — Compute a Growth-Accounting Split

**Goal:** Build the new/retained/resurrected/churned classification from scratch, month by month, and prove your numbers satisfy the accounting identity from Lecture 2.

**Estimated time:** 90 minutes. This is the week's hardest query — budget real time for it.

## Setup

```sql
SELECT COUNT(*) FROM users;    -- 48
SELECT COUNT(*) FROM events;   -- 366
```

Create `solutions.sql`. This exercise builds up in layers — do them in order; each task's output feeds the next.

## Tasks

1. **Calendar spine.** Build a `spine` of every `(user_id, month)` pair from that user's `cohort_month` through `2025-06`, inclusive. *(Hint: SQLite — a `WITH RECURSIVE` CTE stepping one month at a time; Postgres — `generate_series(date '2025-02-01', date '2025-06-01', interval '1 month')`. Expected row count: 12 Feb-cohort users × 5 months + 12 Mar-cohort × 4 + 12 Apr-cohort × 3 + 12 May-cohort × 2 = 60+48+36+24 = **168 rows**.)*

2. **Monthly activity flag.** Build a `monthly_active` set of distinct `(user_id, month)` pairs from `events` (a user is "active" in a month if they have ≥1 event that month). Left-join this onto your Task 1 spine to produce an `active` (0/1) flag for every spine row. *(Expected: summing the `active` column across all 168 rows gives **88** — this should equal the sum of `active_total` across all five months in the Lecture 2 growth-accounting table: 12+14+19+24+19 = 88.)*

3. **Previous-month lookup.** Add two window-function columns to Task 2's result: `prev_active` (the `active` value from that user's immediately preceding spine row — use `LAG` partitioned by `user_id`, ordered by `month`) and `ever_active_before` (whether `active` was ever 1 in any *earlier* spine row for that user — use a windowed `MAX` with a frame ending one row before the current one).

4. **Classify.** Using a `CASE` expression over Task 3's columns, label every row as exactly one of `'new'`, `'retained'`, `'resurrected'`, `'churned'`, or (for rows where none apply — inactive users with no prior activity) `'inactive_no_history'`. Follow the rules from Lecture 2 §5 precisely:
   - `new`: this is the user's `cohort_month` and they were active.
   - `retained`: active this month AND active the previous month.
   - `resurrected`: active this month, NOT active the previous month, but active at some earlier point.
   - `churned`: NOT active this month, but WAS active the previous month.

5. **Aggregate by month.** Group Task 4's output by `month` and count each status, plus `SUM(active)` as `active_total`. *(Expected: matches the Lecture 2 table exactly.)*

6. **Prove the identity.** Write one query that checks, for every month, whether `active_total = new_users + retained + resurrected`. Return any month where this is **false** (there should be none — an empty result set is success).

## Expected result (spot checks)

| month | new | retained | resurrected | churned | active_total |
|---|---:|---:|---:|---:|---:|
| 2025-02 | 12 | 0 | 0 | 0 | 12 |
| 2025-03 | 12 | 2 | 0 | 10 | 14 |
| 2025-04 | 12 | 6 | 1 | 8 | 19 |
| 2025-05 | 12 | 10 | 2 | 9 | 24 |
| 2025-06 | 0 | 14 | 5 | 10 | 19 |

## Done when…

- [ ] `solutions.sql` has all 6 queries, each under a `-- Task N` comment.
- [ ] Task 5's output matches the table above exactly.
- [ ] Task 6 returns **zero rows**.
- [ ] You can explain, from memory, why `churned(month)` is checked against `active_total` of the *previous* month, not the current one.
- [ ] You can point to the specific user and month that makes June's `resurrected = 5` — pick any one and trace their activity history by hand from `events` to confirm.

## Stretch

- Add a `churn_rate` column: `churned(month) / active_total(month − 1)`, expressed as a percentage. Which month had the worst churn rate? Which had the best?
- Re-run the whole classification with the spine built at **week** granularity instead of month. Does the story change, or does it just get noisier with the same shape?

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-04/exercise-02/`.
