# Week 5 — LTV, CAC & Unit Economics

> **Goal:** by Sunday you can take a subscription company's raw customer and spend data and answer the one question every investor, board, and finance team actually cares about — *is this growth worth buying?* — with a fully-loaded CAC, a margin-based LTV, a payback period, and a written verdict, all backed by SQL and pandas, not a gut feeling.

Welcome back to **C38 · Crunch Scale**. Weeks 1–4 built the instrumentation, the acquisition funnel, the activation flow, and the retention cohort. This week we take that retention curve and put a dollar sign on it. A cohort chart that trends up and to the right *feels* good, but it doesn't tell you whether the company is making money on the customers it's buying — or quietly setting cash on fire faster than the top-line growth suggests. LTV, CAC, and payback are how you turn "our cohorts look healthy" into "this channel returns $X for every $1 we spend on it, in Y months." That's the sentence a CFO can act on.

We work all week against one fictional company, **Lumen Metrics** — a B2B SaaS analytics dashboard for e-commerce brands — and two of its growth channels: **`paid_search`** (Google/Meta ads, targeting mid-market brands onto the $149/mo **Growth** plan) and **`organic_content`** (SEO blog + guides, mostly self-serve signups onto the $99/mo **Starter** plan). Same company, same 80% gross margin, two very different acquisition motions. By Friday you'll know, with numbers, which one deserves the next marketing dollar.

You load two small tables once (below), and every lecture, exercise, challenge, and the mini-project queries them.

## Learning objectives

By the end of this week, you will be able to:

- **Build** customer lifetime value (LTV) from an empirical retention curve and **contribution margin** — not revenue — and explain why a revenue-based LTV systematically overstates the truth.
- **Compute** fully-loaded customer acquisition cost (CAC) per channel — media spend *and* team cost, not just ad spend — plus the blended CAC across channels.
- **Calculate** CAC payback period two ways (naive vs. retention-adjusted) and the LTV:CAC ratio, and read both against the standard SaaS health heuristics (3:1 ratio, <12–18 month payback).
- **Model** contribution margin per customer per month, and separate "revenue is growing" from "each customer is profitable."
- **Judge**, in writing, whether a growing subscription business — or one specific channel inside it — is actually unit-profitable, and defend a scale/pause/fix recommendation with the query that proves it.

## Prerequisites

- Comfortable with SQL `SELECT`, `WHERE`, `GROUP BY`, aggregates, `JOIN`, and window functions — [C33 Crunch SQL](../../../C33-CRUNCH-SQL/), or [Week 4](../week-04-retention-cohorts-and-survival/) of this course, which builds the retention-cohort technique this week assumes.
- Comfortable with pandas `groupby`, `merge`, and basic arithmetic on a `DataFrame` — Week 1 of this course, or equivalent.
- **No** prior finance, accounting, or SaaS-metrics background required. That's what this week is for.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` installed. Install steps are in [`resources.md`](./resources.md). **Data lives in SQL and pandas this week — never in a spreadsheet.**

## Set up the seed database (do this first)

Everything this week runs against two tables. Create them once.

**PostgreSQL:**

```bash
createdb crunch_scale_w5
psql crunch_scale_w5
```

**SQLite:**

```bash
sqlite3 crunch_scale_w5.db
```

Then paste this into the shell (it works unchanged on both engines):

```sql
-- ── Customers: every Lumen Metrics signup in FY2025, by acquisition channel ──
CREATE TABLE customers (
    customer_id     INTEGER PRIMARY KEY,
    channel         TEXT    NOT NULL,   -- 'paid_search' | 'organic_content'
    signup_month    DATE    NOT NULL,   -- acquisition cohort, first of month
    mrr             NUMERIC NOT NULL,   -- monthly recurring revenue while active
    churn_month     DATE                -- NULL = still active as of the 2025-12-31 cutoff
);

INSERT INTO customers VALUES
(1,'paid_search','2025-01-01',149.00,'2025-03-01'),
(2,'paid_search','2025-01-01',149.00,'2025-09-01'),
(3,'paid_search','2025-01-01',149.00,'2025-04-01'),
(4,'paid_search','2025-02-01',149.00,'2025-09-01'),
(5,'paid_search','2025-02-01',149.00,NULL),
(6,'paid_search','2025-02-01',149.00,NULL),
(7,'paid_search','2025-03-01',149.00,'2025-06-01'),
(8,'paid_search','2025-03-01',149.00,'2025-05-01'),
(9,'paid_search','2025-03-01',149.00,NULL),
(10,'paid_search','2025-04-01',149.00,'2025-05-01'),
(11,'paid_search','2025-04-01',149.00,NULL),
(12,'paid_search','2025-04-01',149.00,NULL),
(13,'paid_search','2025-05-01',149.00,NULL),
(14,'paid_search','2025-05-01',149.00,'2025-06-01'),
(15,'paid_search','2025-05-01',149.00,'2025-09-01'),
(16,'paid_search','2025-06-01',149.00,'2025-12-01'),
(17,'paid_search','2025-06-01',149.00,NULL),
(18,'paid_search','2025-06-01',149.00,NULL),
(19,'paid_search','2025-07-01',149.00,'2025-09-01'),
(20,'paid_search','2025-07-01',149.00,'2025-09-01'),
(21,'paid_search','2025-07-01',149.00,'2025-12-01'),
(22,'paid_search','2025-08-01',149.00,'2025-11-01'),
(23,'paid_search','2025-08-01',149.00,'2025-10-01'),
(24,'paid_search','2025-08-01',149.00,NULL),
(25,'paid_search','2025-09-01',149.00,NULL),
(26,'paid_search','2025-09-01',149.00,'2025-11-01'),
(27,'paid_search','2025-09-01',149.00,NULL),
(28,'paid_search','2025-10-01',149.00,NULL),
(29,'paid_search','2025-10-01',149.00,'2025-11-01'),
(30,'paid_search','2025-10-01',149.00,NULL),
(31,'paid_search','2025-11-01',149.00,NULL),
(32,'paid_search','2025-11-01',149.00,NULL),
(33,'paid_search','2025-11-01',149.00,NULL),
(34,'paid_search','2025-12-01',149.00,NULL),
(35,'paid_search','2025-12-01',149.00,NULL),
(36,'paid_search','2025-12-01',149.00,NULL),
(37,'organic_content','2025-01-01',99.00,NULL),
(38,'organic_content','2025-02-01',99.00,NULL),
(39,'organic_content','2025-03-01',99.00,'2025-04-01'),
(40,'organic_content','2025-04-01',99.00,NULL),
(41,'organic_content','2025-04-01',99.00,NULL),
(42,'organic_content','2025-05-01',99.00,'2025-12-01'),
(43,'organic_content','2025-05-01',99.00,'2025-06-01'),
(44,'organic_content','2025-06-01',99.00,NULL),
(45,'organic_content','2025-06-01',99.00,NULL),
(46,'organic_content','2025-07-01',99.00,'2025-11-01'),
(47,'organic_content','2025-07-01',99.00,NULL),
(48,'organic_content','2025-08-01',99.00,NULL),
(49,'organic_content','2025-08-01',99.00,NULL),
(50,'organic_content','2025-08-01',99.00,NULL),
(51,'organic_content','2025-09-01',99.00,NULL),
(52,'organic_content','2025-09-01',99.00,NULL),
(53,'organic_content','2025-09-01',99.00,NULL),
(54,'organic_content','2025-10-01',99.00,NULL),
(55,'organic_content','2025-10-01',99.00,NULL),
(56,'organic_content','2025-10-01',99.00,NULL),
(57,'organic_content','2025-11-01',99.00,NULL),
(58,'organic_content','2025-11-01',99.00,'2025-12-01'),
(59,'organic_content','2025-11-01',99.00,NULL),
(60,'organic_content','2025-11-01',99.00,NULL),
(61,'organic_content','2025-12-01',99.00,NULL),
(62,'organic_content','2025-12-01',99.00,NULL),
(63,'organic_content','2025-12-01',99.00,NULL),
(64,'organic_content','2025-12-01',99.00,NULL);

-- ── Channel costs: fully-loaded monthly spend per channel, all of FY2025 ──
CREATE TABLE channel_costs (
    cost_month   DATE    NOT NULL,
    channel      TEXT    NOT NULL,
    ad_spend     NUMERIC NOT NULL,   -- media/tool spend that's tied directly to volume
    team_cost    NUMERIC NOT NULL,   -- allocated salaries + tools for that channel's team
    PRIMARY KEY (cost_month, channel)
);

INSERT INTO channel_costs VALUES
('2025-01-01','organic_content',400,5600),
('2025-01-01','paid_search',2700,3300),
('2025-02-01','organic_content',400,5600),
('2025-02-01','paid_search',2700,3300),
('2025-03-01','organic_content',400,5600),
('2025-03-01','paid_search',2700,3300),
('2025-04-01','organic_content',400,5600),
('2025-04-01','paid_search',2700,3300),
('2025-05-01','organic_content',400,5600),
('2025-05-01','paid_search',2700,3300),
('2025-06-01','organic_content',400,5600),
('2025-06-01','paid_search',2700,3300),
('2025-07-01','organic_content',400,5600),
('2025-07-01','paid_search',2700,3300),
('2025-08-01','organic_content',400,5600),
('2025-08-01','paid_search',2700,3300),
('2025-09-01','organic_content',400,5600),
('2025-09-01','paid_search',2700,3300),
('2025-10-01','organic_content',400,5600),
('2025-10-01','paid_search',2700,3300),
('2025-11-01','organic_content',400,5600),
('2025-11-01','paid_search',2700,3300),
('2025-12-01','organic_content',400,5600),
('2025-12-01','paid_search',2700,3300);
```

Sanity check — this should print `64` and `24`:

```sql
SELECT COUNT(*) FROM customers;       -- 64  (36 paid_search + 28 organic_content)
SELECT COUNT(*) FROM channel_costs;   -- 24  (12 months × 2 channels)
```

Four facts to hold onto all week — you'll need every one of them:

- **Data cutoff is 2025-12-31.** A `NULL` `churn_month` means "still an active, paying subscriber as of the cutoff" — it does **not** mean "will pay forever." Treating `NULL` as "permanent" is *the* classic LTV mistake, and Lecture 1 spends real time on it. This is a *right-censored* dataset: for every recent cohort, you genuinely don't yet know how long they'll last.
- **Lumen Metrics runs an 80% gross margin company-wide** (cloud hosting + support = 20% of revenue, regardless of plan or channel). That 80% is your contribution-margin multiplier all week — same role the flat tax rate played in earlier courses.
- **`channel_costs` is fully loaded on purpose.** `ad_spend` is the media/tool line that scales with volume; `team_cost` is the allocated salary and tooling cost for the people running that channel, and it does **not** scale with volume in the short run. Both belong in CAC. A CAC that only counts `ad_spend` is *not* fully loaded — Lecture 2 shows you exactly how much that understates the real number.
- **The two channels are deliberately asymmetric.** `paid_search` buys customers at a steady, predictable rate (3/month, all year) at a flat cost. `organic_content` starts slow (1 new customer/month in January) and **compounds** (4 new customers/month by December) on the *same* flat monthly budget — because content, unlike ads, keeps working after you've paid for it. Watch what that compounding does to CAC over the year.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Setup + LTV from retention & contribution margin | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | LTV drills — reciprocal formula vs. cohort curve | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Fully-loaded CAC, blended CAC, payback | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Unit economics that hold — CM, SQL + pandas models | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | Challenges: two-channel model + spot the bad story | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (two-channel unit-economics model) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-lifetime-value.md](./lecture-notes/01-lifetime-value.md) | LTV from an empirical retention curve, the churn-rate reciprocal formula, contribution margin, right-censoring, why revenue-based LTV overstates | 2h |
| 2 | [lecture-notes/02-cac-and-payback.md](./lecture-notes/02-cac-and-payback.md) | Fully-loaded CAC by channel, blended CAC, naive vs. retention-adjusted payback, the LTV:CAC ratio | 2h |
| 3 | [lecture-notes/03-unit-economics-that-hold.md](./lecture-notes/03-unit-economics-that-hold.md) | Contribution margin end to end, growing vs. profitable, modeling the whole thing in SQL and pandas | 2h |
| 4 | [exercises/exercise-01-compute-ltv-from-cohorts.md](./exercises/exercise-01-compute-ltv-from-cohorts.md) | Build the pooled retention curve in SQL and compute LTV two ways | 1h |
| 5 | [exercises/exercise-02-loaded-cac-per-channel.md](./exercises/exercise-02-loaded-cac-per-channel.md) | Fully-loaded CAC per channel, per quarter, and blended | 1h |
| 6 | [exercises/exercise-03-payback-and-ratio.md](./exercises/exercise-03-payback-and-ratio.md) | Payback period (both methods) and the LTV:CAC ratio, in pandas | 1h |
| 7 | [challenges/challenge-01-model-two-channel-economics.md](./challenges/challenge-01-model-two-channel-economics.md) | A single, reusable unit-economics model comparing both channels side by side | 1h |
| 8 | [challenges/challenge-02-spot-unprofitable-growth.md](./challenges/challenge-02-spot-unprofitable-growth.md) | A VP's growth memo makes a case with real numbers — find where the math quietly lies | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Full LTV/CAC/payback model for both channels + a scale/pause/fix recommendation | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Build a pooled retention curve from raw customer rows and turn it into a defensible LTV — in SQL, in pandas, or both.
- Compute a *fully-loaded* CAC per channel (not just ad spend), a blended CAC, and know exactly why they diverge.
- State a channel's LTV:CAC ratio and payback period from memory-level muscle, and judge it against the 3:1 / <18-month heuristics without looking them up.
- Explain, precisely, why "revenue is growing" and "this channel is unit-profitable" are two different claims — and which one a diligent operator checks first.
- Read a two-channel unit-economics model and recommend, in writing, which one deserves the next dollar.

## Up next

[Week 6 — RevOps & the Customer Data Stack](../week-06-revops-and-the-customer-data-stack/) — now that you can price a channel, we build the warehouse mart layer that feeds this model automatically instead of by hand.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
