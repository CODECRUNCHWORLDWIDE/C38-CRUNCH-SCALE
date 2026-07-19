# Lecture 3 — Survival and Stickiness

> **Duration:** ~2 hours. **Outcome:** You can build a survival curve that correctly handles cohorts too young to have "failed" yet, and compute a DAU/MAU stickiness ratio — the two numbers that answer "is this a durable growth engine or a leaky bucket, long-run?"

## 1. From retention curve to survival curve

Lecture 1 built a retention *triangle* — one row per cohort. A **survival curve** takes the same idea and asks a slightly different, more forward-looking question: **across all the cohorts you have, what fraction of users are still active N periods after they signed up?** Where the triangle shows you cohort-by-cohort detail, the survival curve blends cohorts together into a single curve you can act on — "by month 3, we typically retain about 17% of a cohort" — which is the number you'd plug into an LTV model in Week 5.

The catch is real: you *can't* just average every cohort's month-3 cell, because not every cohort has a month-3 cell yet.

## 2. The censoring problem

In our seed data, the May 2025 cohort has only reached month 1 as of the observation cutoff (2025-06-30). It has no month-2, month-3, or month-4 data — not because those users churned, but because **time hasn't passed yet**. This is called **right-censoring**, borrowed directly from survival analysis in medicine and reliability engineering, where a patient who's still alive when the study ends is "censored," not counted as having "survived exactly this long" or excluded as if they didn't exist.

Get this wrong and you get it wrong in a specific, seductive way: if you compute month-4 retention as "active at month 4, divided by *all* signups," you're dividing by cohorts that were never eligible to reach month 4 in the first place, and the percentage silently collapses toward zero as your denominator fills up with cohorts too young to have had a chance. The fix is the same one from Lecture 1's triangle, made explicit: **only include a cohort in a given month-number's denominator if that cohort has actually had that much time to pass.**

```sql
WITH cohort_max_m AS (
    -- how many calendar months of history each cohort has, as of the cutoff
    SELECT cohort_month,
        (CAST(strftime('%Y','2025-06-30') AS INT) - CAST(strftime('%Y', cohort_month||'-01') AS INT)) * 12
      + (CAST(strftime('%m','2025-06-30') AS INT) - CAST(strftime('%m', cohort_month||'-01') AS INT)) AS max_m
    FROM (SELECT DISTINCT cohort_month FROM users)
),
activity AS (
    SELECT u.user_id, u.cohort_month,
        (CAST(strftime('%Y', e.event_date) AS INT) - CAST(strftime('%Y', u.signup_date) AS INT)) * 12
      + (CAST(strftime('%m', e.event_date) AS INT) - CAST(strftime('%m', u.signup_date) AS INT)) AS month_number
    FROM users u JOIN events e ON e.user_id = u.user_id
),
m_axis(m) AS (VALUES (0),(1),(2),(3),(4)),
eligible AS (
    -- only count a (cohort, m) pair if that cohort has lived long enough to reach month m
    SELECT u.user_id, u.cohort_month, ma.m
    FROM users u
    CROSS JOIN m_axis ma
    JOIN cohort_max_m cm ON cm.cohort_month = u.cohort_month
    WHERE cm.max_m >= ma.m
)
SELECT
    e.m,
    COUNT(DISTINCT e.user_id)                                          AS eligible_users,
    COUNT(DISTINCT a.user_id)                                          AS active_users,
    ROUND(100.0 * COUNT(DISTINCT a.user_id) / COUNT(DISTINCT e.user_id), 1) AS survival_pct
FROM eligible e
LEFT JOIN activity a ON a.user_id = e.user_id AND a.month_number = e.m
GROUP BY e.m
ORDER BY e.m;
```

Against the seed data:

| month_number | eligible_users | active_users | survival_pct |
|---:|---:|---:|---:|
| 0 | 48 | 48 | 100.0% |
| 1 | 48 | 21 | 43.8% |
| 2 | 36 | 12 | 33.3% |
| 3 | 24 | 4 | 16.7% |
| 4 | 12 | 3 | 25.0% |

Read the `eligible_users` column carefully — it *shrinks* as `month_number` grows (48 → 48 → 36 → 24 → 12), because fewer and fewer cohorts have lived long enough to contribute a month-4 data point. By month 4, only the Feb cohort (n=12) is eligible at all, which is exactly why that last point is the noisiest and least trustworthy on the whole curve — a single 12-person cohort, not a blend. **Always report (or at least know) the eligible-users count alongside a survival percentage; a percentage with no denominator is not evidence of anything.**

**A note on the word "survival."** True survival analysis (the Kaplan-Meier estimator, familiar from clinical trials) models an *absorbing* event — once a patient dies, they don't come back. Product retention isn't absorbing: a churned user can resurrect, as you saw in Lecture 2. What we're building here is the growth-analytics convention that borrows the *name* and the *censoring discipline* from survival analysis, but not the assumption of irreversibility — it's really "the eligible-adjusted retention curve," just called a survival curve by habit across the industry. Know the difference so you don't overclaim rigor you don't have.

## 3. DAU/MAU — the stickiness ratio

A survival curve tells you whether users are *still around* months later. It doesn't tell you how *often* they show up while they're around. A user who logs in once every 25 days is "retained" in a monthly triangle exactly the same as a user who logs in every single day — but those are very different products to be building. **Stickiness** measures usage *frequency*, not just usage *presence*:

```
stickiness = average daily active users (DAU) / monthly active users (MAU)
```

If stickiness is 100%, every MAU shows up every single day — maximally habitual. If it's 3%, the average MAU shows up about once a month — barely a monthly errand. In SQL:

```sql
WITH daily AS (
    SELECT event_date, COUNT(DISTINCT user_id) AS dau
    FROM events
    WHERE event_date LIKE '2025-06%'
    GROUP BY event_date
),
mau AS (
    SELECT COUNT(DISTINCT user_id) AS mau
    FROM events
    WHERE event_date LIKE '2025-06%'
)
SELECT
    (SELECT mau FROM mau)                                          AS mau,
    ROUND((SELECT SUM(dau) FROM daily) / 30.0, 2)                  AS avg_dau,
    ROUND(100.0 * ((SELECT SUM(dau) FROM daily) / 30.0)
                / (SELECT mau FROM mau), 1)                        AS stickiness_pct;
```

(**PostgreSQL:** use `EXTRACT(DAY FROM date_trunc('month', d) + interval '1 month' - interval '1 day')` or simply hardcode the days-in-month like above — either is fine; don't over-engineer a denominator that's a known constant for a given month.)

Against June 2025 for the whole product: **MAU = 19, average DAU = 2.6, stickiness ≈ 13.7%.** That means the typical monthly active user opens the product roughly **1 day out of every 7** — a weekly-cadence product, not a daily habit. Across the three months we have full data for, stickiness is remarkably flat: **13.7% (April) → 14.1% (May) → 13.7% (June).** That stability itself is a finding — usage frequency isn't drifting, for better or worse, even while total MAU is growing.

### Segment it, the same way you segment retention

Stickiness by segment tells a much sharper story than the blended number:

```sql
WITH daily AS (
    SELECT u.segment, e.event_date, COUNT(DISTINCT e.user_id) AS dau
    FROM events e JOIN users u ON u.user_id = e.user_id
    WHERE e.event_date LIKE '2025-06%'
    GROUP BY u.segment, e.event_date
),
mau AS (
    SELECT u.segment, COUNT(DISTINCT e.user_id) AS mau
    FROM events e JOIN users u ON u.user_id = e.user_id
    WHERE e.event_date LIKE '2025-06%'
    GROUP BY u.segment
)
SELECT m.segment, m.mau,
       ROUND(SUM(d.dau) / 30.0, 2)                        AS avg_dau,
       ROUND(100.0 * (SUM(d.dau) / 30.0) / m.mau, 1)       AS stickiness_pct
FROM mau m
JOIN daily d ON d.segment = m.segment
GROUP BY m.segment, m.mau;
```

| segment | MAU | avg DAU | stickiness |
|---|---:|---:|---:|
| self_serve | 10 | 0.93 | 9.3% |
| sales_assisted | 9 | 1.67 | 18.5% |

`sales_assisted` users are exactly **twice** as sticky as `self_serve` — consistent with what you already saw in the survival/segment retention numbers (Lecture 2's classification and this lecture's `eligible`-adjusted table both show `sales_assisted` retaining far better at every month number). Stickiness and retention are telling the same underlying story from two different angles: presence over time (retention/survival) versus frequency while present (stickiness). When both point the same direction, as they do here, you have real signal, not a coincidence in one metric.

## 4. Reading the two curves together

A durable growth engine and a leaky bucket can sometimes look similar on *one* of these curves and diverge sharply on the other:

- **High survival, low stickiness** — users don't churn, but they barely use the product when they're around. Common in "system of record" tools (a CRM some sales reps only open during a deal review). Growth risk is low, but there's little room to sell more usage-based value without a habit-forming push.
- **Low survival, high stickiness (rare, but real)** — the users who stay are extremely engaged, but most people never make it into that group. The product likely has a narrow-but-passionate fit; the fix is usually activation and onboarding (Week 3 territory), not the core product.
- **Low survival, low stickiness** — the leaky bucket, textbook. Neither the presence curve nor the frequency curve gives you a floor to stand on. This is the profile Challenge 1 asks you to diagnose.
- **High survival, high stickiness** — the durable engine. Rare, valuable, and exactly what every growth team is trying to build toward.

## 5. Check yourself

- Why is it wrong to compute month-4 retention as "active at month 4 ÷ all 48 signups," given the seed data?
- What does "eligible_users" mean in the survival query, and why does it shrink as `month_number` increases?
- In what specific way does product retention violate the assumption behind classic Kaplan-Meier survival analysis?
- A product has 90% six-month retention but 4% stickiness. Describe, in one sentence, what kind of product this probably is.
- Two segments have identical unbounded month-1 retention but very different stickiness. What does that tell you that the retention number alone didn't?

## Further reading

- **Wikipedia — Survival analysis (overview of censoring):** <https://en.wikipedia.org/wiki/Survival_analysis>
- **Wikipedia — Kaplan–Meier estimator:** <https://en.wikipedia.org/wiki/Kaplan%E2%80%93Meier_estimator>
- **PostgreSQL — Aggregate Functions (`COUNT DISTINCT`, window aggregates):** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **PostgreSQL — `date_trunc` and interval arithmetic:** <https://www.postgresql.org/docs/current/functions-datetime.html>
