# Lecture 1 — Cohorts and Retention Curves

> **Duration:** ~2 hours. **Outcome:** You can group users into signup cohorts, build the full retention triangle against the seed `users`/`events` tables in one query, and read whether the resulting curve is smiling, flattening, or heading to zero.

## 1. Why "retention rate" alone is a bad number

Ask a founder "what's your retention?" and you'll often get one number — "we're at 35%." That number is close to meaningless until you answer three follow-up questions: **35% of whom**, retained **doing what**, **over what window**? A single blended retention rate mixes users who signed up yesterday with users who signed up a year ago, mixes a power user who opens the product daily with someone who logged in once for a free trial, and mixes "did anything at all" with "did the thing that actually matters." It's a number you can't act on.

**Cohort analysis** fixes this by refusing to blend. A **cohort** is a group of users who share a start event — almost always the signup date, bucketed to a period (day, week, month). You then track *that specific group, and only that group*, forward in time. Comparing the Feb 2025 cohort's month-1 retention to the May 2025 cohort's month-1 retention tells you something a blended number never can: did the product get stickier between February and May?

## 2. The shape of the data you need

Two tables, and only two, are enough to answer almost every retention question a company has:

```sql
-- who signed up, and when
users (user_id, signup_date, cohort_month, segment, channel, country)

-- what they did, and when
events (event_id, user_id, event_date, event_type)
```

This is deliberately the same shape as the acquisition and activation instrumentation from Weeks 1–3 — one row per user, one row per meaningful action. Retention analysis doesn't need new instrumentation; it needs a new *query* over data you already have. That's the whole reason event logging pays for itself: you don't know in Week 1 which questions you'll ask in Week 4, so you log the raw fact (`user X did something on date Y`) and derive every metric — funnels, activation, retention, LTV — from the same source of truth. This is also why the rule for this course is **SQL and pandas, never spreadsheets**: a spreadsheet can hold a retention *report*, but it can't hold the raw event log a report like this is derived from and stay auditable when the definition inevitably changes.

## 3. The cohort key: "months since signup," not calendar month

The first move, and the one beginners get wrong, is picking the right x-axis. If you plot retention by *calendar month* (Feb, Mar, Apr, …), you're comparing users who've had different amounts of time with the product — the May cohort simply hasn't had a chance to reach month 3 yet. The fix is to re-express every event's timing **relative to that user's own signup**, not the calendar:

```sql
SELECT
    u.user_id,
    e.event_date,
    -- calendar months elapsed between signup and this event
    (CAST(strftime('%Y', e.event_date) AS INT) - CAST(strftime('%Y', u.signup_date) AS INT)) * 12
  + (CAST(strftime('%m', e.event_date) AS INT) - CAST(strftime('%m', u.signup_date) AS INT))
    AS month_number
FROM users u
JOIN events e ON e.user_id = u.user_id
WHERE u.user_id = 24
ORDER BY e.event_date;
```

`month_number = 0` means "this happened in the same calendar month the user signed up." `month_number = 1` means "one calendar month after signup," and so on. **Postgres note:** the same idea reads more naturally with `date_trunc` and `AGE`:

```sql
-- PostgreSQL equivalent
SELECT
    u.user_id, e.event_date,
    (DATE_PART('year', e.event_date) - DATE_PART('year', u.signup_date)) * 12
  + (DATE_PART('month', e.event_date) - DATE_PART('month', u.signup_date))
    AS month_number
FROM users u JOIN events e ON e.user_id = u.user_id;
```

This `month_number` is the axis every cell of the triangle sits on. It's the single idea that makes cohort analysis work: **anchor time to the individual, then aggregate**.

## 4. Building the retention triangle

A **retention triangle** (also called a cohort grid) puts cohorts down the rows and `month_number` across the columns. Each cell answers: *of the users who signed up in this cohort, what percent were active exactly `month_number` months later?*

```sql
WITH activity AS (
    SELECT
        u.user_id,
        u.cohort_month,
        (CAST(strftime('%Y', e.event_date) AS INT) - CAST(strftime('%Y', u.signup_date) AS INT)) * 12
      + (CAST(strftime('%m', e.event_date) AS INT) - CAST(strftime('%m', u.signup_date) AS INT))
        AS month_number
    FROM users u
    JOIN events e ON e.user_id = u.user_id
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(*) AS cohort_size
    FROM users
    GROUP BY cohort_month
)
SELECT
    a.cohort_month,
    cs.cohort_size,
    a.month_number,
    COUNT(DISTINCT a.user_id)                                       AS active_users,
    ROUND(100.0 * COUNT(DISTINCT a.user_id) / cs.cohort_size, 1)     AS pct_retained
FROM activity a
JOIN cohort_sizes cs ON cs.cohort_month = a.cohort_month
GROUP BY a.cohort_month, cs.cohort_size, a.month_number
ORDER BY a.cohort_month, a.month_number;
```

Run this against the seed dataset and you get exactly this (verified against both engines):

| cohort_month | cohort_size | month 0 | month 1 | month 2 | month 3 | month 4 |
|---|---:|---:|---:|---:|---:|---:|
| 2025-02 | 12 | 100.0% (12) | 16.7% (2) | 16.7% (2) | 8.3% (1) | 25.0% (3) |
| 2025-03 | 12 | 100.0% (12) | 41.7% (5) | 41.7% (5) | 25.0% (3) | — |
| 2025-04 | 12 | 100.0% (12) | 50.0% (6) | 41.7% (5) | — | — |
| 2025-05 | 12 | 100.0% (12) | 66.7% (8) | — | — | — |

A few things to notice immediately:

- **Month 0 is always 100%.** By construction, everyone in a cohort was active in their own signup month — that's *why* they're in the cohort. Month 0 tells you nothing about retention; the story starts at month 1.
- **The bottom-right is empty, not zero.** The May cohort has no month-2 cell because May 2025 to the observation cutoff (2025-06-30) is only two calendar months. An empty cell means "not observed yet," and a 0% cell means "observed and nobody came back." Confusing the two is one of the most common cohort-analysis mistakes — it silently drags your averages down with cohorts that simply haven't had time to churn *or* stick yet. We'll come back to this as **right-censoring** in Lecture 3.
- **This is why it's a triangle.** Older cohorts (Feb) have filled in more columns than younger cohorts (May) simply because more time has passed for them. A cohort grid is never a rectangle unless the product has been dead — no new signups — for as long as your oldest cohort's history.

## 5. Reading the shape

Once the triangle is built, the interesting work is reading each *row* as a curve and asking what shape it traces. Three shapes come up constantly:

**Terminal (heading to zero).** Retention keeps dropping every period with no sign of leveling off. Eventually it hits 0%. This is the shape of a novelty product, a one-time-use tool, or a funnel with a leak nobody's found yet. If you can't point to a floor the curve is settling toward, assume it's terminal until proven otherwise.

**Flattening (the good shape, most of the time).** Retention drops sharply in the first period or two, then levels off into a stable floor — a core group of users who keep coming back indefinitely. Almost every healthy subscription or habit product looks like this: steep initial drop, then a long flat tail. The floor, not the drop, is what you're selling to investors and to yourself. Look at the Feb cohort above: 100% → 16.7% → 16.7% → 8.3% → 25.0%. The middle two months (16.7%, 16.7%) look like a floor forming — encouraging — but month 3 dips to 8.3% and month 4 jumps back to 25.0%. With only 12 users in the cohort, each person is worth 8.3 percentage points; that's small-sample noise, not necessarily a trend reversal. **Never trust a single cohort's tail; look at where several cohorts' curves are heading.**

**Smiling (rare, and worth noticing when it happens).** Retention dips and then *rises* — later-period retention higher than early-period. This shows up when a product has delayed value: think of a tool people forget about for a few months and then rediscover during a specific recurring need (tax software, a seasonal planning tool), or a network product where a user's early friends weren't active yet but are now. A smiling curve is a strong signal the product's value compounds with time, not just habit.

## 6. Comparing cohorts to see if the product is improving

The real payoff of a triangle isn't one row — it's comparing rows. Look at month-1 retention across our four cohorts: **16.7% → 41.7% → 50.0% → 66.7%.** That's not noise; that's a trend, and it's a steep one. Something changed between the Feb/Mar cohorts and the Apr/May cohorts that made new users dramatically more likely to come back a month later.

This is the single most useful move in retention analysis: **hold `month_number` fixed and compare across cohorts.** It isolates "did we get better at retaining people" from "how long has this cohort existed." A rising diagonal-adjusted column (same month-number, later cohorts higher) is one of the cleanest signals in growth analytics that a specific change — an onboarding fix, a new core feature, better activation — is working. A flat or falling column means whatever you shipped didn't move the needle, no matter how good it looked in a demo.

```sql
-- isolate one column of the triangle to compare cohorts directly
SELECT cohort_month, active_users, cohort_size,
       ROUND(100.0 * active_users / cohort_size, 1) AS month1_retention_pct
FROM ( /* the triangle query above, filtered to month_number = 1 */ ) t
WHERE month_number = 1
ORDER BY cohort_month;
```

## 7. Common mistakes that quietly wreck a triangle

**Counting `month_number = 0` as evidence of retention.** It's tempting to average "month 0 through month 2" and report the blend — don't. Averaging in the guaranteed-100% month-0 column inflates every headline number and hides exactly the drop-off you're trying to measure. Report month 0 as a sanity check (it should always equal cohort size), never as a metric.

**Dividing by the wrong denominator.** `cohort_size` in the triangle query above must be the *original* signup count for that cohort, not the number of users who happened to have any events at all. If you accidentally `JOIN` `users` to `events` with an inner join before computing cohort size, users with zero events silently vanish from the denominator too — and every percentage in that row comes out too high, because you've quietly excluded exactly the people who churned hardest.

**Treating an empty cell as a zero in downstream math.** If you're computing an average retention rate across cohorts and your query does a `LEFT JOIN` that turns missing cells into `NULL`, make sure any aggregate you build (`AVG`, `SUM`) either excludes `NULL`s (the default SQL behavior — safe) or, if you've coalesced them to `0` for display, that you haven't also fed that `0` back into a further calculation. A censored cell coalesced to `0` for a nicer-looking dashboard is a landmine for whoever queries it next.

**Confusing "cohort" with "segment."** A cohort is a *time* bucket (when someone signed up). A segment (self-serve vs. sales-assisted, free vs. paid, channel) is a *type* bucket. They compose — you can and often should build a triangle *per segment* — but conflating them (e.g., calling `self_serve` a "cohort") makes it hard to talk precisely about which axis you're slicing on. Challenge 2 this week asks you to build exactly this composition deliberately.

## 8. Check yourself

- Why does every cohort's month-0 cell always read 100%, and why does that make it useless for judging retention?
- What's the difference between an empty cell in the triangle and a cell that reads 0%?
- A cohort's curve reads 100% → 60% → 35% → 20% → 19% → 20%. What shape is this, and what would you tell a stakeholder about the business?
- Why is comparing the *same column* (same `month_number`) across different cohort rows more informative than comparing different columns within one row?
- The May cohort in our seed data only has a month-0 and month-1 column. Why — and what would you need to wait for before you could compare its month-2 retention to Feb's?

If those are automatic, Lecture 2 gets precise about what "retention" even means — N-day, unbounded, and rolling are three different metrics that all get called "retention," and mixing them up is how two honest analysts land on two different numbers for the same product.

## Further reading

- **PostgreSQL — Date/Time Functions and Operators:** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **SQLite — Date and Time Functions:** <https://www.sqlite.org/lang_datefunc.html>
- **PostgreSQL — `date_trunc`:** <https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-TRUNC>
