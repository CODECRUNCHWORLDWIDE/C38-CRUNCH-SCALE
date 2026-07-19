# Lecture 3 — Diagnosing the Onboarding Funnel

> **Duration:** ~2 hours. **Outcome:** You can turn an onboarding sequence into an ordered step funnel in SQL, compute step-over-step conversion, locate the single worst drop, measure time-to-value as a full distribution, and connect both findings into one diagnosis.

Lecture 2 gave you the destination: `invited_teammate` is Crunch Boards' activation event. This lecture is about the road to it — where, specifically, do users fall off between signing up and getting there?

## 1. Onboarding as an ordered step funnel

A **step funnel** takes a sequence of events that (mostly) happen in order and reports, for each step, how many users reached it and what fraction of the *previous* step's users made it there. Crunch Boards' onboarding sequence:

```
signed_up → verified_email → created_first_board → created_first_card → invited_teammate → completed_first_task
```

This is different from the reach table in Lecture 1 in one important way: reach compares everyone back to the *original* denominator (400 signups). A funnel compares each step to the *step before it* — that's what exposes where the sharpest single drop is, which raw reach numbers can hide.

## 2. Computing funnel counts

Reuse the reach-style query from Lecture 1, restricted to the six ordered steps:

```sql
SELECT
    event_name,
    COUNT(DISTINCT user_id) AS users_reached
FROM events
WHERE event_name IN (
    'signed_up','verified_email','created_first_board',
    'created_first_card','invited_teammate','completed_first_task'
)
GROUP BY event_name;
```

Against the seed, ordered by the funnel sequence (not alphabetically — order the rows yourself, either with a `CASE WHEN ... END` sort key or in application code):

| step | users_reached | % of signups |
|---|---:|---:|
| 1. signed_up | 400 | 100.0% |
| 2. verified_email | 322 | 80.5% |
| 3. created_first_board | 256 | 64.0% |
| 4. created_first_card | 209 | 52.2% |
| 5. invited_teammate | 116 | 29.0% |
| 6. completed_first_task | 101 | 25.2% |

## 3. Step-over-step conversion — the number that actually matters

"% of signups" tells you cumulative loss. It does **not** tell you which single step is the worst offender, because early losses compound. You need step-over-step conversion: `this step's users ÷ previous step's users`.

```sql
WITH counts AS (
    SELECT 1 AS step_order, 'signed_up' AS step, COUNT(DISTINCT user_id) AS n FROM events WHERE event_name = 'signed_up'
    UNION ALL
    SELECT 2, 'verified_email', COUNT(DISTINCT user_id) FROM events WHERE event_name = 'verified_email'
    UNION ALL
    SELECT 3, 'created_first_board', COUNT(DISTINCT user_id) FROM events WHERE event_name = 'created_first_board'
    UNION ALL
    SELECT 4, 'created_first_card', COUNT(DISTINCT user_id) FROM events WHERE event_name = 'created_first_card'
    UNION ALL
    SELECT 5, 'invited_teammate', COUNT(DISTINCT user_id) FROM events WHERE event_name = 'invited_teammate'
    UNION ALL
    SELECT 6, 'completed_first_task', COUNT(DISTINCT user_id) FROM events WHERE event_name = 'completed_first_task'
)
SELECT
    step_order,
    step,
    n,
    LAG(n) OVER (ORDER BY step_order)                                              AS prev_n,
    ROUND(100.0 * n / NULLIF(LAG(n) OVER (ORDER BY step_order), 0), 1)             AS pct_converted,
    ROUND(100.0 - 100.0 * n / NULLIF(LAG(n) OVER (ORDER BY step_order), 0), 1)     AS pct_dropped
FROM counts
ORDER BY step_order;
```

`LAG()` is the window function doing the real work here — it pulls the previous row's `n` into the current row without a self-join. Result:

| step | n | prev_n | pct_converted | pct_dropped |
|---|---:|---:|---:|---:|
| signed_up | 400 | — | — | — |
| verified_email | 322 | 400 | 80.5% | 19.5% |
| created_first_board | 256 | 322 | 79.5% | 20.5% |
| created_first_card | 209 | 256 | 81.6% | 18.4% |
| **invited_teammate** | **116** | **209** | **55.5%** | **44.5%** |
| completed_first_task | 101 | 116 | 87.1% | 12.9% |

Look at the shape: the first three transitions all convert around 80% (losing roughly a fifth of users at each step is normal onboarding friction). Then `created_first_card → invited_teammate` collapses to **55.5% conversion — a 44.5% drop**, by far the worst single transition in the funnel, more than double every other step's loss rate. That's not noise; that's a real, addressable stall, and it's exactly the step leading up to the activation event Lecture 2 identified. Diagnosis and activation-event choice have converged on the same place — which is a good sign you've found something real rather than an artifact of one analysis.

## 4. Time-to-value — the distribution, not the average

**Time-to-value (TTV)** is the elapsed time between signup and the activation event, for users who reach it. Reporting only the *average* TTV is a classic mistake — it hides the shape, and time distributions are almost always right-skewed (a few very slow stragglers drag the mean up past where most users actually land).

```sql
SELECT
    u.user_id,
    EXTRACT(EPOCH FROM (e.event_at - u.signup_at)) / 3600.0 AS ttv_hours
FROM users u
JOIN events e
  ON e.user_id = u.user_id
 AND e.event_name = 'invited_teammate'
ORDER BY ttv_hours;
```

*(SQLite has no `EXTRACT`; compute hours as `(julianday(e.event_at) - julianday(u.signup_at)) * 24`.)*

**PostgreSQL** can compute percentiles directly with `PERCENTILE_CONT`:

```sql
SELECT
    ROUND(MIN(ttv_hours)::numeric, 1)                                       AS min_h,
    ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY ttv_hours)::numeric, 1) AS p25_h,
    ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY ttv_hours)::numeric, 1) AS median_h,
    ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY ttv_hours)::numeric, 1) AS p75_h,
    ROUND(MAX(ttv_hours)::numeric, 1)                                       AS max_h,
    ROUND(AVG(ttv_hours)::numeric, 1)                                       AS mean_h
FROM (
    SELECT EXTRACT(EPOCH FROM (e.event_at - u.signup_at)) / 3600.0 AS ttv_hours
    FROM users u JOIN events e ON e.user_id = u.user_id AND e.event_name = 'invited_teammate'
) t;
```

**SQLite has no `PERCENTILE_CONT`.** The portable fallback is to sort the values and pick by row position (`OFFSET`), which is an *approximate* percentile (no interpolation between the two nearest ranks) but is exactly right for course purposes:

```sql
-- median of 116 rows: the (116-1)*0.5 = 57.5th position -> average rows at offset 57 and 58 (0-indexed)
SELECT ttv_hours FROM (
    SELECT (julianday(e.event_at) - julianday(u.signup_at)) * 24 AS ttv_hours
    FROM users u JOIN events e ON e.user_id = u.user_id AND e.event_name = 'invited_teammate'
    ORDER BY ttv_hours
) ORDER BY ttv_hours LIMIT 2 OFFSET 57;   -- average these two for the median
```

Against the seed (n = 116 inviters):

| stat | hours | ≈ days |
|---|---:|---:|
| min | 21.1 | 0.9 |
| p25 | 80.0 | 3.3 |
| median | 112.3 | 4.7 |
| p75 | 133.0 | 5.5 |
| max | 185.9 | 7.7 |
| mean | 108.1 | 4.5 |

And bucketed into ranges (a query using `CASE WHEN` on the hours, `GROUP BY` the bucket label):

| bucket | users |
|---|---:|
| under 24h | 2 |
| 1–3 days | 17 |
| 3–7 days | 88 |
| over 7 days | 9 |

**Read the shape, not just the median.** Three-quarters of inviters take between roughly 3 and 5.5 days — this is not a same-day action for most users. That's a meaningful operational fact: if your success team's "day-1 nudge" playbook assumes users invite teammates on day one, it's aimed at the wrong day for 98% of this cohort (only 2 of 116 invited within 24 hours). The distribution tells you *when* to intervene, which the median alone would not.

## 5. Connecting the two findings into one diagnosis

Put §3 and §4 side by side:

- The funnel's worst drop is `created_first_card → invited_teammate` (44.5% lost).
- The median gap between `created_first_card` and `invited_teammate` specifically (not from signup) is about **50 hours (~2.1 days)** — you can compute this the same way as §4, just swapping the join to `created_first_card` as the start event instead of `signup_at`.
- Users create a board and a card **alone** — nothing in the current flow prompts them to invite anyone until they've already built something solo. By the time they might think to invite a teammate, they've spent two days using Crunch Boards as a single-player to-do list, which is not the product's intended value.

That's a diagnosis, in one sentence: **the onboarding flow lets users complete the "solo" half of setup with no prompt toward the "collaborative" half that actually drives retention, and the biggest funnel drop sits exactly at that seam.** This is the finding Challenge 2 and the mini-project build on — turning it into a proposed fix and a quantified (projected) lift.

## 6. Check yourself

- Why does step-over-step conversion (§3) reveal a problem that cumulative "% of signups" (§2) can hide?
- Which window function does the step-conversion query rely on, and what would you have to write instead without it?
- Why is reporting only the *mean* TTV a mistake? What does the Crunch Boards TTV distribution's shape tell you that the mean alone wouldn't?
- SQLite has no `PERCENTILE_CONT`. What's the portable fallback, and what precision do you lose by using it?
- In one sentence, state the onboarding diagnosis for Crunch Boards, tying together the funnel drop and the TTV finding.
- The `completed_first_task` step converts at 87.1% — the *best* single-step conversion in the whole funnel. Does that mean it's a low-priority step to think about? Why or why not?

You now have everything the mini-project needs: an activation event definition, a lift table, a funnel with its worst drop located, and a TTV distribution. The exercises turn each of these into hands-on reps; the challenges push you to defend and act on them.

## Further reading

- **PostgreSQL — Window Functions**: <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — Window Function reference (`LAG`, `PERCENTILE_CONT`)**: <https://www.postgresql.org/docs/current/functions-window.html>
- **SQLite — Window Functions**: <https://www.sqlite.org/windowfunctions.html>
- **Amplitude — "How to Build a Funnel Analysis"**: <https://amplitude.com/blog/funnel-analysis>
