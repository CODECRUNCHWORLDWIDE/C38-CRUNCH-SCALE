# Week 9 — Pricing & Packaging

> **Goal:** by Sunday you can take a mispriced, one-size-fits-all product, prove — with a willingness-to-pay survey, a segmented tier design, and a price-sensitivity model, all in SQL and pandas — exactly how much money is being left on the table, and forecast the net revenue impact (churn *and* expansion, not just the sticker price) of fixing it.

Welcome back to **C38 · Crunch Scale**. Every week so far has measured the business as it *is*: the funnel, the activation flow, the retention curve, the unit economics, the warehouse that keeps the numbers honest. This week you get to change one of the numbers on purpose. Pricing is the single highest-leverage lever in a subscription business — a small, well-reasoned price change moves more profit than most growth teams' entire acquisition budget — and it's also the lever operators are most afraid to touch, because it's the one place where the customer feels the decision directly. The fix for that fear isn't confidence, it's evidence: a willingness-to-pay estimate instead of a guess, a tier design that matches what different customers actually value, and a forecast that accounts for churn *and* expansion before you ship the change, not after.

This week's company is **ScopeIQ** — a product-analytics and session-replay tool for early-stage SaaS teams (think: "why did this user bounce," heatmaps, funnels, replay). ScopeIQ shipped fast and priced lazily: **one flat plan, $39/month, unlimited usage**, sold to everyone from a solo indie hacker tracking 300 monthly users to a 40-person agency tracking 145,000. That single sentence is this week's entire curriculum in miniature — a flat price on a product with wildly different value delivered per customer is *always* a pricing bug, and by Friday you will have found it, fixed it, and forecast the fix.

You load four small tables once (below), and every lecture, exercise, challenge, and the mini-project queries them. **No spreadsheet touches any of this — that is a hard rule of this course.** Data lives in SQL and pandas. Spreadsheets are taught only in C41 Crunch Excel, and never as a data store.

## Learning objectives

By the end of this week, you will be able to:

- **Measure** willingness to pay with the Van Westendorp Price Sensitivity Meter (survey-based) and behavioral WTP signals (usage-based) — and know when to trust each.
- **Design** Good-Better-Best pricing tiers around a defensible **value metric**, and allocate features across tiers without cannibalizing your best customers or bill-shocking your smallest ones.
- **Model** price sensitivity and elasticity from real conversion data, fit a demand curve in SQL/pandas, and find the revenue-maximizing price analytically.
- **Reason** precisely about discounts, grandfathering periods, and **expansion revenue** — the three levers that separate a naive "multiply by the new price" forecast from one a CFO will actually believe.
- **Forecast** the net revenue impact of a price change *before* shipping it — combining new-signup elasticity, existing-customer churn elasticity, and expansion revenue into one written recommendation, backed by the query that proves it.

## Prerequisites

- Comfortable with SQL `SELECT`, `WHERE`, `GROUP BY`, aggregates, and `CASE` — [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) or equivalent.
- Comfortable with pandas `groupby`, basic arithmetic on a `DataFrame`, and simple linear regression (`numpy.polyfit` or `numpy.linalg.lstsq` — Lecture 3 walks through it from scratch, no prior stats course assumed).
- Week 5 of this course (LTV, CAC & unit economics) is not a hard requirement, but the phrase "contribution margin" will feel familiar if you've done it — this week reuses that instinct for a different question: not "is this customer worth acquiring" but "are we charging this customer the right amount."
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` and `numpy` installed. Install steps are in [`resources.md`](./resources.md). **Data lives in SQL and pandas this week — never in a spreadsheet.**

## Set up the seed database (do this first)

Everything this week runs against four tables, all describing **ScopeIQ**. Create them once.

**PostgreSQL:**

```bash
createdb crunch_scale_w9
psql crunch_scale_w9
```

**SQLite:**

```bash
sqlite3 crunch_scale_w9.db
```

Then paste this into the shell (it works unchanged on both engines):

```sql
-- ── 1. wtp_survey: Van Westendorp Price Sensitivity Meter, 30 respondents ──
-- Each respondent named four prices for ScopeIQ, in increasing order:
--   too_cheap    — "so cheap I'd doubt the quality"
--   cheap        — "a bargain — a great buy for the money"
--   expensive    — "starting to get expensive, but I'd still consider it"
--   too_expensive — "so expensive I would not buy it"
CREATE TABLE wtp_survey (
    respondent_id  INTEGER PRIMARY KEY,
    segment        TEXT    NOT NULL,   -- 'indie' | 'team' | 'agency'
    too_cheap      NUMERIC NOT NULL,
    cheap          NUMERIC NOT NULL,
    expensive      NUMERIC NOT NULL,
    too_expensive  NUMERIC NOT NULL
);

INSERT INTO wtp_survey VALUES
(1,'indie',8,14,22,30),
(2,'indie',9,15,25,34),
(3,'indie',10,17,28,38),
(4,'indie',11,19,31,42),
(5,'indie',12,20,34,46),
(6,'indie',13,22,36,49),
(7,'indie',14,24,39,53),
(8,'indie',15,26,42,57),
(9,'indie',16,27,45,61),
(10,'indie',18,31,50,68),
(11,'team',20,34,54,72),
(12,'team',22,37,59,79),
(13,'team',24,41,65,86),
(14,'team',26,44,70,94),
(15,'team',28,48,76,101),
(16,'team',30,51,81,108),
(17,'team',32,54,86,115),
(18,'team',34,58,92,122),
(19,'team',36,61,97,130),
(20,'team',39,66,105,140),
(21,'agency',55,88,138,182),
(22,'agency',60,96,150,198),
(23,'agency',65,104,162,214),
(24,'agency',70,112,175,231),
(25,'agency',75,120,188,248),
(26,'agency',80,128,200,264),
(27,'agency',85,136,212,280),
(28,'agency',90,144,225,297),
(29,'agency',95,152,238,314),
(30,'agency',102,163,255,337);

-- ── 2. accounts_usage: the SAME 30 accounts, but their real usage ──
-- current_price is the flat $39 everyone pays today, regardless of usage.
-- mtu = Monthly Tracked Users — the candidate value metric for this week.
CREATE TABLE accounts_usage (
    account_id     INTEGER PRIMARY KEY,
    segment        TEXT    NOT NULL,
    mtu            INTEGER NOT NULL,
    current_price  NUMERIC NOT NULL
);

INSERT INTO accounts_usage VALUES
(1,'indie',300,39.00),
(2,'indie',450,39.00),
(3,'indie',600,39.00),
(4,'indie',800,39.00),
(5,'indie',950,39.00),
(6,'indie',1200,39.00),
(7,'indie',1500,39.00),
(8,'indie',1800,39.00),
(9,'indie',2200,39.00),
(10,'indie',2900,39.00),
(11,'team',4200,39.00),
(12,'team',5800,39.00),
(13,'team',7500,39.00),
(14,'team',9800,39.00),
(15,'team',12500,39.00),
(16,'team',15800,39.00),
(17,'team',19200,39.00),
(18,'team',22500,39.00),
(19,'team',26000,39.00),
(20,'team',29500,39.00),
(21,'agency',35000,39.00),
(22,'agency',42000,39.00),
(23,'agency',51000,39.00),
(24,'agency',63000,39.00),
(25,'agency',75000,39.00),
(26,'agency',88000,39.00),
(27,'agency',102000,39.00),
(28,'agency',118000,39.00),
(29,'agency',131000,39.00),
(30,'agency',145000,39.00);

-- ── 3. price_experiment: a 6-month randomized price test on the signup page ──
-- Prospects were shown one of 5 prices at random when requesting a demo/quote;
-- quotes_shown = sample size, conversions = how many became paying customers.
CREATE TABLE price_experiment (
    price        NUMERIC NOT NULL,
    quotes_shown INTEGER NOT NULL,
    conversions  INTEGER NOT NULL
);

INSERT INTO price_experiment VALUES
(29,400,88),
(35,400,72),
(39,400,62),
(45,400,48),
(55,400,28);

-- ── 4. subscription_base: ScopeIQ's full installed base, by tier, ──
-- AFTER the Week 9 repackaging (Starter/Growth/Scale) has already shipped.
-- This is the table Friday's price-increase forecast runs against.
CREATE TABLE subscription_base (
    tier            TEXT    PRIMARY KEY,
    customer_count  INTEGER NOT NULL,
    current_price   NUMERIC NOT NULL,
    monthly_signups NUMERIC NOT NULL   -- avg new customers/month at current_price
);

INSERT INTO subscription_base VALUES
('Starter',1150,29.00,85),
('Growth',640,69.00,40),
('Scale',85,179.00,4);
```

Sanity check — this should print `30`, `30`, `5`, `3`:

```sql
SELECT COUNT(*) FROM wtp_survey;
SELECT COUNT(*) FROM accounts_usage;
SELECT COUNT(*) FROM price_experiment;
SELECT COUNT(*) FROM subscription_base;
```

Four facts to hold onto all week — you'll need every one of them:

- **`wtp_survey` and `accounts_usage` describe the same 30 customers**, `respondent_id` = `account_id`. The person who told you their price thresholds is the same account whose real usage you can see. That link is deliberate — it's how you'll check whether stated willingness to pay matches revealed usage-based value.
- **The three segments are not the tiers — yet.** `indie`/`team`/`agency` are informal labels for "who this customer is," useful for building intuition. `mtu` (Monthly Tracked Users) is the actual **value metric** you'll price against once you formalize tiers in Lecture 2 — a customer is a `Starter` or a `Growth` account because of their usage, not their label.
- **`price_experiment` measures NEW-customer conversion**, at the moment a prospect first sees a price on a page. It tells you nothing directly about how an *existing*, paying, dependent customer reacts to a price increase — those two elasticities are usually very different, and Lecture 3 spends real time on why.
- **`subscription_base` is a flash-forward.** It reflects ScopeIQ's installed base *after* the Starter/Growth/Scale tiers from Lecture 2 are already live at scale (1,150 + 640 + 85 = 1,875 paying customers) — not the 30-account illustrative set. Friday's forecast operates at real portfolio scale on purpose: a 30-account sample is fine for *designing* a tier, but you forecast revenue impact on the number that's actually on the books.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Setup + willingness to pay (Van Westendorp, behavioral) | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | WTP drills — segment curves, PMC/PME/OPP by hand | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Tiers & packaging — value metric, Good-Better-Best | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Modeling a price change — elasticity, SQL + pandas | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | Challenges: repackage a segment + forecast an increase | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (WTP → tiers → price-change forecast) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-willingness-to-pay.md](./lecture-notes/01-willingness-to-pay.md) | Van Westendorp PSM end to end (TC/B/GE/TE curves, PMC/PME/OPP), plus behavioral WTP signals | 2h |
| 2 | [lecture-notes/02-tiers-and-packaging.md](./lecture-notes/02-tiers-and-packaging.md) | Value metrics, Good-Better-Best design, feature allocation, avoiding cliffs and cannibalization | 2h |
| 3 | [lecture-notes/03-modeling-a-price-change.md](./lecture-notes/03-modeling-a-price-change.md) | Demand curves, elasticity, discounts/grandfathering, expansion revenue, the full forecast model | 2h |
| 4 | [exercises/exercise-01-wtp-from-survey.md](./exercises/exercise-01-wtp-from-survey.md) | Build all four Van Westendorp curves and the segment WTP ranges yourself | 1h |
| 5 | [exercises/exercise-02-design-three-tiers.md](./exercises/exercise-02-design-three-tiers.md) | Map 30 accounts into Starter/Growth/Scale and compute the MRR impact | 1h |
| 6 | [exercises/exercise-03-price-sensitivity-curve.md](./exercises/exercise-03-price-sensitivity-curve.md) | Fit a demand curve to `price_experiment`, find the revenue-maximizing price, compute elasticity | 1h |
| 7 | [challenges/challenge-01-repackage-for-a-segment.md](./challenges/challenge-01-repackage-for-a-segment.md) | Design an SSO add-on so `team` accounts don't have to jump to `Scale` for one feature | 1h |
| 8 | [challenges/challenge-02-forecast-a-price-increase.md](./challenges/challenge-02-forecast-a-price-increase.md) | Forecast raising the Growth tier $69→$79 across a 1,875-customer base, with grandfathering | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Full pricing-change model — WTP → tiers → forecast with churn and expansion — plus a written recommendation | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Turn a stack of survey responses into four Van Westendorp curves and read off PMC, PME, OPP, and IDP without a tool doing it for you.
- Choose a value metric, design three tiers around it, and explain in one sentence why each feature sits in the tier it sits in.
- Fit a demand curve to real conversion data, find the revenue-maximizing price with calculus, and state the price elasticity at any point on the curve.
- Explain, precisely, why new-customer conversion elasticity and existing-customer churn elasticity are different numbers — and why that difference is *the* reason companies can raise prices on existing customers even when they wouldn't dare raise the price on the signup page.
- Forecast the net MRR impact of a price change — churn, new signups, and expansion revenue together — and defend a ship/hold/redesign recommendation with the query that proves it.

## Up next

Week 10 — Experimentation & Causal Inference — now that you can price a product with evidence instead of instinct, we build the A/B testing and causal-inference toolkit that validates *any* growth change, pricing included, before it ships to everyone.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
