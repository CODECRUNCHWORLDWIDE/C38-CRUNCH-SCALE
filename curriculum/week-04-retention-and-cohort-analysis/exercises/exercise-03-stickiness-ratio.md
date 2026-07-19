# Exercise 3 — Compute DAU/MAU Stickiness

**Goal:** Compute the stickiness ratio for the whole product across three months, then break it down by segment, and interpret what the numbers say about usage frequency.

**Estimated time:** 60 minutes.

## Setup

```sql
SELECT COUNT(*) FROM users;    -- 48
SELECT COUNT(*) FROM events;   -- 366
```

Create `solutions.sql`.

## Tasks

1. **MAU for one month.** Write a query returning the count of distinct users active at any point during June 2025 (`event_date LIKE '2025-06%'`). *(Expected: 19.)*

2. **Daily active users.** Write a query returning `event_date` and `COUNT(DISTINCT user_id)` for every day in June 2025 that has at least one event. *(Expected: several dozen rows, one per distinct active day — not all 30 days will appear, since some June days have zero activity in this seed.)*

3. **Average DAU.** Using Task 2, compute the average daily active users across **all 30 days of June** (days with zero activity count as 0 — divide the sum of daily actives by 30, not by the number of *rows* from Task 2). *(Expected: 2.60.)*

4. **Stickiness for one month.** Combine Tasks 1 and 3 into `stickiness = 100 * avg_DAU / MAU`. *(Expected: 13.7%.)*

5. **Stickiness trend.** Repeat Task 4 for April, May, and June 2025 in one query (`GROUP BY` the month instead of hardcoding it). *(Expected: April 13.7%, May 14.1%, June 13.7% — MAU is growing, but the ratio is stable.)*

6. **Stickiness by segment.** Repeat Task 4, but split by `segment`, for June 2025 only. *(Expected: `self_serve` ≈ 9.3%, `sales_assisted` ≈ 18.5%.)*

## Expected result (spot checks)

| month | MAU | avg DAU | stickiness |
|---|---:|---:|---:|
| 2025-04 | 19 | 2.60 | 13.7% |
| 2025-05 | 24 | 3.39 | 14.1% |
| 2025-06 | 19 | 2.60 | 13.7% |

| segment (June) | MAU | avg DAU | stickiness |
|---|---:|---:|---:|
| self_serve | 10 | 0.93 | 9.3% |
| sales_assisted | 9 | 1.67 | 18.5% |

## Done when…

- [ ] `solutions.sql` has all 6 queries, each under a `-- Task N` comment.
- [ ] Task 5 and Task 6's numbers match the tables above.
- [ ] You can state, in one sentence, what "13.7% stickiness" means in terms an average MAU would recognize about their own usage pattern (hint: express it as "roughly 1 day in every N").
- [ ] You can explain why `sales_assisted` being exactly ~2× as sticky as `self_serve` is consistent with what Lecture 3's segment retention table already showed — these are two different measurements pointing at the same underlying difference.

## Stretch

- Compute a **weekly** active users (WAU) version of stickiness (`avg WAU / MAU`) for June. Is a WAU/MAU ratio a more or less forgiving metric than DAU/MAU, and for what kind of product would you prefer it?
- The seed dataset is small (48 users), so daily counts are sparse — most calendar days have 0 or 1 active user. Write two sentences on why this makes the *segment gap* (9.3% vs. 18.5%) a more trustworthy signal than the *exact magnitude* of either number, and what you'd want before reporting these figures to a board.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-04/exercise-03/`.
