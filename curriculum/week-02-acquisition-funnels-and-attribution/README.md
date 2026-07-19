# Week 2 — Acquisition Funnels & Attribution

> **Goal:** by Sunday you can take a raw touch-and-event log and answer the two questions every growth stakeholder actually asks — "where's the drop-off?" and "which channel deserves the credit?" — writing funnel and attribution SQL from memory, and defending a CAC number instead of just reporting one.

Last week you picked a north-star metric and got events landing in a warehouse. This week you turn those events into the two artifacts that decide where a growth team spends its time and its money: the **acquisition funnel** (how many people make it from first touch to paying customer, and where they fall off) and the **attribution model** (which channel gets credit for that paying customer, and how much). Get comfortable with both and you'll never again accept "traffic is up" or "that channel converts great" as a finished sentence — you'll ask *converts what, measured how, credited to whom*.

We work against one seeded warehouse — five tables modeling **Crunch**'s acquisition data for the first ten days of a launch window — for the whole week. Set it up once (below); every lecture, exercise, challenge, and the mini-project queries it.

## Learning objectives

By the end of this week, you will be able to:

- **Model** a funnel as an ordered sequence of steps in an events table, and compute step-by-step counts and conversion rates in SQL — not in a spreadsheet, not by eyeballing a chart.
- **Use window functions** (`LAG`, `ROW_NUMBER`, `COUNT() OVER`) to find each user's first and last touch, order their steps chronologically, and bound conversions to a time window.
- **Build first-touch, last-touch, linear, and position-based attribution models** from a single touch table, and explain in one sentence what each model rewards.
- **Compare attribution models side by side** on the same conversions and predict, before running the query, which channels will gain or lose credit under each one.
- **Compute per-channel CAC** (spend ÷ attributed conversions) under more than one attribution model, and read CAC *next to* conversion rate and volume — never in isolation.
- **Distinguish a channel that scales** (marginal CAC holds as spend grows) **from a channel that only reshuffles organic demand** (cheap last-touch CAC because it's capturing intent another channel already created).
- **Debug a funnel or attribution query** that runs without error but silently double-counts or under-counts — the single most common analytics bug in a growth org.

## Prerequisites

- Week 1 of this course (north-star metric, AARRR, landing events in a warehouse) — or equivalent comfort with an events-table mental model.
- SQL at the level of [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) weeks 1–4: `SELECT`/`WHERE`/`ORDER BY`, joins, `GROUP BY`/`HAVING`, and window functions (`ROW_NUMBER`, `LAG`/`LEAD`, `PARTITION BY`). If window functions feel shaky, skim [C33 week 5](../../../C33-CRUNCH-SQL/curriculum/) before Monday.
- PostgreSQL 16+ or SQLite 3.35+, same as every week. **Data lives in SQL — never in a spreadsheet.**

## Set up the seed database (do this first)

Everything this week runs against five tables: `channels`, `users`, `touches`, `funnel_events`, `conversions`, plus `ad_spend`. It's a compressed, ten-day launch window for **Crunch**, a B2B SaaS product — 20 users, 31 marketing touches, 59 funnel events, 10 paying customers, and $4,500 of ad spend across four paid channels. Small enough to hold in your head, real enough to argue about.

**PostgreSQL:**

```bash
createdb crunch_growth
psql crunch_growth
```

**SQLite:**

```bash
sqlite3 crunch_growth.db
```

Paste this into the shell (unchanged on both engines):

```sql
CREATE TABLE channels (
    channel_id     INTEGER PRIMARY KEY,
    channel_name   TEXT    NOT NULL,
    channel_type   TEXT    NOT NULL     -- 'paid' | 'organic' | 'owned' | 'referral'
);

CREATE TABLE users (
    user_id        INTEGER PRIMARY KEY,
    first_seen_ts  TIMESTAMP NOT NULL
);

CREATE TABLE touches (
    touch_id       INTEGER PRIMARY KEY,
    user_id        INTEGER NOT NULL REFERENCES users(user_id),
    channel_id     INTEGER NOT NULL REFERENCES channels(channel_id),
    touch_ts       TIMESTAMP NOT NULL
);

CREATE TABLE funnel_events (
    event_id       INTEGER PRIMARY KEY,
    user_id        INTEGER NOT NULL REFERENCES users(user_id),
    step_name      TEXT    NOT NULL,    -- 'site_visit' | 'signup' | 'activated' | 'paid_conversion'
    event_ts       TIMESTAMP NOT NULL
);

CREATE TABLE conversions (
    user_id        INTEGER PRIMARY KEY REFERENCES users(user_id),
    converted_ts   TIMESTAMP NOT NULL,
    plan           TEXT    NOT NULL,    -- 'Starter' | 'Growth' | 'Scale'
    revenue        NUMERIC NOT NULL     -- first-month recurring revenue, USD
);

CREATE TABLE ad_spend (
    spend_id       INTEGER PRIMARY KEY,
    channel_id     INTEGER NOT NULL REFERENCES channels(channel_id),
    spend_date     DATE    NOT NULL,
    spend_amount   NUMERIC NOT NULL
);

INSERT INTO channels VALUES
(1,'Organic Search','organic'),
(2,'Direct','owned'),
(3,'Referral','referral'),
(4,'Email','owned'),
(5,'Google Search - Nonbrand','paid'),
(6,'Google Search - Brand','paid'),
(7,'Meta Ads','paid'),
(8,'Affiliate','paid');

INSERT INTO users VALUES
(1,'2025-01-02 09:15'),(2,'2025-01-01 08:00'),(3,'2025-01-01 10:00'),(4,'2025-01-02 12:00'),
(5,'2025-01-03 15:00'),(6,'2025-01-01 09:00'),(7,'2025-01-02 16:00'),(8,'2025-01-03 09:45'),
(9,'2025-01-02 08:00'),(10,'2025-01-03 11:00'),(11,'2025-01-04 14:00'),(12,'2025-01-05 09:30'),
(13,'2025-01-02 13:00'),(14,'2025-01-06 10:15'),(15,'2025-01-05 08:00'),(16,'2025-01-04 09:00'),
(17,'2025-01-06 12:00'),(18,'2025-01-01 07:30'),(19,'2025-01-07 09:00'),(20,'2025-01-10 08:00');

INSERT INTO touches VALUES
(1,1,5,'2025-01-02 09:15'),(2,1,4,'2025-01-05 14:00'),
(3,2,7,'2025-01-01 08:00'),
(4,3,1,'2025-01-01 10:00'),
(5,4,5,'2025-01-02 12:00'),(6,4,6,'2025-01-04 12:30'),
(7,5,3,'2025-01-03 15:00'),
(8,6,8,'2025-01-01 09:00'),(9,6,4,'2025-01-06 10:00'),
(10,7,2,'2025-01-02 16:00'),
(11,8,5,'2025-01-03 09:45'),
(12,9,7,'2025-01-02 08:00'),(13,9,7,'2025-01-04 08:15'),(14,9,4,'2025-01-07 09:00'),
(15,10,1,'2025-01-03 11:00'),(16,10,2,'2025-01-05 11:30'),
(17,11,6,'2025-01-04 14:00'),
(18,12,8,'2025-01-05 09:30'),
(19,13,3,'2025-01-02 13:00'),(20,13,4,'2025-01-08 09:00'),
(21,14,5,'2025-01-06 10:15'),
(22,15,7,'2025-01-05 08:00'),
(23,16,1,'2025-01-04 09:00'),(24,16,4,'2025-01-09 09:00'),
(25,17,2,'2025-01-06 12:00'),
(26,18,5,'2025-01-01 07:30'),(27,18,7,'2025-01-05 07:45'),(28,18,6,'2025-01-09 08:00'),
(29,19,8,'2025-01-07 09:00'),(30,19,8,'2025-01-08 09:30'),
(31,20,5,'2025-01-10 08:00');

INSERT INTO funnel_events VALUES
(1,1,'site_visit','2025-01-05 14:05'),(2,1,'signup','2025-01-05 14:20'),(3,1,'activated','2025-01-06 10:00'),(4,1,'paid_conversion','2025-01-09 11:00'),
(5,2,'site_visit','2025-01-01 08:05'),(6,2,'signup','2025-01-01 08:30'),(7,2,'activated','2025-01-02 09:00'),(8,2,'paid_conversion','2025-01-04 09:00'),
(9,3,'site_visit','2025-01-01 10:02'),(10,3,'signup','2025-01-02 11:00'),(11,3,'activated','2025-01-03 12:00'),(12,3,'paid_conversion','2025-01-06 09:30'),
(13,4,'site_visit','2025-01-04 12:35'),(14,4,'signup','2025-01-04 13:00'),(15,4,'activated','2025-01-05 09:00'),
(16,5,'site_visit','2025-01-03 15:10'),(17,5,'signup','2025-01-03 15:30'),
(18,6,'site_visit','2025-01-06 10:05'),(19,6,'signup','2025-01-06 10:20'),(20,6,'activated','2025-01-07 11:00'),(21,6,'paid_conversion','2025-01-10 08:00'),
(22,7,'site_visit','2025-01-02 16:02'),
(23,8,'site_visit','2025-01-03 09:47'),
(24,9,'site_visit','2025-01-07 09:05'),(25,9,'signup','2025-01-07 09:30'),(26,9,'activated','2025-01-08 10:00'),(27,9,'paid_conversion','2025-01-12 10:00'),
(28,10,'site_visit','2025-01-05 11:35'),(29,10,'signup','2025-01-05 12:00'),(30,10,'activated','2025-01-06 09:00'),
(31,11,'site_visit','2025-01-04 14:05'),(32,11,'signup','2025-01-04 14:30'),(33,11,'activated','2025-01-05 15:00'),(34,11,'paid_conversion','2025-01-07 09:00'),
(35,12,'site_visit','2025-01-05 09:35'),(36,12,'signup','2025-01-05 10:00'),
(37,13,'site_visit','2025-01-08 09:05'),(38,13,'signup','2025-01-08 09:30'),(39,13,'activated','2025-01-09 10:00'),(40,13,'paid_conversion','2025-01-13 11:00'),
(41,14,'site_visit','2025-01-06 10:17'),
(42,15,'site_visit','2025-01-05 08:05'),(43,15,'signup','2025-01-05 08:30'),
(44,16,'site_visit','2025-01-09 09:05'),(45,16,'signup','2025-01-09 09:30'),(46,16,'activated','2025-01-10 10:00'),(47,16,'paid_conversion','2025-01-14 10:30'),
(48,17,'site_visit','2025-01-06 12:02'),(49,17,'signup','2025-01-06 12:30'),(50,17,'activated','2025-01-07 13:00'),
(51,18,'site_visit','2025-01-09 08:05'),(52,18,'signup','2025-01-09 08:30'),(53,18,'activated','2025-01-10 09:00'),(54,18,'paid_conversion','2025-01-13 09:00'),
(55,19,'site_visit','2025-01-08 09:35'),
(56,20,'site_visit','2025-01-10 08:05'),(57,20,'signup','2025-01-10 08:20'),(58,20,'activated','2025-01-11 09:00'),(59,20,'paid_conversion','2025-01-15 10:00');

INSERT INTO conversions VALUES
(1,'2025-01-09 11:00','Growth',149),
(2,'2025-01-04 09:00','Starter',49),
(3,'2025-01-06 09:30','Growth',149),
(6,'2025-01-10 08:00','Scale',349),
(9,'2025-01-12 10:00','Growth',149),
(11,'2025-01-07 09:00','Starter',49),
(13,'2025-01-13 11:00','Growth',149),
(16,'2025-01-14 10:30','Scale',349),
(18,'2025-01-13 09:00','Growth',149),
(20,'2025-01-15 10:00','Starter',49);

INSERT INTO ad_spend VALUES
(1,5,'2025-01-01',150),(2,5,'2025-01-02',150),(3,5,'2025-01-03',160),(4,5,'2025-01-04',160),(5,5,'2025-01-05',170),
(6,5,'2025-01-06',170),(7,5,'2025-01-07',180),(8,5,'2025-01-08',180),(9,5,'2025-01-09',190),(10,5,'2025-01-10',190),
(11,6,'2025-01-01',40),(12,6,'2025-01-02',40),(13,6,'2025-01-03',40),(14,6,'2025-01-04',40),(15,6,'2025-01-05',40),
(16,6,'2025-01-06',40),(17,6,'2025-01-07',40),(18,6,'2025-01-08',40),(19,6,'2025-01-09',40),(20,6,'2025-01-10',40),
(21,7,'2025-01-01',80),(22,7,'2025-01-02',100),(23,7,'2025-01-03',120),(24,7,'2025-01-04',140),(25,7,'2025-01-05',160),
(26,7,'2025-01-06',180),(27,7,'2025-01-07',200),(28,7,'2025-01-08',220),(29,7,'2025-01-09',240),(30,7,'2025-01-10',260),
(31,8,'2025-01-01',70),(32,8,'2025-01-02',70),(33,8,'2025-01-03',70),(34,8,'2025-01-04',70),(35,8,'2025-01-05',70),
(36,8,'2025-01-06',70),(37,8,'2025-01-07',70),(38,8,'2025-01-08',70),(39,8,'2025-01-09',70),(40,8,'2025-01-10',70);
```

Sanity checks — these should return exactly these numbers:

```sql
SELECT COUNT(*) FROM users;              -- 20
SELECT COUNT(*) FROM touches;            -- 31
SELECT COUNT(*) FROM funnel_events;      -- 59
SELECT COUNT(*) FROM conversions;        -- 10
SELECT SUM(spend_amount) FROM ad_spend;  -- 4500
```

A few things are baked into this data **on purpose** — you'll need them:

- Every user has a `site_visit` event, but only 16 signed up, 13 activated, and 10 paid. That drop-off is the funnel.
- Users 9 and 18 have **three** touches each (multi-touch paths) — 9 even touches the same channel (Meta Ads) twice before converting via Email. These are your attribution-model stress tests.
- `Google Search - Brand` (channel 6) has flat, cheap spend ($40/day) and converts two users — one of whom (18) touched two *other* paid channels first. That's not an accident; it's this week's core lesson, and Lecture 3 makes you prove it.
- `Meta Ads` (channel 7) spend **ramps** from $80/day to $260/day over the ten days while unique converters stay flat — that's a saturation curve hiding in a spend table.
- `Affiliate` (channel 8) spent $700 and touched three users, but only **one** of them ever paid — and even then, Affiliate wasn't the last touch. It looks active. It isn't delivering.

## Weekly schedule

Adds up to ~28 hours (full-time pace). Treat it as a target, not a contract.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed setup; funnel anatomy + window functions | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Funnel query fluency; conversion windows | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Attribution models — first/last/linear/position | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Attribution exercise; multi-touch challenge | 0h | 1h | 1.5h | 0.5h | 1h | 0h | 4h |
| Friday | Channels that scale; CAC report | 2h | 1h | 1h | 0.5h | 1h | 0.5h | 6h |
| Saturday | Debug challenge; mini-project | 0h | 0h | 0.5h | 0h | 0h | 2.5h | 3h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 1h | 2h |
| **Total** | | **6h** | **6h** | **3h** | **3.5h** | **5h** | **4h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-anatomy-of-an-acquisition-funnel.md](./lecture-notes/01-anatomy-of-an-acquisition-funnel.md) | Funnel steps as events, conditional counting, window functions, time-lag conversion windows | 2h |
| 2 | [lecture-notes/02-attribution-models.md](./lecture-notes/02-attribution-models.md) | First-touch, last-touch, linear, position-based attribution, built from `touches` in SQL | 2h |
| 3 | [lecture-notes/03-channels-that-scale.md](./lecture-notes/03-channels-that-scale.md) | Marginal CAC, saturation, spotting channels that reshuffle organic demand | 2h |
| 4 | [exercises/exercise-01-build-a-funnel-query.md](./exercises/exercise-01-build-a-funnel-query.md) | Build the full site_visit → paid funnel with drop-off | 1h |
| 5 | [exercises/exercise-02-first-vs-last-touch.md](./exercises/exercise-02-first-vs-last-touch.md) | Compare first- and last-touch attribution on real revenue | 1.5h |
| 6 | [exercises/exercise-03-channel-cac-report.md](./exercises/exercise-03-channel-cac-report.md) | Join spend to attributed conversions; compute CAC two ways | 1h |
| 7 | [challenges/challenge-01-multi-touch-attribution-model.md](./challenges/challenge-01-multi-touch-attribution-model.md) | Build linear and position-based (40/20/40) models from scratch | 1.5h |
| 8 | [challenges/challenge-02-debug-a-misleading-funnel.md](./challenges/challenge-02-debug-a-misleading-funnel.md) | Find and fix a fan-out bug that silently triples attributed revenue | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Full funnel + multi-touch attribution report + a spend recommendation | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Turn a raw event log into a step-by-step funnel with defensible conversion rates.
- Build four different attribution models from the same touch data and explain why they disagree.
- Report CAC next to conversion rate and volume, and say which channel earns the next dollar.
- Spot — and prove, in SQL — when a channel's great numbers are an artifact of the attribution model, not real growth.

## Up next

Week 3 — Activation & onboarding: once people arrive through the right channels, the next question is why so few of them stick around long enough to get value.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
