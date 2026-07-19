# Lecture 3 — Reading Channels That Scale

> **Duration:** ~2 hours. **Outcome:** You can join `ad_spend` to attributed conversions to compute CAC under more than one model, spot a channel whose spend is climbing faster than its results (saturation), and tell a channel that's genuinely driving growth from one that's just relabeling demand another channel already created.

Lecture 2 showed you that `Google Search - Brand` looks completely different depending on which attribution model you run — $49 under first-touch, $198 under last-touch. This lecture is about what to *do* with that disagreement, and about the harder, more expensive version of the same trap: a channel whose CAC report looks great in isolation and terrible the moment you put it next to anything else.

## 1. CAC is a ratio, and a ratio hides its two halves

Customer Acquisition Cost is spend divided by conversions attributed to that spend. It looks like one number. It is always secretly two:

```sql
SELECT
    c.channel_name,
    SUM(s.spend_amount)                            AS total_spend,
    COUNT(DISTINCT lt.user_id)                      AS conversions,
    ROUND(SUM(s.spend_amount)
          / NULLIF(COUNT(DISTINCT lt.user_id), 0), 2) AS cac
FROM channels c
JOIN ad_spend s ON s.channel_id = c.channel_id
LEFT JOIN (
    SELECT user_id, channel_id
    FROM (
        SELECT user_id, channel_id,
               ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts DESC) AS rn
        FROM touches
    ) ranked
    WHERE rn = 1
) lt ON lt.channel_id = c.channel_id
GROUP BY c.channel_name
ORDER BY cac NULLS LAST;
```

```
       channel_name         | total_spend | conversions |   cac
-----------------------------+-------------+-------------+---------
 Google Search - Brand       |         400 |           2 |  200.00
 Meta Ads                    |        1700 |           1 | 1700.00
 Google Search - Nonbrand    |        1700 |           1 | 1700.00
 Affiliate                   |         700 |           0 |
```

`NULLIF(COUNT(...), 0)` turns a zero-conversion denominator into `NULL` instead of crashing on a divide-by-zero — the correct SQL idiom for "this ratio is undefined, not zero." Affiliate's CAC is blank, which is a more honest answer than any number: $700 spent, zero last-touch conversions, an undefined return.

Read only this table and `Google Search - Brand` is the hero of the growth report: cheapest CAC by a factor of 8.5x. That conclusion is wrong, and Section 3 proves it with this exact data.

## 2. Marginal CAC and the saturation curve

Total CAC (spend ÷ total conversions) tells you what the channel has cost *on average*. It says nothing about what the **next dollar** would cost — and the next dollar is the only one a growth team actually gets to decide about. Look at Meta Ads' daily spend against the *dates* its touches actually land:

```sql
SELECT spend_date, spend_amount
FROM ad_spend
WHERE channel_id = 7   -- Meta Ads
ORDER BY spend_date;
```

```
 spend_date | spend_amount
------------+---------------
 2025-01-01 |            80
 2025-01-02 |           100
 2025-01-03 |           120
 2025-01-04 |           140
 2025-01-05 |           160
 2025-01-06 |           180
 2025-01-07 |           200
 2025-01-08 |           220
 2025-01-09 |           240
 2025-01-10 |           260
```

```sql
SELECT DATE(touch_ts) AS touch_day, COUNT(*) AS meta_touches
FROM touches
WHERE channel_id = 7
GROUP BY DATE(touch_ts)
ORDER BY touch_day;
```

```
 touch_day  | meta_touches
------------+---------------
 2025-01-01 |             1
 2025-01-02 |             1
 2025-01-04 |             1
 2025-01-05 |             2
```

Spend on Meta Ads more than triples across the window — $80/day to $260/day — while every single touch happened in the **first five days**, on spend of $80–$160/day. Days 6 through 10 spent $180, $200, $220, $240, and $260 — over $1,100 total — and produced **zero** additional touches, let alone conversions. That's a saturation curve hiding in a spend table: the channel had an effective ceiling around $160/day, and every dollar past it was, in this window, pure waste. A total-CAC number ($1,700 ÷ 1 = $1,700) doesn't show you this. Splitting spend by period and counting results *in that same period* does:

```sql
SELECT
    CASE WHEN spend_date <= '2025-01-05' THEN 'days 1-5' ELSE 'days 6-10' END AS period,
    SUM(spend_amount) AS spend
FROM ad_spend
WHERE channel_id = 7
GROUP BY 1;
```

```
   period    | spend
-------------+-------
 days 1-5    |   600
 days 6-10   |  1100
```

$600 in the first five days bought every Meta touch this dataset has. $1,100 in the second five days bought nothing measurable. That's the question "what did we spend" can never answer and "what did the *next* dollar buy" always can — and it's a period-over-period `GROUP BY`, not a fancier model, that surfaces it.

## 3. Channels that only reshuffle organic demand

Now the harder trap. Section 1's table crowned `Google Search - Brand` the cheapest channel at $200 CAC. Before you believe it, pull the **full path**, not just the last touch, for each of its two credited conversions:

```sql
SELECT t.user_id, c.channel_name, t.touch_ts,
       ROW_NUMBER() OVER (PARTITION BY t.user_id ORDER BY t.touch_ts) AS touch_order
FROM touches t
JOIN channels c ON c.channel_id = t.channel_id
WHERE t.user_id IN (
    SELECT user_id FROM touches WHERE channel_id = 6   -- Google Search - Brand
)
ORDER BY t.user_id, t.touch_ts;
```

```
 user_id |       channel_name        |      touch_ts        | touch_order
---------+----------------------------+-----------------------+-------------
      11 | Google Search - Brand      | 2025-01-04 14:00:00  |           1
      18 | Google Search - Nonbrand   | 2025-01-01 07:30:00  |           1
      18 | Meta Ads                   | 2025-01-05 07:45:00  |           2
      18 | Google Search - Brand      | 2025-01-09 08:00:00  |           3
```

User 11 is genuine: `Google Search - Brand` is their *only* touch — they searched Crunch's brand name directly and converted. That's a real, if small, acquisition. User 18 is not: they were **found by Nonbrand search on day 1, retargeted by Meta on day 5**, and only searched the brand name directly on day 9 — after two other channels had already done the work of creating awareness and intent. Last-touch handed 100% of that conversion's credit to the channel that happened to be nearest the finish line. This is the textbook shape of a channel that **reshuffles organic demand**: users type your brand name into Google right before converting *because another channel already convinced them to*, and brand search harvests credit that belongs upstream.

You don't need a fancier attribution model to catch this — you need to run the query that checks whether a channel's converters have a longer path *behind* their last touch, and look at it:

```sql
WITH lt AS (
    SELECT user_id, channel_id,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts DESC) AS rn
    FROM touches
),
path_length AS (
    SELECT user_id, COUNT(*) AS n_touches
    FROM touches
    GROUP BY user_id
)
SELECT
    c.channel_name,
    COUNT(*)                                   AS last_touch_conversions,
    ROUND(AVG(pl.n_touches), 2)                AS avg_path_length_behind_it
FROM lt
JOIN conversions cv ON cv.user_id = lt.user_id
JOIN channels c      ON c.channel_id = lt.channel_id
JOIN path_length pl  ON pl.user_id = lt.user_id
WHERE lt.rn = 1
GROUP BY c.channel_name
ORDER BY avg_path_length_behind_it DESC;
```

```
       channel_name         | last_touch_conversions | avg_path_length_behind_it
-----------------------------+--------------------------+----------------------------
 Email                       |                        5 |                       2.20
 Google Search - Brand       |                        2 |                       2.00
 Google Search - Nonbrand    |                        1 |                       1.00
 Meta Ads                    |                        1 |                       1.00
 Organic Search              |                        1 |                       1.00
```

An average path length above 1 behind a last-touch channel is your signal to stop trusting its CAC in isolation and go read the actual paths, the way we just did for `Google Search - Brand`. A channel whose converters average 1.0 touches (like Organic Search here) earned its credit outright — there was nothing upstream to reshuffle.

## 4. CAC alongside conversion, never alone

The fix isn't "always use first-touch" or "always use last-touch" — it's reporting more than one number per channel so a single cheap ratio can't carry the whole recommendation. A minimum useful channel report has spend, conversions under at least two models, CAC under both, and a volume signal (top-of-funnel touches):

```sql
WITH ft AS (
    SELECT user_id, channel_id,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC) AS rn
    FROM touches
),
lt AS (
    SELECT user_id, channel_id,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts DESC) AS rn
    FROM touches
),
spend AS (
    SELECT channel_id, SUM(spend_amount) AS total_spend
    FROM ad_spend GROUP BY channel_id
)
SELECT
    c.channel_name,
    c.channel_type,
    sp.total_spend,
    COUNT(DISTINCT ft.user_id) FILTER (WHERE ft.rn = 1 AND cv1.user_id IS NOT NULL) AS first_touch_conversions,
    COUNT(DISTINCT lt.user_id) FILTER (WHERE lt.rn = 1 AND cv2.user_id IS NOT NULL) AS last_touch_conversions,
    ROUND(sp.total_spend / NULLIF(COUNT(DISTINCT ft.user_id)
          FILTER (WHERE ft.rn = 1 AND cv1.user_id IS NOT NULL), 0), 2) AS first_touch_cac,
    ROUND(sp.total_spend / NULLIF(COUNT(DISTINCT lt.user_id)
          FILTER (WHERE lt.rn = 1 AND cv2.user_id IS NOT NULL), 0), 2) AS last_touch_cac
FROM channels c
LEFT JOIN spend sp ON sp.channel_id = c.channel_id
LEFT JOIN ft ON ft.channel_id = c.channel_id
LEFT JOIN conversions cv1 ON cv1.user_id = ft.user_id
LEFT JOIN lt ON lt.channel_id = c.channel_id
LEFT JOIN conversions cv2 ON cv2.user_id = lt.user_id
WHERE c.channel_type = 'paid'
GROUP BY c.channel_name, c.channel_type, sp.total_spend
ORDER BY c.channel_name;
```

`FILTER (WHERE ...)` is the clean way to compute several conditional counts off one `GROUP BY` without stacking `CASE WHEN ... END` inside every aggregate — PostgreSQL supports it natively. SQLite doesn't have `FILTER`; rewrite each as `COUNT(DISTINCT CASE WHEN <condition> THEN user_id END)`. Run it, and the four paid channels tell a coherent story together that none of them tell alone:

- **Google Search - Nonbrand** — $1,700 spent, cheap-ish first-touch CAC (~$567), expensive last-touch CAC ($1,700). Reads as: a genuine discovery channel, undervalued by any report that only looks at who closed.
- **Meta Ads** — $1,700 spent, saturating past day 5 (Section 2), first-touch CAC ~$850, last-touch CAC $1,700. Reads as: works for discovery up to a spend ceiling; scaling it further without expanding audience or creative won't buy more customers.
- **Google Search - Brand** — $400 spent, cheap last-touch CAC ($200), but half its "conversions" (Section 3) had real upstream discovery work behind them. Reads as: partly genuine, partly harvesting credit from Nonbrand and Meta — don't cut those two to fund more brand spend.
- **Affiliate** — $700 spent, touches three users, zero of them last-touch, one converts at all (through a different channel's last touch). Reads as: the weakest of the four — Challenge 2 makes you prove this one with a query someone got wrong.

## 5. Where the next dollar goes

Put spend, both CAC views, and the saturation and reshuffling checks on one page and a recommendation falls out without needing a single new number: hold or trim Affiliate (Section 4), don't scale Meta Ads past its saturation point without new audiences (Section 2), don't take `Google Search - Brand`'s last-touch CAC at face value when allocating new budget (Section 3), and protect or grow Nonbrand search and Organic — the two channels that keep showing up as genuine, undervalued discovery sources across every check in this lecture. That's the exact recommendation the mini-project asks you to write up, with your own numbers as the evidence.

## 6. Check yourself

- Why does `NULLIF(count, 0)` matter when computing CAC, and what would happen without it?
- What's the difference between "total CAC" and "marginal CAC," and which one should drive a decision to increase spend?
- Using this week's data, explain in one sentence why Meta Ads' spend curve is a saturation signal even though its total CAC ($1,700) looks similar to Google Nonbrand's.
- What query pattern lets you check whether a last-touch channel's conversions had real work done upstream, without building a whole new attribution model?
- Why is Affiliate's CAC `NULL` rather than `0` in Section 1's query, and why does that distinction matter?
- Name three numbers a channel report needs, at minimum, so that a single cheap CAC can't drive a bad budget decision by itself.

You now have the full toolkit for this week: funnel counting and windowing (Lecture 1), four attribution models (Lecture 2), and CAC read correctly against spend, saturation, and upstream credit (Lecture 3). The exercises make you build all three from scratch; the challenges make you defend and debug them.

## Further reading

- **PostgreSQL — Aggregate expressions (`FILTER`):** <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
- **PostgreSQL — `NULLIF`, `COALESCE`:** <https://www.postgresql.org/docs/current/functions-conditional.html>
- **CXL — "Marketing Attribution: A Marketer's Guide"** (concepts, vendor-neutral parts): <https://cxl.com/blog/attribution-modelling/>
- **SQLite — Aggregate Functions (no native `FILTER`):** <https://www.sqlite.org/lang_aggfunc.html>
