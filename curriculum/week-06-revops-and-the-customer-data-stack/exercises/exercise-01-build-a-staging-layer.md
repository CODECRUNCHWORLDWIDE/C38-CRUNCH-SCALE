# Exercise 1 — Build a Staging Layer

**Goal:** Turn all four raw tables into clean, typed, conformed `stg_*` models — the layer every mart in this course will read from instead of touching `raw_*` directly.

**Estimated time:** 90 minutes.

## Setup

Connected to `crunch_flow`, with the four sanity-check counts from the README confirmed (`15`, `58`, `18`, `15`). New `solutions.sql` under `-- Task N` comments.

## Tasks

1. **`stg_subscriptions`.** Create it exactly as built in Lecture 2, Section 4 (join `raw_stripe_subscriptions` to `stg_users` on email, normalize annual amounts to monthly `mrr_cents`). Query `SELECT COUNT(*) FROM stg_subscriptions;`. *(Expected: **18** rows — one per real Stripe subscription object, including Ben's, Drew's, and Ines's two-rows-each subscription history. If you get 15, you deduped away real history — go re-read Lecture 2 Section 4's warning about that.)*

2. **`stg_users`.** Create the view from Lecture 1, Section 2 (lowercase/trim email, pass through the rest). Query it. *(Expected: 15 rows, no `NULL` emails, all lowercase.)*

3. **`stg_events`.** Build a staging view over `raw_events` that: casts `event_ts` to a proper timestamp type (`event_ts::timestamp` on Postgres; SQLite already stores it as comparable text, so a plain pass-through is fine there), and adds a derived `event_date` column (`event_ts::date` / `date(event_ts)`) for easy day-level grouping later. No rows should be dropped — every raw event is legitimate. *(Expected: 58 rows, `SELECT COUNT(DISTINCT event_name) FROM stg_events;` → **9** distinct event names.)*

   ```sql
   -- starter shape — fill in the two derived columns
   CREATE VIEW stg_events AS
   SELECT
       event_id,
       user_id,
       event_name,
       event_ts,          -- TODO: cast properly
       -- TODO: event_date
       source
   FROM raw_events;
   ```

4. **`stg_app_subscriptions`.** This is the new one — build a staging view over `raw_app_subscriptions` that conforms the **legacy plan names** to the modern taxonomy and normalizes `price_usd` into the same `mrr_cents` (monthly, in cents) shape `stg_subscriptions` uses, so the two are directly comparable in Exercise 3:

   ```sql
   CREATE VIEW stg_app_subscriptions AS
   SELECT
       user_id,
       CASE plan
           WHEN 'Basic'      THEN 'Starter'
           WHEN 'Pro'        THEN 'Growth'
           WHEN 'Enterprise' THEN 'Scale'
           ELSE plan
       END                                                          AS plan,
       status,
       CASE billing_cycle
           WHEN 'annual' THEN ROUND(price_usd * 100 / 12.0)
           ELSE ROUND(price_usd * 100)
       END                                                          AS mrr_cents,
       billing_cycle,
       start_date,
       end_date
   FROM raw_app_subscriptions;
   ```

   Run it and inspect user 9 (Ines), user 11 (Kira), and user 13 (Mira) — you'll need these three rows for Exercise 3. *(Expected: 15 rows, one per user, `plan` values only ever `Starter`/`Growth`/`Scale`.)*

5. **Join sanity check.** Confirm every row in `stg_subscriptions` joins cleanly to a real `dim_user`-eligible row in `stg_users` — i.e., no orphaned subscriptions. `SELECT COUNT(*) FROM stg_subscriptions s LEFT JOIN stg_users u ON u.user_id = s.user_id WHERE u.user_id IS NULL;` *(Expected: **0**. If you get a nonzero count, your email `JOIN` in `stg_subscriptions` is silently dropping or misjoining rows — check case and whitespace.)*

6. **Legacy-name audit.** Before you trust Task 4's `CASE` mapping, prove it's exhaustive: `SELECT DISTINCT plan FROM raw_app_subscriptions WHERE plan NOT IN ('Basic','Pro','Enterprise');` *(Expected: **0 rows** — if this ever returns something, your `CASE` in Task 4 has an unmapped value silently falling through the `ELSE` branch.)*

## Done when…

- [ ] All four `stg_*` objects exist as real database objects (`\d` in psql or `.tables` in sqlite3 shows them).
- [ ] `stg_subscriptions` has 18 rows, not 15 — you can explain in one sentence why deduping to "latest per customer" would have been a bug here.
- [ ] `stg_app_subscriptions` uses only the modern plan taxonomy, never `Basic`/`Pro`/`Enterprise`.
- [ ] Task 5's orphan check returns 0.
- [ ] Task 6's audit returns 0 rows.

## Stretch

- Add a `source_system` literal column (`'stripe'` / `'internal_app'`) to `stg_subscriptions` and `stg_app_subscriptions` respectively — you'll want it in Exercise 3 when you `UNION` or compare the two.
- `stg_events` currently has no notion of "session" — group consecutive events per user that are within 30 minutes of each other into a synthetic `session_id` using a window function (`LAG`). This is a genuine data-engineering pattern, not just practice; you'll meet it again if you take Week 2's funnel work further.
- Write one query that shows, per `event_name`, the count of events and the earliest and latest `event_ts` — a five-second "does this table look sane" health check you should run after every real-world load.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-06/exercise-01/`.
