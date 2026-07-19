# Lecture 2 — Attribution Models

> **Duration:** ~2 hours. **Outcome:** You can build first-touch, last-touch, linear, and position-based attribution from a single `touches` table, in SQL, and explain — with real numbers from this week's data — why each model gives a different channel the credit for the same revenue.

Attribution is not a lookup. There is no column anywhere that says "channel X caused this sale" — causation isn't something an events table can record, only correlation in time. Attribution is a **modeling choice**: a rule you pick for splitting credit among the touches that preceded a conversion. Different rules reward different behavior, and picking one without saying so out loud is how a growth team ends up defunding the channel that was actually working.

## 1. The touch table

Every marketing touch a user has before converting lives in `touches` — one row per (user, channel, timestamp):

```sql
SELECT t.user_id, c.channel_name, t.touch_ts
FROM touches t
JOIN channels c ON c.channel_id = t.channel_id
WHERE t.user_id IN (9, 18)
ORDER BY t.user_id, t.touch_ts;
```

```
 user_id |      channel_name         |      touch_ts
---------+----------------------------+---------------------
       9 | Meta Ads                   | 2025-01-02 08:00:00
       9 | Meta Ads                   | 2025-01-04 08:15:00
       9 | Email                      | 2025-01-07 09:00:00
      18 | Google Search - Nonbrand   | 2025-01-01 07:30:00
      18 | Meta Ads                   | 2025-01-05 07:45:00
      18 | Google Search - Brand      | 2025-01-09 08:00:00
```

User 9 hit Meta Ads twice (a retargeting impression, then a click) before an email nudge closed them. User 18 touched three *different* channels. Both converted and both are worth $149. Who gets the credit?

## 2. First-touch attribution

**Rewards discovery.** First-touch gives 100% of the credit to the earliest touch before conversion — the channel that made the user aware Crunch existed at all.

```sql
WITH first_touch AS (
    SELECT
        user_id,
        channel_id,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC) AS touch_rank
    FROM touches
)
SELECT
    c.channel_name,
    COUNT(*)          AS conversions_credited,
    SUM(cv.revenue)   AS revenue_credited
FROM first_touch ft
JOIN conversions cv ON cv.user_id = ft.user_id
JOIN channels c      ON c.channel_id = ft.channel_id
WHERE ft.touch_rank = 1
GROUP BY c.channel_name
ORDER BY revenue_credited DESC;
```

`ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC)` numbers each user's touches 1, 2, 3... in time order; filtering to `touch_rank = 1` keeps only the earliest. Run this against all ten converters and you get:

```
       channel_name         | conversions_credited | revenue_credited
-----------------------------+-----------------------+-------------------
 Organic Search              |                     2 |               498
 Affiliate                   |                     1 |               349
 Google Search - Nonbrand    |                     3 |               347
 Meta Ads                    |                     2 |               198
 Referral                    |                     1 |               149
 Google Search - Brand       |                     1 |                49
```

Notice `Google Search - Brand` — a channel that will look fantastic under last-touch (next section) — earns credit for only **one** conversion here, and it's the cheap Starter plan. Under first-touch, discovery-heavy channels (Organic, Affiliate, Nonbrand search) dominate, because that's what the model measures: who got there first.

## 3. Last-touch attribution

**Rewards closing.** Last-touch gives 100% of the credit to the touch immediately before conversion — flip the sort order and the filter:

```sql
WITH last_touch AS (
    SELECT
        user_id,
        channel_id,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts DESC) AS touch_rank
    FROM touches
)
SELECT
    c.channel_name,
    COUNT(*)          AS conversions_credited,
    SUM(cv.revenue)   AS revenue_credited
FROM last_touch lt
JOIN conversions cv ON cv.user_id = lt.user_id
JOIN channels c      ON c.channel_id = lt.channel_id
WHERE lt.touch_rank = 1
GROUP BY c.channel_name
ORDER BY revenue_credited DESC;
```

```
       channel_name         | conversions_credited | revenue_credited
-----------------------------+-----------------------+-------------------
 Email                       |                     5 |              1145
 Google Search - Brand       |                     2 |               198
 Organic Search              |                     1 |               149
 Meta Ads                    |                     1 |                49
 Google Search - Nonbrand    |                     1 |                49
```

Total credited revenue here is **$1,590** — same as first-touch, same as `SELECT SUM(revenue) FROM conversions;`. That's not a coincidence, it's a correctness check: any single-touch model (first- or last-) assigns exactly 100% of each conversion's revenue to exactly one channel, so the grand total must always reconcile to total revenue. **If a first-touch or last-touch report you write doesn't sum back to total revenue, you have a bug** — usually a missing `touch_rank = 1` filter, or a tie in `touch_ts` producing more than one rank-1 row per user and double-crediting a conversion. Reconcile every attribution report against `SUM(revenue) FROM conversions` before you trust it.

`Email` dominates last-touch because it's structurally a *closing* channel here — every multi-touch user in this dataset ends their path with an email nudge. That's realistic: email rarely creates awareness, it reminds someone who already knows you. Crediting Email with the "acquisition," full stop, would defund the channels that did the actual discovery work. Lecture 3 comes back to this.

## 4. Linear attribution

**Rewards presence.** Linear splits credit evenly across every touch in the path — a 1-touch path gives 100% to that touch, a 3-touch path gives ⅓ each:

```sql
WITH touch_counts AS (
    SELECT user_id, COUNT(*) AS n_touches
    FROM touches
    GROUP BY user_id
)
SELECT
    c.channel_name,
    ROUND(SUM(cv.revenue / tc.n_touches::numeric), 2) AS revenue_credited
FROM touches t
JOIN touch_counts tc ON tc.user_id = t.user_id
JOIN conversions cv  ON cv.user_id = t.user_id
JOIN channels c       ON c.channel_id = t.channel_id
GROUP BY c.channel_name
ORDER BY revenue_credited DESC;
```

For user 18 (3 touches, $149 revenue), each of Google Nonbrand, Meta Ads, and Google Brand gets `149 / 3 = 49.67`. For user 9 (3 touches, $149), Meta Ads gets credited **twice** — once per touch — for `2 × (149/3) = 99.33`, and Email gets `49.67`. Linear is the only model of the four where a channel touching the same user twice earns proportionally more, which is exactly right if you believe repeated exposure has value, and exactly wrong if you think it's double-counting the same click. Decide before you build the report, not after someone asks why Meta's number moved.

On SQLite, drop the `::numeric` cast — SQLite doesn't need it, but do make sure at least one operand in `revenue / n_touches` is a float (`revenue * 1.0 / n_touches`) or integer division will floor your fractional credit to zero on 1-vs-3 splits.

## 5. Position-based (U-shaped) attribution

**Rewards discovery and closing, discounts the middle.** The most common model in real marketing stacks: 40% of credit to the first touch, 40% to the last touch, and the remaining 20% split evenly across whatever touches happened in between. A 1-touch path is 100% to that touch (there's no "middle" to discount). A 2-touch path is 50/50 (no room for a middle weight either — teams differ on this edge case; we split the 40/40 rule to 50/50 when there's no third touch, keeping first and last equal).

```sql
WITH ranked AS (
    SELECT
        t.user_id,
        t.channel_id,
        ROW_NUMBER() OVER (PARTITION BY t.user_id ORDER BY t.touch_ts ASC)  AS rn,
        COUNT(*)     OVER (PARTITION BY t.user_id)                          AS n_touches
    FROM touches t
),
weighted AS (
    SELECT
        user_id,
        channel_id,
        CASE
            WHEN n_touches = 1                        THEN 1.0
            WHEN n_touches = 2 AND rn IN (1, 2)        THEN 0.5
            WHEN rn = 1                                THEN 0.4
            WHEN rn = n_touches                        THEN 0.4
            ELSE 0.2 / (n_touches - 2)                 -- split among middle touches
        END AS weight
    FROM ranked
)
SELECT
    c.channel_name,
    ROUND(SUM(cv.revenue * w.weight), 2) AS revenue_credited
FROM weighted w
JOIN conversions cv ON cv.user_id = w.user_id
JOIN channels c       ON c.channel_id = w.channel_id
GROUP BY c.channel_name
ORDER BY revenue_credited DESC;
```

Two window functions stacked in `ranked`: `ROW_NUMBER()` gives each touch its position, and `COUNT(*) OVER (PARTITION BY user_id)` — with no `ORDER BY` inside the window — gives every row in that user's group the *same* value: the total touch count. That second pattern (a window `COUNT`/`SUM` with no `ORDER BY`) is how you attach a group-level total to every detail row without a separate aggregate-and-join step.

For user 18's 3-touch path: Google Nonbrand (first) gets 40% of $149 = $59.60, Meta Ads (middle, the only one) gets the full 20% = $29.80, Google Brand (last) gets 40% = $59.60. For user 9's 3-touch path (Meta, Meta, Email): the *first* Meta touch gets 40%, the *second* Meta touch is the middle at 20%, Email (last) gets 40% — so Meta Ads ends up with 60% of that user's revenue between its two touches, and position-based, unlike linear, still distinguishes "first exposure" from "everything else."

## 6. Comparing all four on the same conversions

| Channel | First-touch | Last-touch | Linear | Position-based |
|---|---:|---:|---:|---:|
| Organic Search | $498 | $149 | $323.50 | $323.50 |
| Google Search - Nonbrand | $347 | $49 | $173.17 | $183.10 |
| Affiliate | $349 | $0 | $174.50 | $174.50 |
| Meta Ads | $198 | $49 | $198.00 | $168.20 |
| Google Search - Brand | $49 | $198 | $98.67 | $108.60 |
| Email | $0 | $1,145 | $547.67 | $557.60 |
| Referral | $149 | $0 | $74.50 | $74.50 |
| **Total** | **$1,590** | **$1,590** | **$1,590** | **$1,590** |

*(These are the exact figures from running all four queries against the seed data — verify your own output against this table in Exercises 1–2 and Challenge 1. Every column sums to $1,590, the total revenue in `conversions` — your check that each model, however it internally splits credit, is fully reconciled.)*

Read the table by row, not by column: `Google Search - Brand` swings from **$49** (first-touch) to **$198** (last-touch) — a 4x difference for the exact same two conversions, purely because of which model you pick. That's not noise; it's the whole reason this lecture exists. A team that only ever runs last-touch reports will systematically overvalue closing channels and defund discovery channels, then wonder why the top of the funnel dries up two quarters later.

## 7. Which model to use when

- **First-touch** — answer "what's driving new awareness?" Good for evaluating top-of-funnel/brand-building spend, bad for evaluating retargeting or lifecycle channels (they'll always show near-zero).
- **Last-touch** — answer "what's converting people who are already in motion?" Good for evaluating landing pages, checkout flows, and closing nudges; systematically undervalues everything upstream. Most ad platforms' self-reported conversion numbers are last-touch (or worse, last-touch-within-their-own-platform, ignoring every other channel entirely) — never take a platform's own dashboard as neutral.
- **Linear** — answer "how much total exposure is each channel providing?" Fair to every touch but treats a single repeated impression the same as three genuinely different channels — can overweight retargeting-heavy paths.
- **Position-based** — the pragmatic default for most growth teams: acknowledges that discovery and closing both matter more than the muddled middle, without zeroing anything out entirely.

No model is "correct." The discipline is running more than one, on the same data, and only trusting a channel decision that holds up across models. When first-touch and last-touch tell wildly different stories about the same channel (as `Google Search - Brand` does here), that disagreement *is* the finding — Lecture 3 tells you exactly what to do with it.

## 8. Check yourself

- Why does `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC)` give you first-touch, and what single change gives you last-touch?
- For a 3-touch path, what fraction does the middle touch get under linear vs. under position-based (40/20/40)?
- What does `COUNT(*) OVER (PARTITION BY user_id)` (no `ORDER BY`) compute, and how does that differ from `COUNT(*) GROUP BY user_id`?
- Explain, using user 9's data, why linear attribution gives Meta Ads more total credit than position-based does for that same user.
- Why is it dangerous to trust a single ad platform's self-reported conversion count as your attribution model?
- Total revenue across all conversions is $1,590. Does that number change depending on which attribution model you run? Why or why not?

Lecture 3 uses these same models — plus the `ad_spend` table you haven't touched yet — to answer the question underneath all of this: which channel actually deserves the next dollar.

## Further reading

- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — `ROW_NUMBER`, frame clauses:** <https://www.postgresql.org/docs/current/functions-window.html>
- **Google Analytics — Attribution models overview** (concepts transfer even if you never touch the tool): <https://support.google.com/analytics/answer/10596866>
- **SQLite — Window Functions:** <https://www.sqlite.org/windowfunctions.html>
