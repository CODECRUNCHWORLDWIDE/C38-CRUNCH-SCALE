# Exercise 2 — Compute a Time-to-Value Distribution

**Goal:** Compute the full time-to-value (TTV) distribution — not just an average — for Crunch Boards' activation event, on your engine of choice, and describe its shape.

**Estimated time:** 60 minutes.

## Setup

Connected, seed loaded. New tasks in `solutions.sql` under `-- Task N` comments, plus a short `ttv-report.md`.

## Tasks

1. **Raw TTV, per user.** Write a query returning `user_id` and `ttv_hours` (hours between `signup_at` and their `invited_teammate` event) for every user who reached it. Order by `ttv_hours` ascending. *(Expected: 116 rows, minimum ≈ 21.1 hours, maximum ≈ 185.9 hours.)*

2. **Summary stats.**
   - **PostgreSQL:** use `PERCENTILE_CONT` to compute p25, median (p50), and p75, plus `MIN`, `MAX`, `AVG`.
   - **SQLite:** compute the same five numbers using the sorted-rows-plus-`OFFSET` technique from Lecture 3 §4.
   *(Expected: min ≈ 21.1h, p25 ≈ 80.0h, median ≈ 112.3h, p75 ≈ 133.0h, max ≈ 185.9h, mean ≈ 108.1h.)*

3. **Bucket the distribution.** Using a `CASE WHEN` on `ttv_hours / 24.0` (days), bucket every inviter into `<24h`, `1-3d`, `3-7d`, `>7d`, and count each bucket. *(Expected: 2, 17, 88, 9.)*

4. **Compare mean vs. median.** State both numbers side by side and explain in one sentence why they're close here but wouldn't necessarily be for a more right-skewed distribution (e.g. one with a long tail of very slow adopters).

5. **Step-specific TTV.** Compute the *gap* (not from signup, but from `created_first_card` to `invited_teammate`) for the same 116 users. *(Expected median ≈ 50 hours, ≈ 2.1 days.)* Compare this to the total TTV median from Task 2 — what fraction of the total time-to-value happens *before* the user even creates their first card, versus in the card→invite gap itself?

## Done when…

- [ ] Task 1 returns exactly 116 rows.
- [ ] Task 2's five summary numbers are within rounding of the "Expected" hints, computed correctly for your engine (Postgres `PERCENTILE_CONT` or SQLite's manual method — both attempted if you have both engines installed).
- [ ] Task 3's four bucket counts sum to 116.
- [ ] `ttv-report.md` states the median, the mean, and one sentence on what the shape (mostly clustered in 3–7 days, a long right tail) implies for when a product team should time an onboarding nudge.

## Stretch

- Split the TTV distribution by `plan` (`Free` vs `Pro`). Do paying-tier users invite teammates faster? Compute both medians and say which SQL construct you used to do it in one query (`GROUP BY` + `PERCENTILE_CONT`, or two separate queries).
- Recompute Task 1's TTV using `created_first_board` (instead of signup) as the start time. How much of the total TTV happens before the user even makes their first board?

## Submission

Commit `solutions.sql` and `ttv-report.md` to your portfolio under `c38-week-03/exercise-02/`.
