# Week 1 — Homework

Five problems, ~5 hours total, spread across the week. A mix of hands-on SQL against the StreakLab seed, written analysis, and one build-your-own-data task. Commit each.

All queries run against the `users`/`events` seed from the [README](./README.md) unless a problem says otherwise. "Today" is **2026-06-21**.

---

## Problem 1 — Reproduce every headline number (60 min)

Without looking back at the lecture files, write fresh SQL for each of these from memory (peek only if truly stuck, then note which ones you peeked at):

1. Total signups.
2. Activation rate (count + %).
3. Weekly Engaged Users as of 2026-06-21.
4. Subscriber count + total new MRR.
5. Invites sent + accepted.

**Deliver** `reproduce.sql` with all 5 queries and a one-line note on which (if any) you needed to check a lecture for.

---

## Problem 2 — Channel-level breakdown (60 min)

Lecture 2 computed each AARRR headline number **overall**. This problem breaks every one of them down **by `signup_channel`**.

1. Activation rate by channel (5 rows: channel, activated count, total count, %).
2. Weekly Engaged Users by channel, as of 2026-06-21.
3. Subscriber count and total revenue by channel.
4. Write two sentences: which channel looks strongest across all three numbers, and which channel looks like it's producing signups but not much else?

**Deliver** `by-channel.sql` (the 3 queries) plus `by-channel-notes.md` (the two sentences).

---

## Problem 3 — Explain the metric, precisely (45 min)

In `metric-writeup.md`, answer in prose (no more than 450 words total):

1. Write StreakLab's Weekly Engaged Users definition precisely enough that a new engineer, with no other context, could implement the `WHERE`/`HAVING` clause correctly from your sentence alone. (This is harder than it sounds — try it on a friend or a rubber duck and see if they'd get it right.)
2. Explain, in your own words, the difference between a **leading** and a **lagging** indicator, using one StreakLab example of each.
3. Explain why "80% activation rate" is meaningless on its own without also stating the denominator (15 users), referencing Lecture 3, section 5.
4. Name one metric from this week that you think is currently borderline between vanity and actionable for StreakLab specifically, and defend your call.

---

## Problem 4 — Time-bucketed trend (60 min)

StreakLab's WEU as of a single day (2026-06-21) is one point. A trend needs several.

1. Compute Weekly Engaged Users **as of** each of these three dates: 2026-06-14, 2026-06-18, 2026-06-21. (Same definition each time — 3+ check-ins in the trailing 7 days ending on that date — just three different `event_time` windows.)
2. Put the three numbers in a small markdown table in `trend.md`: date, WEU, and one sentence on the direction and a guess at *why*, based on what you know about the cohorts (remember: cohort C only signed up 2026-06-15, so it barely exists as of 2026-06-14).
3. Write the **single SQL query** (not three separate ones) that would produce all three numbers in one result set, using a `GENERATE_SERIES` of the three as-of dates cross-joined against the check-in logic. *(This is a preview of the kind of query Week 4's retention-curve work builds constantly — don't worry if it takes a few tries.)*

**Deliver** `trend.md` (table + reasoning) and `trend.sql` (the single combined query).

---

## Problem 5 — Extend the dataset (75 min)

Make the data your own and query it — same spirit as C33's extension homework, applied to events instead of a single table.

1. Invent **3 new StreakLab users** (`user_id` 16, 17, 18) signing up on 2026-06-20, one from each of three different `signup_channel` values you choose.
2. Write realistic `events` rows for each: at minimum a `session_start`, and for at least one of them a `habit_created` and a few `checkin_logged` events, so at least one of your 3 new users would count as activated.
3. Load them (you now have 18 users).
4. Recompute Problem 1's five headline numbers with your 3 new users included, and note in `extend-notes.md` exactly which numbers changed and by how much.
5. Then **undo** it cleanly: delete your 3 users' events first (foreign key), then the users, and confirm you're back to 15 users / 149 events.

**Deliver** `extend.sql` (inserts, the 5 recomputed queries, and the cleanup) plus `extend-notes.md` (what changed).

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 75 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
