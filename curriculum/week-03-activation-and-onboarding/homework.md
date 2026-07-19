# Week 3 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on queries, written explanation, and a build-your-own-scenario task. Commit each.

All queries run against the `users`/`events` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Set up both engines (45 min)

Get **both** PostgreSQL and SQLite working against this week's seed.

1. Run `generate_seed.py`, load the CSVs into `crunch_scale` (Postgres) and `crunch_scale.db` (SQLite).
2. Confirm `SELECT COUNT(*) FROM users;` → `400` and `SELECT COUNT(*) FROM events;` → `5017` in **both**.
3. Run `SELECT event_name, COUNT(*) FROM events GROUP BY event_name ORDER BY 2 DESC;` in both engines and confirm the counts match each other exactly.

**Deliver** `setup.md` with the version of each engine and one sentence on a syntax difference you had to work around loading the CSVs (e.g. `\copy` vs `.import`).

---

## Problem 2 — Twenty warm-up queries (75 min)

Write and run each. Put them in `warmups.sql` with a `-- N` comment and the result beneath each.

1. Total signups per `channel`, sorted descending.
2. Total signups per `country`.
3. How many users are on the `'Pro'` plan vs. `'Free'`?
4. The earliest and latest `signup_at` in the dataset.
5. How many distinct `event_name` values exist in `events`?
6. How many `app_open` events exist in total (across all users, all time)?
7. The user with the most total events (any type) — their `user_id` and event count.
8. Everyone who reached `connected_integration` — list their `user_id` and `channel`.
9. How many users reached `created_first_board` but never reached `created_first_card`? *(These are users stuck one step earlier than this week's main diagnosis.)*
10. Of the users in Q9, what's their most common `channel`?
11. The average number of `app_open` events per user, across all 400 users (including zero for users who never returned).
12. How many users have **zero** events other than `signed_up`? *(Never even verified their email.)*
13. The five users with the **earliest** `invited_teammate` event (by `event_at`), and their `user_id`.
14. For each `plan`, the count of users who reached `invited_teammate` and the count who didn't.
15. The `channel` with the highest raw count of `invited_teammate` reaches (not rate — raw count).
16. How many events, in total, happened on a Saturday or Sunday? *(Hint: `EXTRACT(DOW ...)` in Postgres, `strftime('%w', ...)` in SQLite.)*
17. The average `ttv_hours` (signup → `invited_teammate`) for users acquired via `'Referral'` vs. `'Paid Search'`.
18. How many users fired **more than one** `app_open` event on the same calendar day at least once? *(A rough proxy for a highly engaged single-day session.)*
19. The month (by `signup_at`) with the most signups.
20. Confirm: does `COUNT(DISTINCT user_id)` from `events WHERE event_name = 'invited_teammate'` equal the `n_did` your Lecture 2 lift-table query reported for `invited`? Show both numbers.

---

## Problem 3 — Explain activation vs. acquisition vs. retention (45 min)

In `funnel-stages-writeup.md`, answer in prose (no more than 450 words total):

1. In your own words, define acquisition, activation, and retention, and explain why a metric belonging to one stage is usually a poor proxy for another (use "signup" and "payment" as your two examples, per Lecture 1).
2. Pick **one real product you use** (not Crunch Boards, not an example from the lectures) and propose what you think its activation event probably is. State your reasoning against the four criteria from Lecture 1 — you won't have the data to verify it, but you should be able to argue for it the way the lectures argued for `invited_teammate`.
3. Explain, in your own words, the difference between a lift table (Lecture 2) and a step funnel (Lecture 3) — what question does each answer that the other can't?

---

## Problem 4 — Percentiles across engines (45 min)

The same computation, both engines, in `percentiles.sql`:

1. **Postgres:** compute the p10, p50 (median), and p90 of `ttv_hours` (signup → `invited_teammate`) using `PERCENTILE_CONT`.
2. **SQLite:** compute the same three percentiles using the sorted-rows-plus-`OFFSET` technique.
3. Compare your two sets of numbers — they should be close but not necessarily bit-identical (interpolated vs. rank-based percentiles differ slightly). Explain the difference in one sentence.

**Deliver** the three queries per engine, plus a two-sentence note on when you'd reach for the SQLite fallback technique even on a Postgres project (hint: think about portability of analytics code that has to run in a lightweight test environment).

---

## Problem 5 — Extend the dataset with a new candidate event (60 min)

Make the scenario your own.

1. Propose a **new** candidate activation event Crunch Boards doesn't currently track (e.g. `used_mobile_app`, `created_second_board`, `commented_on_card`). Write 2–3 `INSERT` statements per user for **10 new synthetic users** (`user_id` 401–410) into `users`, and a plausible event sequence into `events` for each, including your new event for some of them and `app_open` events in later weeks for a few (so at least one has meaningful "week-4" activity).
2. Load them.
3. Re-run the reach query (Lecture 1 §5) and the lift-table query (Lecture 2 §3) — restricted to just your 10 new users plus your new candidate event — and report what you find (with only 10 users, don't over-interpret small numbers — say so explicitly).
4. Then **undo** it cleanly: delete both new tables' new rows and confirm you're back to 400 users / 5017 events.

**Deliver** `extend.sql` (the inserts, the two re-run queries with output, and the cleanup) plus 2–3 sentences on why a real growth team would never draw a strong conclusion from a 10-user sample, even though the mechanics of the query are identical to the 400-user version.

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
