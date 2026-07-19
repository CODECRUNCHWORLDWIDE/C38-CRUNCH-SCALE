# Week 1 — Growth Foundations & Metrics

> **Goal:** by Sunday you can name a company's north-star metric, show the input metrics that actually move it, map its funnel end to end, and prove every one of those numbers by querying raw events in PostgreSQL — never by trusting a dashboard or a spreadsheet.

Welcome to **C38 · Crunch Scale**. Growth work has a credibility problem: two people can look at the same business and report two different "growth rates" because they picked different metrics, different date windows, or — worst of all — because someone typed a number into a spreadsheet cell by hand and nobody can reproduce it. This week fixes that at the root. You'll pick one number a team should obsess over (the **north star**), decompose it into metrics a team can actually influence, map the whole customer journey with the **AARRR framework**, and then instrument and load a real **event schema into PostgreSQL** so every metric traces back to a row you can `SELECT`.

We work all week against one seed product: **StreakLab**, a fictional habit-tracking app with a free tier and a $9/month Pro tier. You'll load two tables — `users` and `events` — once (below), and every lecture, exercise, challenge, and the mini-project queries them. This same seed dataset is the backbone of the rest of C38: acquisition funnels (Week 2), activation (Week 3), retention cohorts (Week 4), LTV/CAC (Week 5), and beyond all reuse this schema, so build it carefully now.

## Learning objectives

By the end of this week, you will be able to:

- **Define** what a metric is, why a growth team needs exactly one guiding **north-star metric**, and how to derive the **input metrics** that a team can move week to week.
- **Distinguish** an actionable metric from a **vanity metric** — and explain why a number going "up and to the right" can still be worthless.
- **Map** the **AARRR funnel** (Acquisition, Activation, Retention, Revenue, Referral) onto a real product and identify where a specific product's real leverage sits.
- **Design** a minimal but honest **event schema** — a `users` dimension table and an `events` fact table — the same shape production analytics warehouses use (Segment, Amplitude, Snowplow, RudderStack all converge on this).
- **Load** that schema into **PostgreSQL** and query a full **metric tree** — north star plus its input metrics — straight from raw events, with zero spreadsheets anywhere in the pipeline.

## Prerequisites

- SQL fluency at the level of **[C33 Crunch SQL](../../../C33-CRUNCH-SQL/)** — comfortable with `SELECT`, `WHERE`, joins, `GROUP BY`/aggregates, and date functions. Week 1 leans hardest on joins and date-bucketed aggregation.
- PostgreSQL 16+ installed (SQLite 3.35+ works as a fallback — see [`resources.md`](./resources.md)). If you only install one engine, install **PostgreSQL**.
- No prior growth, marketing, or product-analytics background assumed.
- **Data tooling rule for this entire course:** whenever you need to store, model, or query data, you use **SQL and/or Python (pandas)**. Never a spreadsheet. If you catch yourself wanting to paste numbers into Excel to "just check something," that instinct is exactly what this course is retraining.

## Set up the seed database (do this first)

Everything this week runs against two tables: `users` (one row per person who signs up) and `events` (one row per thing a person *does*). This is the canonical shape of an analytics warehouse's raw layer — a **dimension table** (who) and a **fact table** (what happened, when).

**PostgreSQL:**

```bash
createdb crunch_scale          # make a database named "crunch_scale"
psql crunch_scale              # open a session against it
```

**SQLite (fallback):**

```bash
sqlite3 crunch_scale.db
```

Then paste this into the shell (unchanged on both engines):

```sql
CREATE TABLE users (
    user_id        INTEGER PRIMARY KEY,
    signup_at      TIMESTAMP NOT NULL,
    signup_channel TEXT      NOT NULL,   -- how they found StreakLab
    country        TEXT      NOT NULL
);

CREATE TABLE events (
    event_id     INTEGER PRIMARY KEY,
    user_id      INTEGER   NOT NULL REFERENCES users(user_id),
    event_name   TEXT      NOT NULL,     -- session_start | habit_created | checkin_logged
                                          -- | invite_sent | invite_accepted | subscription_started
    event_time   TIMESTAMP NOT NULL,
    event_value  NUMERIC                 -- only populated for subscription_started (monthly price, USD)
);
```

Now load 15 users:

```sql
INSERT INTO users (user_id, signup_at, signup_channel, country) VALUES
(1,'2026-06-01 07:30:00','paid_search','USA'),
(2,'2026-06-01 07:30:00','organic_search','Canada'),
(3,'2026-06-01 07:30:00','paid_social','UK'),
(4,'2026-06-01 07:30:00','social_organic','India'),
(5,'2026-06-01 07:30:00','referral','Brazil'),
(6,'2026-06-08 07:30:00','paid_search','USA'),
(7,'2026-06-08 07:30:00','organic_search','Canada'),
(8,'2026-06-08 07:30:00','paid_social','UK'),
(9,'2026-06-08 07:30:00','social_organic','India'),
(10,'2026-06-08 07:30:00','referral','Brazil'),
(11,'2026-06-15 07:30:00','paid_search','USA'),
(12,'2026-06-15 07:30:00','organic_search','Canada'),
(13,'2026-06-15 07:30:00','paid_social','UK'),
(14,'2026-06-15 07:30:00','social_organic','India'),
(15,'2026-06-15 07:30:00','referral','Brazil');
```

And 149 events (paste all four blocks in order — they share one sequence of `event_id`s):

```sql
INSERT INTO events (event_id, user_id, event_name, event_time, event_value) VALUES
(1,1,'session_start','2026-06-01 08:00:00',NULL),
(2,1,'habit_created','2026-06-01 08:05:00',NULL),
(3,1,'checkin_logged','2026-06-01 20:00:00',NULL),
(4,1,'checkin_logged','2026-06-02 20:00:00',NULL),
(5,1,'checkin_logged','2026-06-03 20:00:00',NULL),
(6,1,'invite_sent','2026-06-03 12:00:00',NULL),
(7,1,'checkin_logged','2026-06-04 20:00:00',NULL),
(8,1,'invite_accepted','2026-06-04 12:30:00',NULL),
(9,1,'subscription_started','2026-06-04 09:30:00',9.00),
(10,1,'checkin_logged','2026-06-05 20:00:00',NULL),
(11,1,'checkin_logged','2026-06-06 20:00:00',NULL),
(12,1,'checkin_logged','2026-06-07 20:00:00',NULL),
(13,1,'checkin_logged','2026-06-08 20:00:00',NULL),
(14,1,'checkin_logged','2026-06-09 20:00:00',NULL),
(15,1,'checkin_logged','2026-06-10 20:00:00',NULL),
(16,1,'checkin_logged','2026-06-12 20:00:00',NULL),
(17,1,'checkin_logged','2026-06-13 20:00:00',NULL),
(18,1,'checkin_logged','2026-06-14 20:00:00',NULL),
(19,1,'checkin_logged','2026-06-15 20:00:00',NULL),
(20,1,'checkin_logged','2026-06-16 20:00:00',NULL),
(21,1,'checkin_logged','2026-06-17 20:00:00',NULL),
(22,1,'checkin_logged','2026-06-19 20:00:00',NULL),
(23,1,'checkin_logged','2026-06-20 20:00:00',NULL),
(24,1,'checkin_logged','2026-06-21 20:00:00',NULL),
(25,2,'session_start','2026-06-01 08:00:00',NULL),
(26,2,'habit_created','2026-06-02 08:05:00',NULL),
(27,2,'checkin_logged','2026-06-02 20:00:00',NULL),
(28,2,'checkin_logged','2026-06-04 20:00:00',NULL),
(29,2,'checkin_logged','2026-06-06 20:00:00',NULL),
(30,2,'checkin_logged','2026-06-09 20:00:00',NULL),
(31,2,'checkin_logged','2026-06-11 20:00:00',NULL),
(32,2,'checkin_logged','2026-06-13 20:00:00',NULL),
(33,2,'checkin_logged','2026-06-16 20:00:00',NULL),
(34,2,'checkin_logged','2026-06-18 20:00:00',NULL),
(35,2,'checkin_logged','2026-06-20 20:00:00',NULL),
(36,3,'session_start','2026-06-01 08:00:00',NULL),
(37,3,'habit_created','2026-06-01 08:05:00',NULL),
(38,3,'checkin_logged','2026-06-01 20:00:00',NULL),
(39,3,'checkin_logged','2026-06-02 20:00:00',NULL),
(40,3,'checkin_logged','2026-06-03 20:00:00',NULL);
```

```sql
INSERT INTO events (event_id, user_id, event_name, event_time, event_value) VALUES
(41,3,'checkin_logged','2026-06-04 20:00:00',NULL),
(42,3,'checkin_logged','2026-06-05 20:00:00',NULL),
(43,3,'checkin_logged','2026-06-06 20:00:00',NULL),
(44,3,'checkin_logged','2026-06-07 20:00:00',NULL),
(45,4,'session_start','2026-06-01 08:00:00',NULL),
(46,5,'session_start','2026-06-01 08:00:00',NULL),
(47,5,'habit_created','2026-06-01 08:05:00',NULL),
(48,5,'checkin_logged','2026-06-01 20:00:00',NULL),
(49,5,'checkin_logged','2026-06-03 20:00:00',NULL),
(50,5,'checkin_logged','2026-06-04 20:00:00',NULL),
(51,5,'checkin_logged','2026-06-05 20:00:00',NULL),
(52,5,'checkin_logged','2026-06-06 20:00:00',NULL),
(53,5,'subscription_started','2026-06-06 09:30:00',9.00),
(54,5,'checkin_logged','2026-06-08 20:00:00',NULL),
(55,5,'checkin_logged','2026-06-09 20:00:00',NULL),
(56,5,'checkin_logged','2026-06-10 20:00:00',NULL),
(57,5,'checkin_logged','2026-06-11 20:00:00',NULL),
(58,5,'checkin_logged','2026-06-13 20:00:00',NULL),
(59,5,'checkin_logged','2026-06-14 20:00:00',NULL),
(60,5,'checkin_logged','2026-06-15 20:00:00',NULL),
(61,5,'checkin_logged','2026-06-17 20:00:00',NULL),
(62,5,'checkin_logged','2026-06-18 20:00:00',NULL),
(63,5,'checkin_logged','2026-06-19 20:00:00',NULL),
(64,5,'checkin_logged','2026-06-21 20:00:00',NULL),
(65,6,'session_start','2026-06-08 08:00:00',NULL),
(66,6,'habit_created','2026-06-08 08:05:00',NULL),
(67,6,'checkin_logged','2026-06-08 20:00:00',NULL),
(68,6,'checkin_logged','2026-06-09 20:00:00',NULL),
(69,6,'checkin_logged','2026-06-10 20:00:00',NULL),
(70,6,'invite_sent','2026-06-10 12:00:00',NULL),
(71,6,'checkin_logged','2026-06-11 20:00:00',NULL),
(72,6,'invite_accepted','2026-06-11 12:30:00',NULL),
(73,6,'subscription_started','2026-06-11 09:30:00',9.00),
(74,6,'checkin_logged','2026-06-12 20:00:00',NULL),
(75,6,'checkin_logged','2026-06-13 20:00:00',NULL),
(76,6,'checkin_logged','2026-06-14 20:00:00',NULL),
(77,6,'checkin_logged','2026-06-15 20:00:00',NULL),
(78,6,'checkin_logged','2026-06-16 20:00:00',NULL),
(79,6,'checkin_logged','2026-06-17 20:00:00',NULL),
(80,6,'checkin_logged','2026-06-19 20:00:00',NULL);
```

```sql
INSERT INTO events (event_id, user_id, event_name, event_time, event_value) VALUES
(81,6,'checkin_logged','2026-06-20 20:00:00',NULL),
(82,6,'checkin_logged','2026-06-21 20:00:00',NULL),
(83,7,'session_start','2026-06-08 08:00:00',NULL),
(84,7,'habit_created','2026-06-09 08:05:00',NULL),
(85,7,'checkin_logged','2026-06-09 20:00:00',NULL),
(86,7,'checkin_logged','2026-06-11 20:00:00',NULL),
(87,7,'checkin_logged','2026-06-13 20:00:00',NULL),
(88,7,'checkin_logged','2026-06-16 20:00:00',NULL),
(89,7,'checkin_logged','2026-06-18 20:00:00',NULL),
(90,7,'checkin_logged','2026-06-20 20:00:00',NULL),
(91,8,'session_start','2026-06-08 08:00:00',NULL),
(92,8,'habit_created','2026-06-08 08:05:00',NULL),
(93,8,'checkin_logged','2026-06-08 20:00:00',NULL),
(94,8,'checkin_logged','2026-06-09 20:00:00',NULL),
(95,8,'checkin_logged','2026-06-10 20:00:00',NULL),
(96,8,'checkin_logged','2026-06-11 20:00:00',NULL),
(97,8,'checkin_logged','2026-06-12 20:00:00',NULL),
(98,8,'checkin_logged','2026-06-13 20:00:00',NULL),
(99,8,'checkin_logged','2026-06-14 20:00:00',NULL),
(100,9,'session_start','2026-06-08 08:00:00',NULL),
(101,10,'session_start','2026-06-08 08:00:00',NULL),
(102,10,'habit_created','2026-06-08 08:05:00',NULL),
(103,10,'checkin_logged','2026-06-08 20:00:00',NULL),
(104,10,'checkin_logged','2026-06-10 20:00:00',NULL),
(105,10,'checkin_logged','2026-06-11 20:00:00',NULL),
(106,10,'checkin_logged','2026-06-12 20:00:00',NULL),
(107,10,'checkin_logged','2026-06-13 20:00:00',NULL),
(108,10,'subscription_started','2026-06-13 09:30:00',9.00),
(109,10,'checkin_logged','2026-06-15 20:00:00',NULL),
(110,10,'checkin_logged','2026-06-16 20:00:00',NULL),
(111,10,'checkin_logged','2026-06-17 20:00:00',NULL),
(112,10,'checkin_logged','2026-06-18 20:00:00',NULL),
(113,10,'checkin_logged','2026-06-20 20:00:00',NULL),
(114,10,'checkin_logged','2026-06-21 20:00:00',NULL),
(115,11,'session_start','2026-06-15 08:00:00',NULL),
(116,11,'habit_created','2026-06-15 08:05:00',NULL),
(117,11,'checkin_logged','2026-06-15 20:00:00',NULL),
(118,11,'checkin_logged','2026-06-16 20:00:00',NULL),
(119,11,'checkin_logged','2026-06-17 20:00:00',NULL),
(120,11,'invite_sent','2026-06-17 12:00:00',NULL);
```

```sql
INSERT INTO events (event_id, user_id, event_name, event_time, event_value) VALUES
(121,11,'checkin_logged','2026-06-18 20:00:00',NULL),
(122,11,'invite_accepted','2026-06-18 12:30:00',NULL),
(123,11,'subscription_started','2026-06-18 09:30:00',9.00),
(124,11,'checkin_logged','2026-06-19 20:00:00',NULL),
(125,11,'checkin_logged','2026-06-20 20:00:00',NULL),
(126,11,'checkin_logged','2026-06-21 20:00:00',NULL),
(127,12,'session_start','2026-06-15 08:00:00',NULL),
(128,12,'habit_created','2026-06-16 08:05:00',NULL),
(129,12,'checkin_logged','2026-06-16 20:00:00',NULL),
(130,12,'checkin_logged','2026-06-18 20:00:00',NULL),
(131,12,'checkin_logged','2026-06-20 20:00:00',NULL),
(132,13,'session_start','2026-06-15 08:00:00',NULL),
(133,13,'habit_created','2026-06-15 08:05:00',NULL),
(134,13,'checkin_logged','2026-06-15 20:00:00',NULL),
(135,13,'checkin_logged','2026-06-16 20:00:00',NULL),
(136,13,'checkin_logged','2026-06-17 20:00:00',NULL),
(137,13,'checkin_logged','2026-06-18 20:00:00',NULL),
(138,13,'checkin_logged','2026-06-19 20:00:00',NULL),
(139,13,'checkin_logged','2026-06-20 20:00:00',NULL),
(140,13,'checkin_logged','2026-06-21 20:00:00',NULL),
(141,14,'session_start','2026-06-15 08:00:00',NULL),
(142,15,'session_start','2026-06-15 08:00:00',NULL),
(143,15,'habit_created','2026-06-15 08:05:00',NULL),
(144,15,'checkin_logged','2026-06-15 20:00:00',NULL),
(145,15,'checkin_logged','2026-06-17 20:00:00',NULL),
(146,15,'checkin_logged','2026-06-18 20:00:00',NULL),
(147,15,'checkin_logged','2026-06-19 20:00:00',NULL),
(148,15,'checkin_logged','2026-06-20 20:00:00',NULL),
(149,15,'subscription_started','2026-06-20 09:30:00',9.00);
```

Sanity check — this should print `15` and `149`:

```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM events;
```

Three signup **cohorts** of five users each landed on 2026-06-01, 2026-06-08, and 2026-06-15. Data is loaded through **2026-06-21** — treat that as "today" everywhere this week. Notice the cohorts have *unequal observation windows* (cohort A has 21 days of history, cohort C only 7) — that's intentional and mirrors every real product's data: you always know more about your older users than your newer ones. Within each cohort, `user_id`s follow the same five behavior patterns (a power user, a casual user, a user who churns after one week, a user who signs up and never activates, and a referred power user) — a deliberate simplification so you can sanity-check your own queries against numbers you can reason out by hand.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed setup; why metrics + the north star | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | The AARRR funnel | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Events to a warehouse | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Metric-tree queries; funnel judgment | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Defend a metric under pressure | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (instrument + baseline queries) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-why-metrics-and-the-north-star.md](./lecture-notes/01-why-metrics-and-the-north-star.md) | What a metric is, why one north star, deriving input metrics | 2h |
| 2 | [lecture-notes/02-the-aarrr-funnel.md](./lecture-notes/02-the-aarrr-funnel.md) | Acquisition, Activation, Retention, Revenue, Referral — mapped to StreakLab | 2h |
| 3 | [lecture-notes/03-events-to-warehouse.md](./lecture-notes/03-events-to-warehouse.md) | Designing an event schema and building a metric tree in SQL | 2h |
| 4 | [exercises/exercise-01-pick-a-north-star.md](./exercises/exercise-01-pick-a-north-star.md) | Choose north stars for three different products and defend them | 1h |
| 5 | [exercises/exercise-02-classify-vanity-vs-actionable.md](./exercises/exercise-02-classify-vanity-vs-actionable.md) | Sort a metric list into vanity vs. actionable, with reasons | 1h |
| 6 | [exercises/exercise-03-query-a-metric-from-events.md](./exercises/exercise-03-query-a-metric-from-events.md) | Compute real metrics from `users`/`events` in SQL | 1.5h |
| 7 | [challenges/challenge-01-design-a-metric-tree.md](./challenges/challenge-01-design-a-metric-tree.md) | Build a full north-star + input-metric tree with SQL for every node | 1.5h |
| 8 | [challenges/challenge-02-defend-a-north-star.md](./challenges/challenge-02-defend-a-north-star.md) | Defend a north star against gaming, gaps, and an exec's pushback | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Instrument the event schema + ship a baseline metrics query set | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few growth-metrics links worth your time | — |

## By the end of this week you can…

- Say, in one sentence, why a product needs exactly **one** north-star metric and what makes a candidate metric a *good* one.
- Sort any metric you're handed into **vanity** or **actionable**, and explain the test you used.
- Draw the **AARRR funnel** for a real product and point to the stage with the most leverage.
- Design a `users` + `events` schema for a new product from a blank page.
- Write SQL that computes a north-star metric and its input metrics **directly from raw events** — no dashboard, no spreadsheet, fully reproducible.

## Up next

[Week 2 — Acquisition funnels & attribution](../week-02-acquisition-funnels-and-attribution/) — now that events are landing cleanly, we follow users in from every channel and argue about who gets credit for the signup.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
