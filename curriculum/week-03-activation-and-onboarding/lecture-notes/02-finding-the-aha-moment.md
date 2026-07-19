# Lecture 2 — Finding the Aha Moment

> **Duration:** ~2 hours. **Outcome:** You can build a retention-lift table in SQL that ranks candidate activation events by how well they predict long-run retention, and you can name the three traps (reach, reverse causation, and window leakage) that turn a good-looking lift number into a bad decision.

The "aha moment" is growth folklore for a real, measurable thing: the specific early behavior that most cleanly separates users who stick around from users who don't. Facebook's early growth team is widely credited with framing theirs as "7 friends in 10 days"; Twitter's as "following 30 accounts." Whether or not those exact numbers are gospel, the *method* behind them is exactly reproducible, and this lecture is that method.

## 1. Define retention first — you need a yardstick

You cannot ask "what predicts retention?" without first defining retention. This week we use a simple, deliberately blunt definition so the mechanics stay clear (Week 4 builds the full cohort-retention machinery):

> **`week4_active`** — a user fired at least one `app_open` event in the 7-day window starting 21 days after their signup (i.e., during "week 4" of their lifecycle, counting the signup week as week 1).

```sql
SELECT
    u.user_id,
    EXISTS (
        SELECT 1 FROM events e
        WHERE e.user_id = u.user_id
          AND e.event_name = 'app_open'
          AND e.event_at >= u.signup_at + INTERVAL '21 days'
          AND e.event_at <  u.signup_at + INTERVAL '28 days'
    ) AS week4_active
FROM users u;
```

*(SQLite: replace `u.signup_at + INTERVAL '21 days'` with `datetime(u.signup_at, '+21 days')`, same pattern for `'+28 days'`.)*

Run `SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE week4_active) / COUNT(*), 1) FROM (...) t;` over that and you get Crunch Boards' baseline: **29.8%** of all signups are still opening the app in week 4. That number alone is your north-star-adjacent baseline — every candidate event gets judged against it.

## 2. Why week 4, not week 1 or week 12?

The retention window you pick to define the "yardstick" outcome isn't arbitrary, and it's worth justifying before you build anything on top of it.

- **Too early (e.g. `week1_active`)** measures whether a user *finished onboarding*, which is almost the same information as the activation flags themselves — you'd mostly be correlating an early event with another early event, not with durable behavior. The lift table would tell you little you didn't already know.
- **Too late (e.g. `week12_active`)** is a better test of durable retention, but by week 12 so many other factors have piled up (pricing changes, seasonal usage, feature releases in between) that isolating "did this one early action matter" gets noisy, and you have to wait three months to know if an onboarding fix worked.
- **Week 4** is a common middle ground in SaaS-style products: far enough out that "week-1 novelty" usage has settled and you're seeing genuine habitual behavior, close enough that you get a read in about a month, not a quarter. Different products choose differently (a daily habit app might use week 1; an enterprise tool with monthly billing cycles might use week 8) — the principle, not the exact number 4, is what transfers.

This week's data supports checking the sensitivity of your conclusions to this choice — Exercise materials and the mini-project stick to week 4 throughout so results are comparable, but it's worth remembering the choice was a judgment call, not a law of nature.

## 3. Build one flag row per user

The core trick of aha-moment analysis is a **user-grain table**: one row per user, with a boolean column for each candidate early action, plus the retention outcome. Once you have that table, "which action predicts retention" becomes a `GROUP BY` away.

```sql
WITH flags AS (
    SELECT
        u.user_id,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'verified_email')        AS verified,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'created_first_board')   AS created_board,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'created_first_card')    AS created_card,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'invited_teammate')      AS invited,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'completed_first_task')  AS completed_task,
        EXISTS (SELECT 1 FROM events e WHERE e.user_id = u.user_id AND e.event_name = 'connected_integration') AS connected,
        EXISTS (
            SELECT 1 FROM events e
            WHERE e.user_id = u.user_id AND e.event_name = 'app_open'
              AND e.event_at >= u.signup_at + INTERVAL '21 days'
              AND e.event_at <  u.signup_at + INTERVAL '28 days'
        ) AS week4_active
    FROM users u
)
SELECT * FROM flags;
```

Every downstream question this lecture (and this week's exercises) asks is now just an aggregate over `flags`.

## 4. The lift table

For each candidate action, compute: how many users did it (reach), the week-4 retention rate *among those who did it*, the week-4 retention rate *among those who didn't*, and the **lift** — the percentage-point gap between the two.

```sql
SELECT
    'invited_teammate' AS candidate,
    COUNT(*) FILTER (WHERE invited)                                                   AS n_did,
    ROUND(100.0 * COUNT(*) FILTER (WHERE invited AND week4_active)
                / NULLIF(COUNT(*) FILTER (WHERE invited), 0), 1)                        AS retained_if_did,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT invited AND week4_active)
                / NULLIF(COUNT(*) FILTER (WHERE NOT invited), 0), 1)                    AS retained_if_not,
    ROUND(
        100.0 * COUNT(*) FILTER (WHERE invited AND week4_active)     / NULLIF(COUNT(*) FILTER (WHERE invited), 0) -
        100.0 * COUNT(*) FILTER (WHERE NOT invited AND week4_active) / NULLIF(COUNT(*) FILTER (WHERE NOT invited), 0)
    , 1) AS lift_pp
FROM flags;
```

Repeating this pattern for every candidate (Exercise/Challenge material — don't hand-run six near-identical blocks in the lecture) produces this table against the seed data:

| candidate | n_did | pct reach | retained_if_did | retained_if_not | lift (pp) |
|---|---:|---:|---:|---:|---:|
| verified_email | 322 | 80.5% | 34.8% | 9.0% | +25.8 |
| created_first_board | 256 | 64.0% | 39.1% | 13.2% | +25.9 |
| created_first_card | 209 | 52.2% | 45.5% | 12.6% | +32.9 |
| **invited_teammate** | **116** | **29.0%** | **65.5%** | **15.1%** | **+50.4** |
| completed_first_task | 101 | 25.2% | 65.3% | 17.7% | +47.6 |
| connected_integration | 37 | 9.2% | 48.6% | 27.8% | +20.8 |

`invited_teammate` jumps out: the biggest lift of any candidate (+50.4pp — inviters retain at 65.5% vs. 15.1% for non-inviters, more than 4×), *and* it still has a workable 29% reach — nowhere near as rare as `connected_integration`. That combination (real lift + real reach) is what an aha moment actually looks like in a lift table. This is the empirical result Lecture 1's intuition ("collaboration is the point of a Kanban tool") predicted — the SQL confirms it rather than assumes it.

## 5. Three traps that make a lift table lie to you

A big lift number is not automatically a decision. Check for these before you commit.

### Trap 1 — reach

`connected_integration` has a real lift (+20.8pp) but only 9.2% reach. You already met this trap in Lecture 1: a metric almost nobody can hit isn't a lever for the whole funnel, however cleanly it correlates. Treat lift and reach as two axes, not one — plot them mentally, and prefer candidates in the "high lift, workable reach" quadrant.

### Trap 2 — reverse causation / endogeneity

`completed_first_task` has almost the same lift as `invited_teammate` (+47.6pp vs. +50.4pp). Tempting to call it a tie. But look at *when* it happens: in this data, `completed_first_task` only fires **after** `invited_teammate` for the overwhelming majority of users who reach it (101 of the 116 inviters go on to complete a task — check it: `SELECT COUNT(*) FROM flags WHERE completed_task AND invited;`). That ordering matters. A user who has already invited a teammate and is completing tasks with them is already deep into being an engaged, retained user — "completing a task" isn't *causing* their retention so much as *confirming* it. This is **reverse causation**: the metric looks predictive because it's a late symptom of the same underlying engagement, not an early lever you can pull. Prefer the *earliest* event in a causal chain that still carries strong lift, not the latest one that merely correlates.

### Trap 3 — window leakage (using future information)

Every flag in `flags` above was computed with `EXISTS`, over the *entire* event history — no time bound. That's fine for this teaching example because we're explaining the mechanics, but it's a real bug waiting to happen in production: if you don't cap "did the user invite a teammate" to, say, the first 7 days after signup, a user who invited someone on day 90 still counts as "did it" — even though on day 3, when you'd actually want to nudge them, you had no idea yet. An activation event has to be knowable *early* to be actionable early. Exercise 1 has you add this cap explicitly and see how much the reach numbers shrink once you do.

## 6. Choosing the winner, and being honest about what you've proven

Given the table above: `invited_teammate` is the strongest candidate — highest lift among events with meaningful reach, earliest in the causal chain among the high-lift options, and it's actionable (you can literally build an "invite your team" prompt into onboarding).

But say this precisely, because it matters: **correlation from a lift table is a hypothesis, not proof.** Inviters might retain better *because* collaboration is genuinely sticky — or inviters might just be a different, more serious kind of user to begin with (more established teams, more urgent use case) who would have retained anyway, invite or not. The lift table tells you where to *look*; only a controlled experiment — randomizing who gets an invite nudge and comparing outcomes — tells you whether the invite itself *causes* the retention gain. That's Week 8 (Experimentation & A/B Testing). This week's job is to generate the hypothesis with rigor; don't oversell it as more than that.

## 7. A note on what a lift table can't do for a brand-new product

Everything above assumes you already have weeks of historical events to mine — 400 signups' worth of downstream behavior to correlate against. Early-stage products (or new features) don't have that luxury: with 20 signups and no week-4 data yet, a lift table is statistical noise, not signal. Two honest fallbacks for that situation, both worth knowing even though this week's exercises use the full seed:

- **Borrow from a comparable product or feature** you already have retention data for, as a starting hypothesis, and treat it as provisional until your own data catches up.
- **Use a qualitative signal instead** — the Superhuman survey method from this lecture's further reading (asking users "how would you feel if you could no longer use this product?" and segmenting by "very disappointed") doesn't need weeks of retention data at all, because it asks users to self-report the counterfactual directly. It's a different kind of evidence — sentiment, not behavior — but it's genuinely useful when the event log is still too thin to mine.

Neither replaces the SQL-based lift table once you have enough data — but knowing when you *don't* yet have enough is part of doing this analysis honestly.

## 8. Check yourself

- Why do you need a retention definition (`week4_active`) *before* you can build a lift table at all?
- Why week 4 specifically, and what tradeoff would you be making if you used week 1 or week 12 instead?
- Walk through the `flags` CTE — why is one row per user, with boolean columns, the right shape for this analysis?
- `completed_first_task` and `invited_teammate` have similar lift. Which would you pick as the activation event, and why — name the trap the other one falls into.
- What does "window leakage" mean in this context, and why does it make an activation event look more useful than it really is?
- Why is a lift table evidence for a hypothesis rather than proof of causation? What kind of study would upgrade it to proof?
- If `connected_integration` had 9.2% reach but a lift of +80pp instead of +20.8pp, would your recommendation change? Why or why not?
- Your team ships a new feature with 20 signups and no week-4 data yet. Why would running this week's lift-table method on that data right now be a mistake, and what would you do instead?

Lecture 3 takes the activation event you've now chosen and asks the operational question: where, specifically, in the sequence of steps leading up to it, are users falling out?

## Further reading

- **Reforge — "Finding Your Product's Aha Moment"** (search their public blog index): <https://www.reforge.com/blog>
- **Andrew Chen — writing on activation and retention**: <https://andrewchen.com/>
- **PostgreSQL — Aggregate expressions (`FILTER`, `COUNT`)**: <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
- **PostgreSQL — Subquery expressions (`EXISTS`)**: <https://www.postgresql.org/docs/current/functions-subquery.html>
- **"Correlation is not causation" (a level-headed primer)**: <https://en.wikipedia.org/wiki/Correlation_does_not_imply_causation>
