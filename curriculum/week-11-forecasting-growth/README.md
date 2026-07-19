# Week 11 — Forecasting Growth

> **Goal:** by Sunday you can take a subscription company's trailing twelve months of billing history and turn it into a defensible next-four-quarter MRR forecast — built bottoms-up from new, expansion, contraction, and churned revenue, cross-checked against a retention-adjusted cohort projection, wrapped in base/bull/bear scenarios with a quantified uncertainty band — and write the memo that defends every assumption in it to a CFO who will ask "why?" about each one.

Welcome back to **C38 · Crunch Scale**. Every week before this one has looked backward or sideways: what happened, what a segment looks like today, what an experiment proved. This week you look forward. Forecasting is the moment growth work stops being descriptive and starts being accountable — a forecast is a promise with a number attached, and the only thing worse than missing it is not being able to explain, precisely, which assumption broke. You will build forecasts three different ways this week, on purpose: a bottoms-up MRR bridge (every dollar traced to new/expansion/contraction/churn), a retention-adjusted cohort projection (every dollar traced to a cohort's survival curve), and a simple time-series fit with an honest uncertainty band. When the three roughly agree, you trust the number. When they don't, you've just found the thing worth investigating before anyone puts it in a board deck.

We work all week against **Crunch Flow**, the B2B project-management SaaS from [Week 6](../week-06-revops-and-the-customer-data-stack/) — same company, same three plans (**Starter** $29/mo, **Growth** $99/mo, **Scale** $299/mo). It's now **January 2026**. You have twelve clean months of actuals (Jan–Dec 2025) sitting in an `mrr_bridge_actuals` table — the exact shape of the `fct_mrr_monthly`-style mart Week 6 taught you to build — plus a `cohort_revenue_retention` table tracking six signup cohorts' revenue survival by age. Finance wants a Q1–Q4 2026 forecast on their desk by Friday. This week, you build it.

You load two small tables once (below), and every lecture, exercise, challenge, and the mini-project queries them.

## Learning objectives

By the end of this week, you will be able to:

- **Build** a bottoms-up MRR forecast forward from historical new, expansion, contraction, and churned revenue — a forecast where every assumption is a named, visible number, not a black box.
- **Apply** cohort revenue-retention curves and survival-curve logic to project retention-adjusted revenue, correctly handling right-censored (not-yet-fully-observed) cohorts instead of pretending they're mature.
- **Fit** a simple time-series forecast (linear trend / OLS) to a revenue series, and quantify its uncertainty with a residual-based prediction interval that widens the further out you forecast.
- **Construct** base, bull, and bear scenarios from explicit, named assumption deltas — not vibes — and roll them up into quarterly figures a stakeholder can act on.
- **Communicate** a forecast in writing: state the method, name every assumption, show the range, and defend the number under questioning.

## Prerequisites

- **[Week 5](../week-05-ltv-cac-and-unit-economics/)** of this course — you'll reuse the empirical retention-curve technique, now applied to *revenue* survival instead of LTV.
- **[Week 6](../week-06-revops-and-the-customer-data-stack/)** of this course — this week assumes you understand what an MRR bridge (`starting + new + expansion − contraction − churned = ending`) is and why it's the trusted shape for revenue data.
- Comfortable with SQL window functions (`LAG`, `SUM() OVER`, `AVG() OVER`) — [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 4–5, or equivalent.
- Comfortable with pandas (`groupby`, arithmetic on a `DataFrame`, a basic `for` loop building rows) and `numpy.polyfit` or equivalent for a one-variable linear fit.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` and `numpy` installed. Install steps are in [`resources.md`](./resources.md). **Data lives in SQL and pandas this week — never in a spreadsheet.** A forecast built in spreadsheet cells with no visible formula trail is exactly the failure mode this week teaches you to avoid.

## Set up the seed database (do this first)

This week's data lives in two tables:

1. **`mrr_bridge_actuals`** — one row per month, Jan–Dec 2025, with the full MRR bridge: starting MRR, new MRR, expansion MRR, contraction MRR, churned MRR, ending MRR, plus customer counts. This is your trailing-twelve-months actuals — the thing you forecast *from*.
2. **`cohort_revenue_retention`** — one row per (signup cohort, months-since-signup), tracking what fraction of a cohort's starting MRR is still active at each age. Six cohorts (Jan–Jun 2025), each observed through the same December 2025 cutoff — so older cohorts have more data points than younger ones. That's not a mistake; it's what real cohort data always looks like, and Lecture 2 shows you how to handle it correctly.

**PostgreSQL:**

```bash
createdb crunch_scale_w11
psql crunch_scale_w11
```

**SQLite:**

```bash
sqlite3 crunch_scale_w11.db
```

Then paste this into the shell (it works unchanged on both engines):

```sql
-- ============================================================
-- mrr_bridge_actuals — trailing 12 months of Crunch Flow's
-- MRR bridge. One row per month. This is the number Finance
-- already trusts (it's the mart Week 6 taught you to build).
-- ============================================================
CREATE TABLE mrr_bridge_actuals (
    bridge_id          INTEGER PRIMARY KEY,
    month               DATE    NOT NULL,   -- first of month
    starting_mrr        NUMERIC NOT NULL,
    new_mrr              NUMERIC NOT NULL,
    expansion_mrr        NUMERIC NOT NULL,
    contraction_mrr       NUMERIC NOT NULL,
    churned_mrr            NUMERIC NOT NULL,
    ending_mrr              NUMERIC NOT NULL,
    new_customers            INTEGER NOT NULL,
    churned_customers        INTEGER NOT NULL,
    starting_customers        INTEGER NOT NULL,
    ending_customers            INTEGER NOT NULL
);

INSERT INTO mrr_bridge_actuals VALUES
(1,'2025-01-01',40000,3200,900,350,1100,42650,22,8,300,314),
(2,'2025-02-01',42650,3400,1050,300,1250,45550,24,9,314,329),
(3,'2025-03-01',45550,3600,1200,400,1300,48650,25,9,329,345),
(4,'2025-04-01',48650,3100,1350,450,1400,51250,21,10,345,356),
(5,'2025-05-01',51250,3800,1500,380,1500,54670,26,10,356,372),
(6,'2025-06-01',54670,4000,1650,500,1600,58220,27,11,372,388),
(7,'2025-07-01',58220,3300,1550,480,1700,60890,23,12,388,399),
(8,'2025-08-01',60890,2600,1400,520,1850,62520,18,13,399,404),
(9,'2025-09-01',62520,4300,2000,500,1900,66420,29,13,404,420),
(10,'2025-10-01',66420,4400,2100,550,2000,70370,29,14,420,435),
(11,'2025-11-01',70370,4600,2250,600,2100,74520,30,14,435,451),
(12,'2025-12-01',74520,5200,2400,650,2200,79270,33,15,451,469);

-- ============================================================
-- cohort_revenue_retention — for each signup cohort (age 0 =
-- the cohort's first month), what fraction of its STARTING
-- MRR is still active at each later age. Net of within-cohort
-- churn, contraction, AND expansion — this is revenue
-- retention, not logo retention, and it can exceed 100%.
-- ============================================================
CREATE TABLE cohort_revenue_retention (
    cohort_month         DATE    NOT NULL,
    months_since_signup  INTEGER NOT NULL,
    starting_cohort_mrr  NUMERIC NOT NULL,  -- cohort's MRR at age 0
    active_cohort_mrr    NUMERIC NOT NULL,  -- same cohort's MRR at this age
    PRIMARY KEY (cohort_month, months_since_signup)
);

INSERT INTO cohort_revenue_retention VALUES
-- January 2025 cohort (starting MRR 3200) — 12 data points, ages 0-11
('2025-01-01',0,3200,3200),('2025-01-01',1,3200,3066),('2025-01-01',2,3200,2970),
('2025-01-01',3,3200,2896),('2025-01-01',4,3200,2858),('2025-01-01',5,3200,2842),
('2025-01-01',6,3200,2854),('2025-01-01',7,3200,2886),('2025-01-01',8,3200,2938),
('2025-01-01',9,3200,3002),('2025-01-01',10,3200,3072),('2025-01-01',11,3200,3142),
-- February 2025 cohort (starting MRR 3400) — 11 data points, ages 0-10
('2025-02-01',0,3400,3400),('2025-02-01',1,3400,3257),('2025-02-01',2,3400,3155),
('2025-02-01',3,3400,3077),('2025-02-01',4,3400,3036),('2025-02-01',5,3400,3019),
('2025-02-01',6,3400,3033),('2025-02-01',7,3400,3067),('2025-02-01',8,3400,3121),
('2025-02-01',9,3400,3189),('2025-02-01',10,3400,3264),
-- March 2025 cohort (starting MRR 3600) — 10 data points, ages 0-9
('2025-03-01',0,3600,3600),('2025-03-01',1,3600,3449),('2025-03-01',2,3600,3341),
('2025-03-01',3,3600,3258),('2025-03-01',4,3600,3215),('2025-03-01',5,3600,3197),
('2025-03-01',6,3600,3211),('2025-03-01',7,3600,3247),('2025-03-01',8,3600,3305),
('2025-03-01',9,3600,3377),
-- April 2025 cohort (starting MRR 3100) — 9 data points, ages 0-8
('2025-04-01',0,3100,3100),('2025-04-01',1,3100,2970),('2025-04-01',2,3100,2877),
('2025-04-01',3,3100,2806),('2025-04-01',4,3100,2768),('2025-04-01',5,3100,2753),
('2025-04-01',6,3100,2765),('2025-04-01',7,3100,2796),('2025-04-01',8,3100,2846),
-- May 2025 cohort (starting MRR 3800) — 8 data points, ages 0-7
('2025-05-01',0,3800,3800),('2025-05-01',1,3800,3640),('2025-05-01',2,3800,3526),
('2025-05-01',3,3800,3439),('2025-05-01',4,3800,3393),('2025-05-01',5,3800,3374),
('2025-05-01',6,3800,3390),('2025-05-01',7,3800,3428),
-- June 2025 cohort (starting MRR 4000) — 7 data points, ages 0-6.
-- This cohort signed up right after Crunch Flow shipped a rebuilt
-- onboarding flow (June 2025) — watch its curve against May's.
('2025-06-01',0,4000,4000),('2025-06-01',1,4000,3880),('2025-06-01',2,4000,3800),
('2025-06-01',3,4000,3740),('2025-06-01',4,4000,3720),('2025-06-01',5,4000,3728),
('2025-06-01',6,4000,3760);
```

Sanity check — this should print `12` and `57`:

```sql
SELECT COUNT(*) FROM mrr_bridge_actuals;
SELECT COUNT(*) FROM cohort_revenue_retention;
```

Four facts to hold onto all week — you'll need every one of them:

- **The bridge reconciles by construction**: `ending_mrr = starting_mrr + new_mrr + expansion_mrr − contraction_mrr − churned_mrr`, and next month's `starting_mrr` always equals this month's `ending_mrr`. Exercise 1 makes you prove that in SQL before you trust anything downstream of it.
- **August 2025 is not a typo.** `new_mrr` drops from 4,000 (June) to 3,300 (July) to 2,600 (August) — a real summer slowdown — before rebounding hard to 4,300 in September. One year of monthly data can't *prove* a repeating seasonal pattern (you need at least two full cycles for that, statistically), but it's real signal you can't responsibly ignore either. Challenge 1 makes you decide what to do with it.
- **The cohorts are right-censored, and unevenly so.** The January cohort has 12 months of observed data; the June cohort has 7. That is not something you average away naively — Lecture 2 spends real time on why a straight column-average by age silently overweights the older, more mature cohorts, and what to do instead.
- **June 2025 is a deliberate before/after.** Crunch Flow shipped a rebuilt onboarding flow that month. Compare the June cohort's retention curve to May's at the same ages (age 3: June holds 93.5% vs. May's 90.5%) — that's not noise, and it changes which curve you should apply to *future* cohorts you haven't observed yet.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Setup + bottoms-up MRR bridge forecasting | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | MRR bridge drills — rates, `LAG`, forward projection | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Retention-adjusted projection, right-censoring | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Time-series fit, uncertainty, base/bull/bear | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | Challenges: seasonality + stress-testing assumptions | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (four-quarter forecast + memo) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it, and every SQL/pandas example targets the seed database you just created — the same one for the whole week.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-bottoms-up-mrr-forecast.md](./lecture-notes/01-bottoms-up-mrr-forecast.md) | The MRR bridge as a forecasting engine, computing forward rates with `LAG`, projecting month by month in SQL and pandas | 2h |
| 2 | [lecture-notes/02-retention-adjusted-projection.md](./lecture-notes/02-retention-adjusted-projection.md) | Cohort revenue-retention curves, right-censoring, the pooled/weighted curve, flat-continuation past the observed range | 2h |
| 3 | [lecture-notes/03-scenarios-and-uncertainty.md](./lecture-notes/03-scenarios-and-uncertainty.md) | A simple OLS time-series fit, residual-based prediction intervals, building base/bull/bear from named assumption deltas | 2h |
| 4 | [exercises/exercise-01-build-an-mrr-bridge.md](./exercises/exercise-01-build-an-mrr-bridge.md) | Validate the bridge reconciles; compute expansion/contraction/churn rates and net revenue retention per month | 1h |
| 5 | [exercises/exercise-02-project-retained-revenue.md](./exercises/exercise-02-project-retained-revenue.md) | Build the pooled retention-by-age curve and project unobserved cohorts forward with it | 1h |
| 6 | [exercises/exercise-03-three-scenario-model.md](./exercises/exercise-03-three-scenario-model.md) | Build the base/bull/bear bridge forecast for Q1–Q4 2026 from given assumption deltas | 1h |
| 7 | [challenges/challenge-01-forecast-with-seasonality.md](./challenges/challenge-01-forecast-with-seasonality.md) | Decide how (and how much) to bend the forecast around the August dip with only one year of history | 1h |
| 8 | [challenges/challenge-02-stress-test-assumptions.md](./challenges/challenge-02-stress-test-assumptions.md) | A finance-approved forecast has a fragile assumption buried in it — find it, quantify the exposure, fix it | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Full next-four-quarter forecast — bottoms-up, retention-adjusted, base/bull/bear — plus the assumption memo | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Build a bottoms-up MRR bridge forecast in SQL and pandas, with every rate assumption named and visible.
- Turn a set of right-censored cohort curves into a defensible, flat-continued projection of retained revenue — and explain *why* you didn't extrapolate the curve's trend indefinitely.
- Fit a simple linear time series to a revenue history, and put an honest, widening uncertainty band around a point forecast instead of presenting a single number as fact.
- Build base, bull, and bear scenarios from named assumption deltas, and roll them into a quarterly table a CFO can act on.
- Write an assumption memo that states the method, the numbers, and the range — and survives someone asking "why?" three times in a row.

## Up next

[Week 12 — Capstone: an end-to-end growth system](../week-12-capstone/) — everything this course taught, shipped as one warehouse-backed system with metrics, tests, and a forecast that updates itself.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
