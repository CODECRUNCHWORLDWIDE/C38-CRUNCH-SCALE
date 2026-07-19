# Exercise 3 — Query a Metric from Raw Events

**Goal:** Turn every headline number from the lectures into your own SQL, typed from scratch, against the live `users`/`events` seed data. By the end, "compute a metric" stops meaning "look at a dashboard" and starts meaning "write eight lines of SQL."

**Estimated time:** 90 minutes.

## Setup

Confirm the seed is loaded (see the [week README](../README.md)):

```sql
SELECT COUNT(*) FROM users;    -- must print 15
SELECT COUNT(*) FROM events;   -- must print 149
```

Remember: **"today" is 2026-06-21** everywhere this week — that's the last day the seed data covers. Create `exercise-03.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Signups by channel.** Count signups grouped by `signup_channel`, sorted with the highest first. *(Expected: 5 rows, 3 signups each — the seed is deliberately balanced.)*

2. **Overall activation rate.** What percentage of all 15 users have at least one `habit_created` event? Report both the raw count and the percentage. *(Expected: 12 activated, 80.0%.)*

3. **This week's engaged users (the north star).** Count distinct users with **3 or more** `checkin_logged` events in the trailing 7 days ending 2026-06-21 (i.e., `event_time` from `2026-06-15 00:00:00` up to but not including `2026-06-22 00:00:00`). *(Expected: 10. This is StreakLab's Weekly Engaged Users, exactly as defined in Lecture 1.)*

4. **Subscribers and new MRR.** Count `subscription_started` events and sum their `event_value`. *(Expected: 6 subscribers, $54.00 total.)*

5. **Referral funnel.** Count `invite_sent` and `invite_accepted` events separately (two numbers from one query, using two `FILTER`s or two subqueries — your choice). *(Expected: 3 sent, 3 accepted.)*

6. **Check-in intensity among activated users.** Compute the average number of `checkin_logged` events **per activated user** (activated users only — exclude the 3 who never created a habit). *(Expected: 110 total check-ins ÷ 12 activated users ≈ 9.17.)*

7. **One row, fully joined.** For every user, produce one row with: `user_id`, `signup_channel`, `activated` (boolean), `checkins_last_7d` (integer), `is_weekly_engaged` (boolean), `is_subscriber` (boolean). This is the per-user metric-tree table from Lecture 3, section 4 — write it yourself rather than copying it verbatim. *(Expected: 15 rows; spot-check user 4 — should show `activated = false`, `checkins_last_7d = 0`, `is_weekly_engaged = false`, `is_subscriber = false`.)*

## Expected result (spot checks)

- Task 1 → 5 channels, 3 each.
- Task 2 → 12 / 80.0%.
- Task 3 → 10.
- Task 4 → 6 subscribers, $54.00.
- Task 5 → 3 and 3.
- Task 6 → ≈9.17.
- Task 7 → user 4's row matches the description above exactly.

## Done when…

- [ ] `exercise-03.sql` has all 7 queries under `-- Task N` comments.
- [ ] Every result matches the spot checks above — if one doesn't, trace the disagreement to a specific row before moving on (Lecture 3, section 5).
- [ ] Task 7's query runs as a single `SELECT` (CTEs are fine; a chain of separate ungrouped queries is not).
- [ ] You did not open a spreadsheet at any point.

## Stretch

- Add a **time-to-activate** column to Task 7's query: the number of hours between `signup_at` and the first `habit_created` event, `NULL` if never activated. *(Hint: `EXTRACT(EPOCH FROM (activated_at - signup_at)) / 3600.0`.)* Which archetype activates fastest, and which is slowest?
- Rerun Task 3 as of **2026-06-14** instead of 2026-06-21 (the prior week). Compare the two counts — is StreakLab's north star trending up or down week over week? You now have a two-point trend line, computed entirely from raw events.

## Submission

Commit `exercise-03.sql` to your portfolio under `c38-week-01/exercise-03/`.
