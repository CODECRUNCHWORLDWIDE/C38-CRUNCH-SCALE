# Week 8 — Experimentation & A/B Testing

> **Goal:** by Sunday you can take a growth idea from a one-sentence hunch to a shipped-or-killed decision — write a falsifiable hypothesis with one primary metric and a guardrail, size the test correctly *before* a single visitor is bucketed, randomize cleanly, read the result honestly (significance **and** a confidence interval, not just a p-value), and name the specific way a test in front of you is lying — all in SQL and pandas, never a spreadsheet.

Every other week in this course has taught you to *measure* the business — funnels, retention, LTV, a trustworthy warehouse. This week teaches you to *change* it, and to prove the change is what moved the number, not luck, seasonality, or a bug in your own analysis. That distinction — correlation dressed up as causation — is the single most common way growth teams fool themselves, and it is usually invisible until someone asks the right question in a launch review. This week you become the person who asks it, on your own tests first.

The company is **LoopCart**, a direct-to-consumer e-commerce storefront. LoopCart's current checkout is three pages (cart → shipping → payment). The growth team has a pitch: collapse it into one page, **Express Checkout**, and conversion will go up. You'll run that exact experiment this week — design it, size it, read a real (if modest-scale) result, and learn the five specific ways a test like this goes wrong before you ever touch your own company's traffic.

## Learning objectives

By the end of this week, you will be able to:

- **Frame** an experiment with a single, falsifiable hypothesis, one **primary metric** (the number that decides ship/no-ship) and at least one **guardrail metric** (a number that must not get worse, even if the primary metric wins).
- **Compute** the required sample size and test duration from a baseline rate, a minimum detectable effect (MDE), a significance level (α), and statistical power (1 − β) — in plain Python, no black-box calculator required.
- **Run** a randomized split correctly — choosing the right randomization unit, checking for **sample-ratio mismatch (SRM)** with a chi-square goodness-of-fit test before trusting anything else the test tells you.
- **Compute** statistical significance (a two-proportion z-test) and a 95% confidence interval for the difference, and explain in one sentence why "statistically significant" and "a big effect" are not the same claim.
- **Recognize**, by name and by SQL query, the five traps that invalidate a test result that *looks* clean: **peeking**, **sample-ratio mismatch**, **novelty effects**, **multiple comparisons**, and **Simpson's paradox** — and know the specific defense against each.

## Prerequisites

- **C33 Crunch SQL** (or equivalent): `SELECT`, joins, `GROUP BY`/aggregates, `CASE`, window functions.
- **Python with pandas** (`pip install pandas numpy`) — every significance calculation this week is done in plain Python/pandas, not a stats package black box, so you can see exactly what the formula does.
- No prior statistics course required. This week derives the two-proportion z-test, confidence intervals, and the chi-square test from first principles — you don't need to have seen them before, but you do need to be comfortable with basic algebra and square roots.
- Weeks 1–7 of this course are helpful context (you already know what a north-star metric and a funnel are) but **not required** — this week's dataset stands on its own.
- PostgreSQL 16+ (primary) or SQLite 3.35+ (fallback). No spreadsheet software anywhere this week — **that is a hard rule of this course.** Spreadsheets are taught only in C41 Crunch Excel, and never as a data store or a stats engine.

## Set up the seed database (do this first)

LoopCart ran a **two-week pilot** of Express Checkout before committing engineering time to a full rollout. Visitors who reached the cart page were randomly bucketed into `control` (the existing 3-step checkout) or `treatment` (Express Checkout) the first time they hit the cart, and stayed in that bucket for the whole session (sticky, session-level randomization — more on why that matters in Lecture 1).

**PostgreSQL:**

```bash
createdb crunch_scale_w8
psql crunch_scale_w8
```

**SQLite:**

```bash
sqlite3 crunch_scale_w8.db
```

Paste this into the shell (unchanged on both engines):

```sql
CREATE TABLE checkout_sessions (
    session_id      INTEGER PRIMARY KEY,
    visitor_id      INTEGER NOT NULL,   -- the randomization unit: one row per session, one visitor per bucket
    variant         TEXT    NOT NULL,   -- 'control' (3-step checkout) or 'treatment' (Express Checkout)
    device          TEXT    NOT NULL,   -- 'mobile' or 'desktop'
    country         TEXT    NOT NULL,
    assigned_at     TIMESTAMP NOT NULL, -- when the visitor was bucketed (start of the cart session)
    converted       BOOLEAN NOT NULL,   -- TRUE if this session ended in a completed purchase
    order_value_usd NUMERIC,            -- NULL if not converted
    refunded        BOOLEAN             -- NULL if not converted; TRUE/FALSE once we know
);

INSERT INTO checkout_sessions VALUES
(1,1001,'control','mobile','UK','2026-06-02 09:00:00',TRUE,66,FALSE),
(2,1002,'control','mobile','Canada','2026-06-03 10:00:00',FALSE,NULL,NULL),
(3,1003,'control','desktop','Germany','2026-06-04 11:00:00',TRUE,78,FALSE),
(4,1004,'control','desktop','Australia','2026-06-05 12:00:00',FALSE,NULL,NULL),
(5,1005,'control','mobile','USA','2026-06-06 13:00:00',TRUE,90,FALSE),
(6,1006,'control','mobile','UK','2026-06-07 14:00:00',FALSE,NULL,NULL),
(7,1007,'control','mobile','Canada','2026-06-01 15:00:00',TRUE,60,FALSE),
(8,1008,'control','desktop','Germany','2026-06-02 16:00:00',FALSE,NULL,NULL),
(9,1009,'control','desktop','Australia','2026-06-03 17:00:00',TRUE,72,FALSE),
(10,1010,'control','mobile','USA','2026-06-04 08:00:00',FALSE,NULL,NULL),
(11,1011,'control','mobile','UK','2026-06-05 09:00:00',TRUE,84,TRUE),
(12,1012,'control','mobile','Canada','2026-06-06 10:00:00',FALSE,NULL,NULL),
(13,1013,'control','desktop','Germany','2026-06-07 11:00:00',TRUE,96,FALSE),
(14,1014,'control','desktop','Australia','2026-06-01 12:00:00',FALSE,NULL,NULL),
(15,1015,'control','mobile','USA','2026-06-02 13:00:00',TRUE,66,FALSE),
(16,1016,'control','mobile','UK','2026-06-03 14:00:00',FALSE,NULL,NULL),
(17,1017,'control','mobile','Canada','2026-06-04 15:00:00',TRUE,78,FALSE),
(18,1018,'control','desktop','Germany','2026-06-05 16:00:00',FALSE,NULL,NULL),
(19,1019,'control','desktop','Australia','2026-06-06 17:00:00',TRUE,90,FALSE),
(20,1020,'control','mobile','USA','2026-06-07 08:00:00',FALSE,NULL,NULL),
(21,1021,'control','mobile','UK','2026-06-01 09:00:00',TRUE,60,FALSE),
(22,1022,'control','mobile','Canada','2026-06-02 10:00:00',FALSE,NULL,NULL),
(23,1023,'control','desktop','Germany','2026-06-03 11:00:00',TRUE,72,FALSE),
(24,1024,'control','desktop','Australia','2026-06-04 12:00:00',FALSE,NULL,NULL),
(25,1025,'control','mobile','USA','2026-06-05 13:00:00',TRUE,84,FALSE),
(26,1026,'control','mobile','UK','2026-06-06 14:00:00',FALSE,NULL,NULL),
(27,1027,'control','mobile','Canada','2026-06-07 15:00:00',TRUE,96,FALSE),
(28,1028,'control','desktop','Germany','2026-06-01 16:00:00',FALSE,NULL,NULL),
(29,1029,'control','desktop','Australia','2026-06-02 17:00:00',TRUE,66,FALSE),
(30,1030,'control','mobile','USA','2026-06-03 08:00:00',FALSE,NULL,NULL),
(31,1031,'control','mobile','UK','2026-06-04 09:00:00',TRUE,78,FALSE),
(32,1032,'control','mobile','Canada','2026-06-05 10:00:00',FALSE,NULL,NULL),
(33,1033,'control','desktop','Germany','2026-06-06 11:00:00',TRUE,90,TRUE),
(34,1034,'control','desktop','Australia','2026-06-07 12:00:00',FALSE,NULL,NULL),
(35,1035,'control','mobile','USA','2026-06-01 13:00:00',TRUE,60,FALSE),
(36,1036,'control','mobile','UK','2026-06-02 14:00:00',FALSE,NULL,NULL),
(37,1037,'control','mobile','Canada','2026-06-03 15:00:00',TRUE,72,FALSE),
(38,1038,'control','desktop','Germany','2026-06-04 16:00:00',FALSE,NULL,NULL),
(39,1039,'control','desktop','Australia','2026-06-05 17:00:00',TRUE,84,FALSE),
(40,1040,'control','mobile','USA','2026-06-06 08:00:00',FALSE,NULL,NULL),
(41,1041,'control','mobile','UK','2026-06-07 09:00:00',TRUE,96,FALSE),
(42,1042,'control','mobile','Canada','2026-06-01 10:00:00',FALSE,NULL,NULL),
(43,1043,'control','desktop','Germany','2026-06-02 11:00:00',FALSE,NULL,NULL),
(44,1044,'control','desktop','Australia','2026-06-03 12:00:00',FALSE,NULL,NULL),
(45,1045,'control','mobile','USA','2026-06-04 13:00:00',FALSE,NULL,NULL),
(46,1046,'control','mobile','UK','2026-06-05 14:00:00',FALSE,NULL,NULL),
(47,1047,'control','mobile','Canada','2026-06-06 15:00:00',FALSE,NULL,NULL),
(48,1048,'control','desktop','Germany','2026-06-07 16:00:00',FALSE,NULL,NULL),
(49,1049,'control','desktop','Australia','2026-06-01 17:00:00',FALSE,NULL,NULL),
(50,1050,'control','mobile','USA','2026-06-02 08:00:00',FALSE,NULL,NULL),
(51,1051,'treatment','mobile','UK','2026-06-03 09:00:00',TRUE,80,FALSE),
(52,1052,'treatment','mobile','Canada','2026-06-04 10:00:00',TRUE,85,FALSE),
(53,1053,'treatment','desktop','Germany','2026-06-05 11:00:00',TRUE,90,FALSE),
(54,1054,'treatment','desktop','Australia','2026-06-06 12:00:00',TRUE,50,FALSE),
(55,1055,'treatment','mobile','USA','2026-06-07 13:00:00',TRUE,55,FALSE),
(56,1056,'treatment','mobile','UK','2026-06-01 14:00:00',TRUE,60,TRUE),
(57,1057,'treatment','mobile','Canada','2026-06-02 15:00:00',TRUE,65,FALSE),
(58,1058,'treatment','desktop','Germany','2026-06-03 16:00:00',TRUE,70,FALSE),
(59,1059,'treatment','desktop','Australia','2026-06-04 17:00:00',TRUE,75,FALSE),
(60,1060,'treatment','mobile','USA','2026-06-05 08:00:00',TRUE,80,FALSE),
(61,1061,'treatment','mobile','UK','2026-06-06 09:00:00',TRUE,85,FALSE),
(62,1062,'treatment','mobile','Canada','2026-06-07 10:00:00',TRUE,90,FALSE),
(63,1063,'treatment','desktop','Germany','2026-06-01 11:00:00',TRUE,50,TRUE),
(64,1064,'treatment','desktop','Australia','2026-06-02 12:00:00',TRUE,55,FALSE),
(65,1065,'treatment','mobile','USA','2026-06-03 13:00:00',TRUE,60,FALSE),
(66,1066,'treatment','mobile','UK','2026-06-04 14:00:00',TRUE,65,FALSE),
(67,1067,'treatment','mobile','Canada','2026-06-05 15:00:00',TRUE,70,FALSE),
(68,1068,'treatment','desktop','Germany','2026-06-06 16:00:00',TRUE,75,FALSE),
(69,1069,'treatment','desktop','Australia','2026-06-07 17:00:00',TRUE,80,FALSE),
(70,1070,'treatment','mobile','USA','2026-06-01 08:00:00',TRUE,85,TRUE),
(71,1071,'treatment','mobile','UK','2026-06-02 09:00:00',TRUE,90,FALSE),
(72,1072,'treatment','mobile','Canada','2026-06-03 10:00:00',TRUE,50,FALSE),
(73,1073,'treatment','desktop','Germany','2026-06-04 11:00:00',TRUE,55,FALSE),
(74,1074,'treatment','desktop','Australia','2026-06-05 12:00:00',TRUE,60,FALSE),
(75,1075,'treatment','mobile','USA','2026-06-06 13:00:00',TRUE,65,FALSE),
(76,1076,'treatment','mobile','UK','2026-06-07 14:00:00',TRUE,70,FALSE),
(77,1077,'treatment','mobile','Canada','2026-06-01 15:00:00',TRUE,75,TRUE),
(78,1078,'treatment','desktop','Germany','2026-06-02 16:00:00',TRUE,80,FALSE),
(79,1079,'treatment','desktop','Australia','2026-06-03 17:00:00',TRUE,85,FALSE),
(80,1080,'treatment','mobile','USA','2026-06-04 08:00:00',TRUE,90,FALSE),
(81,1081,'treatment','mobile','UK','2026-06-05 09:00:00',TRUE,50,FALSE),
(82,1082,'treatment','mobile','Canada','2026-06-06 10:00:00',TRUE,55,FALSE),
(83,1083,'treatment','desktop','Germany','2026-06-07 11:00:00',FALSE,NULL,NULL),
(84,1084,'treatment','desktop','Australia','2026-06-01 12:00:00',FALSE,NULL,NULL),
(85,1085,'treatment','mobile','USA','2026-06-02 13:00:00',FALSE,NULL,NULL),
(86,1086,'treatment','mobile','UK','2026-06-03 14:00:00',FALSE,NULL,NULL),
(87,1087,'treatment','mobile','Canada','2026-06-04 15:00:00',FALSE,NULL,NULL),
(88,1088,'treatment','desktop','Germany','2026-06-05 16:00:00',FALSE,NULL,NULL),
(89,1089,'treatment','desktop','Australia','2026-06-06 17:00:00',FALSE,NULL,NULL),
(90,1090,'treatment','mobile','USA','2026-06-07 08:00:00',FALSE,NULL,NULL),
(91,1091,'treatment','mobile','UK','2026-06-01 09:00:00',FALSE,NULL,NULL),
(92,1092,'treatment','mobile','Canada','2026-06-02 10:00:00',FALSE,NULL,NULL),
(93,1093,'treatment','desktop','Germany','2026-06-03 11:00:00',FALSE,NULL,NULL),
(94,1094,'treatment','desktop','Australia','2026-06-04 12:00:00',FALSE,NULL,NULL),
(95,1095,'treatment','mobile','USA','2026-06-05 13:00:00',FALSE,NULL,NULL),
(96,1096,'treatment','mobile','UK','2026-06-06 14:00:00',FALSE,NULL,NULL),
(97,1097,'treatment','mobile','Canada','2026-06-07 15:00:00',FALSE,NULL,NULL),
(98,1098,'treatment','desktop','Germany','2026-06-01 16:00:00',FALSE,NULL,NULL),
(99,1099,'treatment','desktop','Australia','2026-06-02 17:00:00',FALSE,NULL,NULL),
(100,1100,'treatment','mobile','USA','2026-06-03 08:00:00',FALSE,NULL,NULL);
```

Sanity checks — these should print `100`, `50`, `50`, `53`:

```sql
SELECT COUNT(*) FROM checkout_sessions;
SELECT COUNT(*) FROM checkout_sessions WHERE variant = 'control';
SELECT COUNT(*) FROM checkout_sessions WHERE variant = 'treatment';
SELECT COUNT(*) FROM checkout_sessions WHERE converted;
```

This is deliberately **pilot-scale** — 50 sessions per arm, not the thousands a real launch decision needs. That's on purpose: Lecture 2 will show you exactly how to compute whether 50-per-arm was ever going to be enough, *before* you trust anything this table tells you. A few things are baked in on purpose, and you'll need all of them this week:

1. **The lift is large and easy to see** (42% → 64% conversion) — real MDEs are almost never this generous. This dataset is sized to make the arithmetic legible by hand; Exercise 1 makes you size a *realistic* test (single-digit percentage-point lift) and watch the required sample size explode.
2. **`order_value_usd` and `refunded` are only populated for converted sessions** — `NULL` means "never got that far," not "spent $0." Don't confuse the two.
3. **`assigned_at` spans a full calendar week** (2026-06-01 through 2026-06-07) — Homework Problem 3 uses this to show you exactly what "peeking" does to a result across the days of a live test.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed data; hypotheses, primary + guardrail | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Power, MDE, sample-size math | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Significance, confidence intervals | 0h | 1.5h | 1h | 0.5h | 1h | 0h | 4.5h |
| Thursday | Experimentation traps (peeking, SRM, novelty) | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Simpson's paradox, multiple comparisons; challenges | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (design + analyze a full test) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4h** | **3h** | **3.5h** | **5h** | **5h** | **28.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it, and every worked example runs against the `checkout_sessions` seed table unless a file says otherwise.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-designing-an-experiment.md](./lecture-notes/01-designing-an-experiment.md) | Hypothesis structure, primary vs. guardrail metrics, randomization units, pre-launch pitfalls | 2h |
| 2 | [lecture-notes/02-power-and-significance.md](./lecture-notes/02-power-and-significance.md) | MDE, α, power, sample-size math, the two-proportion z-test, confidence intervals | 2h |
| 3 | [lecture-notes/03-experimentation-traps.md](./lecture-notes/03-experimentation-traps.md) | Peeking, SRM, novelty effects, multiple comparisons, Simpson's paradox | 2h |
| 4 | [exercises/exercise-01-size-a-test.md](./exercises/exercise-01-size-a-test.md) | Compute required sample size and duration for a realistic MDE | 1.5h |
| 5 | [exercises/exercise-02-analyze-a-result.md](./exercises/exercise-02-analyze-a-result.md) | Full significance + CI + guardrail read of the `checkout_sessions` pilot | 1.5h |
| 6 | [exercises/exercise-03-detect-srm.md](./exercises/exercise-03-detect-srm.md) | Chi-square goodness-of-fit test to catch a broken randomizer | 1h |
| 7 | [challenges/challenge-01-design-a-full-experiment.md](./challenges/challenge-01-design-a-full-experiment.md) | Design LoopCart's next test end to end, from a vague idea to a launch-ready spec | 1.5h |
| 8 | [challenges/challenge-02-catch-a-broken-test.md](./challenges/challenge-02-catch-a-broken-test.md) | A "clean win" report has 4 baked-in invalidating bugs — find all of them | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Design, size, generate, and analyze a full A/B test end to end; make a ship/no-ship call | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice: peeking across real calendar days, a second sizing drill, a t-test on the guardrail | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Write an experiment brief — hypothesis, primary metric, guardrail metric, randomization unit — that a skeptical engineering lead would actually approve before writing code.
- Compute, from four numbers (baseline rate, MDE, α, power), exactly how many visitors and how many days a test needs — before it launches, not after someone asks why it's still running.
- Run a two-proportion z-test and a 95% confidence interval by hand, in SQL, and in pandas, and know which of "significant," "large," and "precisely estimated" a given result actually is.
- Spot peeking, SRM, a novelty effect, an uncorrected multiple-comparisons problem, or a Simpson's-paradox reversal in someone else's "ship it" report — and say, precisely, why it invalidates the conclusion.
- Take a growth idea from zero to a defensible ship/no-ship recommendation, entirely in SQL and pandas.

## Up next

[Week 9 — Pricing, packaging & price sensitivity](../week-09-pricing-and-packaging/) — now that you can prove a change caused a lift, you'll use that same machinery to test what customers will actually pay.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
