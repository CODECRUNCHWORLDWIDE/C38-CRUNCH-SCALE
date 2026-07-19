# Exercise 1 — Stand Up the Capstone Warehouse

**Goal:** Turn the five `raw_*` tables into clean `stg_*` views and two core dimensions (`dim_user`, `dim_date`) — the foundation every later exercise, challenge, and the mini-project read from.

**Estimated time:** 90 minutes.

## Setup

Connected to `crunch_boards`, with the five sanity-check counts from the README confirmed (`72`, `56`, `18`, `35`, `24`). New `solutions.sql`, one `-- Task N` comment per task.

## Tasks

1. **`stg_users`.** Build the view from Lecture 1, Section 3 (lowercase/trim email, lowercase `signup_channel`, pass through the rest). *(Expected: 72 rows, `SELECT COUNT(DISTINCT signup_channel) FROM stg_users;` → **3**.)*

2. **`stg_events`.** Build the view filtered to `event_name = 'created_first_project'`, renaming `event_ts` to `event_date`. *(Expected: 56 rows — every row in `raw_events`, since this dataset only tracks the one activation event. `SELECT COUNT(DISTINCT user_id) FROM stg_events;` → **56** — confirm no user has *two* activation events; if you get fewer than 56 distinct users, something in your filter is wrong.)*

3. **`stg_marketing_spend`.** Straight pass-through with lowercased `channel`. *(Expected: 18 rows, `SELECT SUM(spend_usd) FROM stg_marketing_spend;` → **38,700.00**.)*

4. **`stg_subscriptions` — the cutoff filter.** Build it exactly as in Lecture 1, Section 3/4, including `WHERE created <= DATE '2025-06-30'`. Run both counts and compare:

   ```sql
   SELECT COUNT(*) FROM raw_subscriptions;    -- Expected: 35
   SELECT COUNT(*) FROM stg_subscriptions;    -- Expected: 30
   ```

   In a one-sentence comment above your `CREATE VIEW`, name the five `user_id`s that get excluded and why. *(Hint: `SELECT user_id, created FROM raw_subscriptions WHERE created > DATE '2025-06-30';`.)*

5. **`stg_experiment_assignments`.** Straight pass-through with lowercased `variant`. *(Expected: 24 rows, `SELECT variant, COUNT(*) FROM stg_experiment_assignments GROUP BY variant;` → **12 control / 12 treatment**, evenly split.)*

6. **`dim_date`.** Build the nine-row calendar spine from Lecture 1, Section 5 — Jan–Jun 2025 (`is_forecast = FALSE`) plus Jul–Sep 2025 (`is_forecast = TRUE`). *(Expected: 9 rows, `SELECT COUNT(*) FROM dim_date WHERE is_forecast = TRUE;` → **3**.)*

7. **`dim_user`.** Build it from `stg_users` + `stg_experiment_assignments`, adding `signup_month` and `in_experiment` as in Lecture 1, Section 6.

   ```sql
   -- starter shape — fill in the two derived columns
   CREATE TABLE dim_user AS
   SELECT
       u.user_id,
       u.email,
       u.company_name,
       u.signup_date,
       -- TODO: signup_month (first of the signup's month)
       u.signup_channel,
       u.plan_at_signup
       -- TODO: in_experiment (0/1)
   FROM stg_users u
   LEFT JOIN stg_experiment_assignments e ON e.user_id = u.user_id;
   ```

   *(Expected: 72 rows. `SELECT signup_channel, COUNT(*) FROM dim_user GROUP BY signup_channel;` → `paid_search 30 / organic 24 / referral 18`. `SELECT in_experiment, COUNT(*) FROM dim_user GROUP BY in_experiment;` → `0 → 48 / 1 → 24`.)*

8. **Orphan check.** Confirm every `stg_subscriptions` row joins cleanly to a `dim_user` row: `SELECT COUNT(*) FROM stg_subscriptions s LEFT JOIN dim_user u ON u.user_id = s.user_id WHERE u.user_id IS NULL;` *(Expected: **0**.)*

## Done when…

- [ ] All five `stg_*` views and both `dim_*` objects exist as real database objects.
- [ ] `stg_subscriptions` has 30 rows, not 35 — you can name the 5 excluded `user_id`s and explain the cutoff in one sentence.
- [ ] `dim_user` correctly flags exactly 24 users as `in_experiment = 1`.
- [ ] Task 8's orphan check returns 0.

## Stretch

- Add a `dim_channel` table with one row per channel (`paid_search`, `organic`, `referral`) and a hand-written one-sentence description of each channel's acquisition mechanic (paid ads, content/SEO, referral bonus) — you'll want a place to hang the CAC-model difference from Lecture 2, Section 3 that isn't a code comment.
- Write a single query that reports, per `stg_*` view, its row count and the `raw_*` table it's built from, side by side — a five-second "did staging drop anything it shouldn't have" health check.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-12/exercise-01/`.
