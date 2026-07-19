# Week 4 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on queries, a written explanation, and a build-your-own-cohort task. Commit each.

All queries run against the `users`/`events` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Cross-engine portability check (45 min)

Whichever engine you used for the mini-project, rebuild **Panel A** (the cohort triangle) and **Panel B** (growth accounting) on the *other* one.

1. If you built the mini-project on PostgreSQL, rebuild both panels on SQLite (or vice versa).
2. The only lines that should need to change are the date-arithmetic expressions (`strftime` vs. `DATE_PART`/subtraction) and, for growth accounting, the calendar-spine construction (`WITH RECURSIVE` vs. `generate_series`).

**Deliver** `portability.sql` (both panels, both engines) and `portability.md` — two or three sentences on which engine's date syntax you found more natural, and one concrete gotcha you hit switching between them.

---

## Problem 2 — Twenty warm-up queries (75 min)

Write and run each against the seed data. Put them in `warmups.sql` with a `-- N` comment and the result beneath each.

1. How many users are in each `segment`?
2. How many users are in each `channel`?
3. The three users with the most total events (any date range).
4. Every event for `user_id = 34`, ordered by date.
5. The single earliest `event_date` in the whole table, and which user it belongs to.
6. The single latest `event_date` in the whole table, and which user it belongs to.
7. Count of events per calendar month (`GROUP BY strftime('%Y-%m', event_date)` or the Postgres equivalent).
8. Users who have **zero** events after their signup month (i.e., never appear in `events` with `month_number >= 1`).
9. The `sales_assisted` user with the most total events.
10. Users whose most recent event is in June 2025 (i.e., still active as of the latest data).
11. For each `channel`, the count of distinct users and the count of total events.
12. The cohort (`cohort_month`) with the highest total event count.
13. Users active in **both** February and June 2025 (their first and the last observed month).
14. The average number of distinct active days per user, across all users (including users with zero events, counted as 0).
15. Users in the `2025-04` cohort who were active in month 1 but **not** month 2.
16. The `self_serve` user with the longest gap (in days) between two consecutive events.
17. Count of events per `country`.
18. Every user who has an event on a weekend (`strftime('%w', event_date)` — 0 or 6 — or `EXTRACT(DOW FROM event_date)` in Postgres).
19. The three cohort/segment combinations (of the 8 total) with the fewest total events.
20. A single query returning, per `cohort_month`, the count of users and the count of *distinct users who churned at least once* (had an active month followed by an inactive one, per the Exercise 2 definition) before June 2025.

---

## Problem 3 — Explain the definitions (45 min)

In `definitions-writeup.md`, answer in prose (no more than 450 words total):

1. In your own words, explain the difference between N-day, unbounded, and rolling retention — and give a one-sentence example question each one is the *right* tool for.
2. Explain right-censoring in your own words, using the May 2025 cohort as your example. What would go wrong if you computed "month-3 retention across all cohorts" by dividing by all 48 signups instead of only the eligible ones?
3. Explain, without SQL, how a "resurrected" user differs from a "retained" user, and why a growth team might care about the distinction when deciding where to spend a re-engagement budget.
4. In one paragraph, explain why stickiness (DAU/MAU) and retention/survival are measuring two different things, using the `self_serve` vs. `sales_assisted` gap from this week as your example.

---

## Problem 4 — Rolling retention at multiple windows (45 min)

The same idea, three window sizes. In `rolling.sql`:

1. Compute rolling **7-day**, **14-day**, and **28-day** active-user counts, each as of `2025-06-30` (i.e., "how many distinct users had an event in the trailing N days").
2. For each window, also compute the percentage of *eligible* users (signed up on or before `2025-06-30`) that represents.
3. Repeat all of Task 1–2 as of `2025-05-31` instead, so you have two snapshots of each window.

**Deliver** the six resulting numbers (3 windows × 2 snapshot dates) in a small markdown table, plus two sentences: does the 7-day window or the 28-day window swing more between the two snapshot dates, and why would you expect that going in?

---

## Problem 5 — Add a cohort, then undo it (60 min)

Make the data your own, temporarily.

1. Write `INSERT` statements adding a **fifth cohort**: 12 new users signed up in **June 2025** (`cohort_month = '2025-06'`), split 9 `self_serve` / 3 `sales_assisted` like the existing cohorts. Give them `user_id` values `49`–`60` and signup dates spread across June.
2. Write a handful of `events` rows for these new users (their month-0 activity only — they haven't had time to reach month 1 yet as of `2025-06-30`).
3. Load them (you now have 60 users).
4. Re-run **Panel A** (the triangle) and **Panel B** (growth accounting) from the mini-project. Confirm the June cohort appears as a new row/column in the triangle at `month_number = 0` only, and that June's `new_users` count in the growth-accounting table increases by 12.
5. Then **undo it cleanly**: `DELETE FROM events WHERE user_id >= 49; DELETE FROM users WHERE user_id >= 49;` and confirm you're back to 48 users / 366 events.

**Deliver** `extend.sql` (the inserts, the two re-run panels with output, and the cleanup) plus one sentence on how adding a 5th cohort changed — or didn't change — the earlier cohorts' numbers in the triangle. *(It shouldn't change them at all — say why that's true, and what it would mean if it had.)*

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
