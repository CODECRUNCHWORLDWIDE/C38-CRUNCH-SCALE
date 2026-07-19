# Exercise 1 — Build a Cohort Retention Triangle

**Goal:** Go from raw `users` + `events` to a full, correctly-shaped retention triangle, entirely from your own query — no peeking at Lecture 1's finished SQL until you've made a real attempt.

**Estimated time:** 75 minutes.

## Setup

Confirm you're connected and both tables loaded:

```sql
SELECT COUNT(*) FROM users;    -- must print 48
SELECT COUNT(*) FROM events;   -- must print 366
```

Create a file `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Months-since-signup for one user.** For `user_id = 24`, list every event with its `event_date` and a computed `month_number` (calendar months elapsed since that user's `signup_date`; signup month = 0). *(Expected: their first event is month 0, and their events span months 0 through 4 — check against the seed data's Mar 2025 cohort.)*

2. **Cohort sizes.** Write a query that returns each `cohort_month` and how many users are in it. *(Expected: all four cohorts have exactly 12 users.)*

3. **Active users per (cohort, month_number).** Extend Task 1's per-event `month_number` logic to *all* users, then count **distinct** active users for every `(cohort_month, month_number)` pair that actually has at least one event. *(Expected: 14 populated pairs total across the four cohorts — Feb has 5, Mar has 4, Apr has 3, May has 2.)*

4. **The full triangle.** Combine Tasks 2 and 3 into one query that returns `cohort_month`, `cohort_size`, `month_number`, `active_users`, and `pct_retained` (active_users as a percentage of cohort_size, rounded to 1 decimal). *(Expected: matches the table in Lecture 1 exactly — Feb month-1 is 16.7%, May month-1 is 66.7%.)*

5. **Isolate one column.** From Task 4, filter to `month_number = 1` only, across all four cohorts, ordered by `cohort_month`. *(Expected: 16.7%, 41.7%, 50.0%, 66.7% — a clean upward trend.)*

6. **Isolate one row.** From Task 4, filter to `cohort_month = '2025-02'` only, ordered by `month_number`. This is the single curve you'd hand to a stakeholder asking "how does the February cohort behave over time?" *(Expected: 100.0%, 16.7%, 16.7%, 8.3%, 25.0%.)*

## Expected result (spot checks)

- Task 2 → every cohort_month shows `12`.
- Task 4 → 14 rows total (the populated triangle cells), plus the four `month_number = 0` rows at 100.0% (so 18 rows if you count month 0).
- Task 5 → four rows, percentages strictly increasing: 16.7 < 41.7 < 50.0 < 66.7.
- Task 6 → five rows (month 0 through month 4), since Feb is the only cohort with a full month-4 column.

## Done when…

- [ ] `solutions.sql` has all 6 queries, each under a `-- Task N` comment.
- [ ] Task 4's triangle matches the Lecture 1 table exactly, to one decimal place.
- [ ] You can explain out loud why the May cohort's rows in Task 4 stop at `month_number = 1`.
- [ ] You can explain why Task 5's numbers, read left to right, are meaningful evidence of something changing — and Task 6's numbers, read left to right, are not comparable to a *different* cohort's row at the same positions without also checking cohort size.

## Stretch

- Rewrite Task 4 so it also reports an `eligible` flag per cell — `TRUE` if that cohort has had enough time to reach that `month_number` as of `2025-06-30`, `FALSE` otherwise (there should be no `FALSE` rows in your output, because Task 3's `JOIN` already only returns cells with data — the point of this stretch is to notice that Task 4 alone can't tell you "no signal" apart from "not old enough yet" unless you add this check explicitly).
- Using the same technique, build the triangle keyed on **week-of-signup** instead of month-of-signup. You'll need a different date-truncation expression and the triangle will have many more, much thinner rows — a good gut check for why most B2B products report monthly, not weekly, cohorts.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-04/exercise-01/`.
