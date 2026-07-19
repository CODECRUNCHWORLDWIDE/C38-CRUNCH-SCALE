# Week 12 — Capstone — The Growth System

> **Goal:** by Sunday you have shipped one coherent, queryable growth system — acquisition funnel, retention cohorts, unit economics, a designed-and-analyzed experiment, and a defended revenue forecast — all built on a warehouse you stood up yourself, with every number in your final deck traceable to a `SELECT` you can re-run on demand.

Eleven weeks of this course taught you one skill at a time: instrument events, build funnels, diagnose activation, model retention, price a customer's lifetime value, warehouse the data properly, segment the base, run an experiment, price a product, score churn, forecast growth. Week 12 doesn't teach a twelfth skill. It asks you to **run the company** — pick a growth question that actually matters, build the warehouse that can answer it, and answer it end to end, in public, with numbers you'd defend to a board.

This is the last week of **C38 · Crunch Scale**, and it is graded differently than every week before it. There is no new SQL syntax. There is no new statistical test. The bar is **integration and judgment**: can you assemble everything you've learned into one system, and can you say — out loud, backed by a query — what you'd do next and why.

## Learning objectives

By the end of this week, you will be able to:

- **Model** a customer data warehouse from raw events through analytics marts — the same raw → staging → marts shape from Week 6, applied to a growth dataset you scope and stand up yourself.
- **Build** the full funnel in one system: acquisition by channel, activation, retention cohorts, and monthly revenue, all backed by mart tables a stakeholder could query without you in the room.
- **Design and analyze** at least one experiment against the system, and state plainly whether the result is strong enough to act on.
- **Model** unit economics (CAC, contribution-margin LTV, payback, LTV:CAC) for the acquisition channels in your warehouse, and use them to make a real budget call.
- **Forecast** growth with at least two independent methods, reconcile where they disagree, and present the whole system as a decision — not a dashboard — with every claim traceable to a query.

## Prerequisites

- Comfortable with everything in Weeks 1–11 of this course: funnels and attribution (Wk 2), activation (Wk 3), retention and cohorts (Wk 4), LTV/CAC/unit economics (Wk 5), the raw → staging → marts warehouse pattern (Wk 6), segmentation (Wk 7), experiment design and significance testing (Wk 8), pricing (Wk 9), lifecycle scoring (Wk 10), and forecasting methods (Wk 11).
- SQL fluency from [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) — joins, aggregation, CTEs, window functions.
- Comfortable in pandas for the parts of the analysis that are cleaner in Python than SQL (regression fits, plotting the forecast comparison).
- PostgreSQL 16+ or SQLite 3.35+ — same as every other week. **Every number in this week lives in SQL or pandas. No spreadsheet, at any point.**

## The capstone dataset — Crunch Boards

This week's warehouse is a fictional but fully worked B2B SaaS company: **Crunch Boards**, a lightweight project-management tool. You seed it once, here, and every lecture, exercise, challenge, and the mini-project builds on top of the same data. It is deliberately **smaller and messier** than earlier weeks' seed tables — 72 signups, not 300 — because that is what a real early-stage company's data actually looks like, and defending a number built on a thin sample is exactly the judgment this capstone is testing.

**Company shape:** three acquisition channels (`paid_search`, `organic`, `referral`), six months of signups (January–June 2025), an onboarding-checklist experiment run on the two most recent cohorts, and a monthly-subscription business on three plans (Starter $29, Growth $99, Scale $299). **Analysis cutoff: 2025-06-30.** A few June signups convert to paying customers in early July — you'll meet them in Lecture 1, and they are there on purpose.

### Set up the seed database (do this first)

**PostgreSQL:**

```bash
createdb crunch_boards
psql crunch_boards
```

**SQLite:**

```bash
sqlite3 crunch_boards.db
```

Then paste this into the shell — it works unchanged on both engines:

```sql
CREATE TABLE raw_users (
    user_id        INTEGER PRIMARY KEY,
    email          TEXT    NOT NULL,
    company_name   TEXT    NOT NULL,
    signup_date    DATE    NOT NULL,
    signup_channel TEXT    NOT NULL,      -- paid_search | organic | referral
    plan_at_signup TEXT    NOT NULL       -- Starter | Growth | Scale
);

CREATE TABLE raw_events (
    event_id   INTEGER PRIMARY KEY,
    user_id    INTEGER NOT NULL,
    event_name TEXT    NOT NULL,          -- this dataset uses one activation event
    event_ts   DATE    NOT NULL
);

CREATE TABLE raw_marketing_spend (
    spend_month DATE    NOT NULL,         -- first of month
    channel     TEXT    NOT NULL,
    spend_usd   NUMERIC NOT NULL
);

CREATE TABLE raw_subscriptions (
    subscription_id INTEGER PRIMARY KEY,
    user_id         INTEGER NOT NULL,
    plan_name       TEXT    NOT NULL,
    status          TEXT    NOT NULL,     -- active | canceled
    amount_usd      NUMERIC NOT NULL,     -- monthly price
    created         DATE    NOT NULL,
    canceled_at     DATE                  -- NULL while active
);

CREATE TABLE raw_experiment_assignments (
    user_id         INTEGER NOT NULL,
    experiment_name TEXT    NOT NULL,
    variant         TEXT    NOT NULL,     -- control | treatment
    assigned_at     DATE    NOT NULL
);

INSERT INTO raw_users VALUES
(1,'owner1@bluecrest.io','Northwind Labs','2025-01-01','paid_search','Growth'),
(2,'owner2@ironvale.io','Bluecrest Studio','2025-01-03','paid_search','Starter'),
(3,'owner3@solarpeak.io','Ironvale Works','2025-01-05','paid_search','Starter'),
(4,'owner4@rivergate.io','Solarpeak Collective','2025-01-07','paid_search','Growth'),
(5,'owner5@amberline.io','Rivergate Group','2025-01-09','paid_search','Starter'),
(6,'owner6@cobaltworks.io','Amberline Partners','2025-01-11','organic','Starter'),
(7,'owner7@silverbrook.io','Cobaltworks Analytics','2025-01-13','organic','Growth'),
(8,'owner8@fernwood.io','Silverbrook Digital','2025-01-15','organic','Scale'),
(9,'owner9@granite.io','Fernwood Systems','2025-01-17','organic','Starter'),
(10,'owner10@harborlight.io','Granite Ventures','2025-01-19','referral','Starter'),
(11,'owner11@maple.io','Harborlight Labs','2025-01-21','referral','Starter'),
(12,'owner12@cedarline.io','Maple Studio','2025-01-23','referral','Starter'),
(13,'owner13@brightfield.io','Cedarline Works','2025-02-01','paid_search','Growth'),
(14,'owner14@stonegate.io','Brightfield Collective','2025-02-03','paid_search','Growth'),
(15,'owner15@pinehollow.io','Stonegate Group','2025-02-05','paid_search','Growth'),
(16,'owner16@westmark.io','Pinehollow Partners','2025-02-07','paid_search','Starter'),
(17,'owner17@coppertrail.io','Westmark Analytics','2025-02-09','paid_search','Growth'),
(18,'owner18@vantage.io','Coppertrail Digital','2025-02-11','organic','Growth'),
(19,'owner19@lighthouse.io','Vantage Systems','2025-02-13','organic','Growth'),
(20,'owner20@meridian.io','Lighthouse Ventures','2025-02-15','organic','Scale'),
(21,'owner21@auroraworks.io','Meridian Labs','2025-02-17','organic','Growth'),
(22,'owner22@redoak.io','Auroraworks Studio','2025-02-19','referral','Starter'),
(23,'owner23@bramblewood.io','Redoak Works','2025-02-21','referral','Scale'),
(24,'owner24@quarryrun.io','Bramblewood Collective','2025-02-23','referral','Growth'),
(25,'owner25@foxglove.io','Quarryrun Group','2025-03-01','paid_search','Scale'),
(26,'owner26@sablewood.io','Foxglove Partners','2025-03-03','paid_search','Starter'),
(27,'owner27@tidewater.io','Sablewood Analytics','2025-03-05','paid_search','Starter'),
(28,'owner28@wrenfield.io','Tidewater Digital','2025-03-07','paid_search','Growth'),
(29,'owner29@cascadia.io','Wrenfield Systems','2025-03-09','paid_search','Scale'),
(30,'owner30@highmoor.io','Cascadia Ventures','2025-03-11','organic','Growth'),
(31,'owner31@thornbury.io','Highmoor Labs','2025-03-13','organic','Growth'),
(32,'owner32@willowmere.io','Thornbury Studio','2025-03-15','organic','Starter'),
(33,'owner33@sparrowdale.io','Willowmere Works','2025-03-17','organic','Growth'),
(34,'owner34@ashford.io','Sparrowdale Collective','2025-03-19','referral','Scale'),
(35,'owner35@windmere.io','Ashford Group','2025-03-21','referral','Growth'),
(36,'owner36@clearview.io','Windmere Partners','2025-03-23','referral','Growth'),
(37,'owner37@stormcrest.io','Clearview Analytics','2025-04-01','paid_search','Starter'),
(38,'owner38@emberfall.io','Stormcrest Digital','2025-04-03','paid_search','Starter'),
(39,'owner39@larkspur.io','Emberfall Systems','2025-04-05','paid_search','Starter'),
(40,'owner40@driftwood.io','Larkspur Ventures','2025-04-07','paid_search','Starter'),
(41,'owner41@hollowreach.io','Driftwood Labs','2025-04-09','paid_search','Growth'),
(42,'owner42@nightingale.io','Hollowreach Studio','2025-04-11','organic','Scale'),
(43,'owner43@palisade.io','Nightingale Works','2025-04-13','organic','Growth'),
(44,'owner44@ridgeline.io','Palisade Collective','2025-04-15','organic','Starter'),
(45,'owner45@saltmarsh.io','Ridgeline Group','2025-04-17','organic','Starter'),
(46,'owner46@timberline.io','Saltmarsh Partners','2025-04-19','referral','Growth'),
(47,'owner47@vesper.io','Timberline Analytics','2025-04-21','referral','Growth'),
(48,'owner48@wildrose.io','Vesper Digital','2025-04-23','referral','Scale'),
(49,'owner49@yellowpine.io','Wildrose Systems','2025-05-01','paid_search','Starter'),
(50,'owner50@zephyrmark.io','Yellowpine Ventures','2025-05-03','paid_search','Starter'),
(51,'owner51@basalt.io','Zephyrmark Labs','2025-05-05','paid_search','Growth'),
(52,'owner52@cinderpoint.io','Basalt Studio','2025-05-07','paid_search','Starter'),
(53,'owner53@duskwood.io','Cinderpoint Works','2025-05-09','paid_search','Growth'),
(54,'owner54@everglade.io','Duskwood Collective','2025-05-11','organic','Starter'),
(55,'owner55@frostgate.io','Everglade Group','2025-05-13','organic','Starter'),
(56,'owner56@glimmer.io','Frostgate Partners','2025-05-15','organic','Starter'),
(57,'owner57@heronbay.io','Glimmer Analytics','2025-05-17','organic','Starter'),
(58,'owner58@ivorytrail.io','Heronbay Digital','2025-05-19','referral','Scale'),
(59,'owner59@juniper.io','Ivorytrail Systems','2025-05-21','referral','Growth'),
(60,'owner60@kestrelmoor.io','Juniper Ventures','2025-05-23','referral','Growth'),
(61,'owner61@lanternhill.io','Kestrelmoor Labs','2025-06-01','paid_search','Starter'),
(62,'owner62@moorland.io','Lanternhill Studio','2025-06-03','paid_search','Growth'),
(63,'owner63@novapeak.io','Moorland Works','2025-06-05','paid_search','Growth'),
(64,'owner64@oakleaf.io','Novapeak Collective','2025-06-07','paid_search','Growth'),
(65,'owner65@prairiewind.io','Oakleaf Group','2025-06-09','paid_search','Starter'),
(66,'owner66@quietstone.io','Prairiewind Partners','2025-06-11','organic','Growth'),
(67,'owner67@ravenscroft.io','Quietstone Analytics','2025-06-13','organic','Starter'),
(68,'owner68@summitgate.io','Ravenscroft Digital','2025-06-15','organic','Growth'),
(69,'owner69@truelight.io','Summitgate Systems','2025-06-17','organic','Growth'),
(70,'owner70@underbrook.io','Truelight Ventures','2025-06-19','referral','Scale'),
(71,'owner71@valewood.io','Underbrook Labs','2025-06-21','referral','Starter'),
(72,'owner72@northwind.io','Valewood Studio','2025-06-23','referral','Scale');

INSERT INTO raw_events VALUES
(1,1,'created_first_project','2025-01-06'),
(2,2,'created_first_project','2025-01-07'),
(3,4,'created_first_project','2025-01-14'),
(4,5,'created_first_project','2025-01-11'),
(5,6,'created_first_project','2025-01-25'),
(6,9,'created_first_project','2025-01-31'),
(7,10,'created_first_project','2025-01-28'),
(8,11,'created_first_project','2025-02-03'),
(9,12,'created_first_project','2025-01-29'),
(10,13,'created_first_project','2025-02-12'),
(11,14,'created_first_project','2025-02-13'),
(12,15,'created_first_project','2025-02-06'),
(13,16,'created_first_project','2025-02-18'),
(14,18,'created_first_project','2025-02-20'),
(15,19,'created_first_project','2025-02-18'),
(16,20,'created_first_project','2025-02-24'),
(17,21,'created_first_project','2025-02-28'),
(18,24,'created_first_project','2025-02-25'),
(19,26,'created_first_project','2025-03-13'),
(20,27,'created_first_project','2025-03-07'),
(21,28,'created_first_project','2025-03-18'),
(22,29,'created_first_project','2025-03-13'),
(23,30,'created_first_project','2025-03-23'),
(24,31,'created_first_project','2025-03-17'),
(25,32,'created_first_project','2025-03-19'),
(26,33,'created_first_project','2025-03-20'),
(27,34,'created_first_project','2025-03-23'),
(28,35,'created_first_project','2025-03-23'),
(29,37,'created_first_project','2025-04-07'),
(30,38,'created_first_project','2025-04-12'),
(31,41,'created_first_project','2025-04-18'),
(32,43,'created_first_project','2025-04-21'),
(33,44,'created_first_project','2025-04-28'),
(34,45,'created_first_project','2025-04-29'),
(35,46,'created_first_project','2025-04-22'),
(36,47,'created_first_project','2025-04-22'),
(37,48,'created_first_project','2025-05-02'),
(38,52,'created_first_project','2025-05-18'),
(39,54,'created_first_project','2025-05-15'),
(40,55,'created_first_project','2025-05-26'),
(41,56,'created_first_project','2025-05-25'),
(42,57,'created_first_project','2025-05-22'),
(43,58,'created_first_project','2025-05-22'),
(44,59,'created_first_project','2025-06-03'),
(45,60,'created_first_project','2025-06-03'),
(46,62,'created_first_project','2025-06-08'),
(47,63,'created_first_project','2025-06-16'),
(48,64,'created_first_project','2025-06-21'),
(49,65,'created_first_project','2025-06-10'),
(50,66,'created_first_project','2025-06-23'),
(51,67,'created_first_project','2025-06-15'),
(52,68,'created_first_project','2025-06-16'),
(53,69,'created_first_project','2025-06-20'),
(54,70,'created_first_project','2025-07-03'),
(55,71,'created_first_project','2025-06-23'),
(56,72,'created_first_project','2025-07-03');

INSERT INTO raw_marketing_spend VALUES
('2025-01-01','organic',2400),
('2025-01-01','paid_search',3600),
('2025-01-01','referral',450),
('2025-02-01','organic',2400),
('2025-02-01','paid_search',3600),
('2025-02-01','referral',450),
('2025-03-01','organic',2400),
('2025-03-01','paid_search',3600),
('2025-03-01','referral',450),
('2025-04-01','organic',2400),
('2025-04-01','paid_search',3600),
('2025-04-01','referral',450),
('2025-05-01','organic',2400),
('2025-05-01','paid_search',3600),
('2025-05-01','referral',450),
('2025-06-01','organic',2400),
('2025-06-01','paid_search',3600),
('2025-06-01','referral',450);

INSERT INTO raw_subscriptions VALUES
(1,1,'Growth','canceled',99,'2025-01-11','2025-05-14'),
(2,2,'Starter','active',29,'2025-01-10',NULL),
(3,4,'Growth','canceled',99,'2025-01-19','2025-02-25'),
(4,5,'Starter','active',29,'2025-01-19',NULL),
(5,6,'Starter','canceled',29,'2025-02-02','2025-05-08'),
(6,9,'Starter','active',29,'2025-02-08',NULL),
(7,10,'Starter','active',29,'2025-02-02',NULL),
(8,12,'Starter','active',29,'2025-02-02',NULL),
(9,13,'Growth','canceled',99,'2025-02-19','2025-03-24'),
(10,14,'Growth','canceled',99,'2025-02-25','2025-04-05'),
(11,16,'Starter','active',29,'2025-02-22',NULL),
(12,19,'Growth','canceled',99,'2025-02-28','2025-03-24'),
(13,21,'Growth','active',99,'2025-03-13',NULL),
(14,28,'Growth','canceled',99,'2025-03-29','2025-04-17'),
(15,30,'Growth','active',99,'2025-04-05',NULL),
(16,31,'Growth','active',99,'2025-03-20',NULL),
(17,32,'Starter','active',29,'2025-03-22',NULL),
(18,35,'Growth','active',99,'2025-03-31',NULL),
(19,38,'Starter','canceled',29,'2025-04-21','2025-05-15'),
(20,43,'Growth','active',99,'2025-04-30',NULL),
(21,46,'Growth','active',99,'2025-04-28',NULL),
(22,47,'Growth','active',99,'2025-05-02',NULL),
(23,48,'Scale','active',299,'2025-05-07',NULL),
(24,54,'Starter','active',29,'2025-05-20',NULL),
(25,57,'Starter','active',29,'2025-05-30',NULL),
(26,58,'Scale','active',299,'2025-06-02',NULL),
(27,59,'Growth','active',99,'2025-06-06',NULL),
(28,60,'Growth','active',99,'2025-06-08',NULL),
(29,64,'Growth','active',99,'2025-07-04',NULL),
(30,65,'Starter','active',29,'2025-06-15',NULL),
(31,66,'Growth','active',99,'2025-07-07',NULL),
(32,69,'Growth','active',99,'2025-06-28',NULL),
(33,70,'Scale','active',299,'2025-07-11',NULL),
(34,71,'Starter','active',29,'2025-07-04',NULL),
(35,72,'Scale','active',299,'2025-07-09',NULL);

INSERT INTO raw_experiment_assignments VALUES
(49,'onboarding_checklist_v2','control','2025-05-01'),
(50,'onboarding_checklist_v2','treatment','2025-05-03'),
(51,'onboarding_checklist_v2','control','2025-05-05'),
(52,'onboarding_checklist_v2','treatment','2025-05-07'),
(53,'onboarding_checklist_v2','control','2025-05-09'),
(54,'onboarding_checklist_v2','treatment','2025-05-11'),
(55,'onboarding_checklist_v2','control','2025-05-13'),
(56,'onboarding_checklist_v2','treatment','2025-05-15'),
(57,'onboarding_checklist_v2','control','2025-05-17'),
(58,'onboarding_checklist_v2','treatment','2025-05-19'),
(59,'onboarding_checklist_v2','control','2025-05-21'),
(60,'onboarding_checklist_v2','treatment','2025-05-23'),
(61,'onboarding_checklist_v2','control','2025-06-01'),
(62,'onboarding_checklist_v2','treatment','2025-06-03'),
(63,'onboarding_checklist_v2','control','2025-06-05'),
(64,'onboarding_checklist_v2','treatment','2025-06-07'),
(65,'onboarding_checklist_v2','control','2025-06-09'),
(66,'onboarding_checklist_v2','treatment','2025-06-11'),
(67,'onboarding_checklist_v2','control','2025-06-13'),
(68,'onboarding_checklist_v2','treatment','2025-06-15'),
(69,'onboarding_checklist_v2','control','2025-06-17'),
(70,'onboarding_checklist_v2','treatment','2025-06-19'),
(71,'onboarding_checklist_v2','control','2025-06-21'),
(72,'onboarding_checklist_v2','treatment','2025-06-23');
```

Sanity checks — these should print exactly these numbers:

```sql
SELECT COUNT(*) FROM raw_users;                        -- 72
SELECT COUNT(*) FROM raw_events;                        -- 56
SELECT COUNT(*) FROM raw_marketing_spend;                -- 18
SELECT COUNT(*) FROM raw_subscriptions;                  -- 35
SELECT COUNT(*) FROM raw_experiment_assignments;          -- 24
SELECT COUNT(*) FROM raw_subscriptions WHERE status = 'active';    -- 27
SELECT COUNT(*) FROM raw_subscriptions WHERE status = 'canceled';  -- 8
```

Notice `raw_subscriptions` has **35** rows but only **30** of them were `created` on or before the analysis cutoff of `2025-06-30` — five June signups (users 64, 66, 70, 71, 72) convert to paying customers in early July. That's deliberate right-censoring, straight out of Week 5: the raw layer holds everything, but a mart built "as of 2025-06-30" must explicitly filter to it, or it will silently claim knowledge it doesn't have yet.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Scope the capstone; stand up the warehouse | 2h | 1.5h | 0h | 0.5h | 0.5h | 0h | 4.5h |
| Tuesday | Marts: acquisition, activation, retention | 1h | 1.5h | 0h | 0.5h | 1h | 0h | 4h |
| Wednesday | Unit economics + the capstone experiment | 1h | 1.5h | 0h | 0.5h | 1h | 0h | 4h |
| Thursday | Assemble the whole system (Challenge 1) | 0h | 0h | 2h | 0.5h | 1h | 0h | 3.5h |
| Friday | Forecast + present the plan (Lecture 3, Challenge 2) | 2h | 0h | 1.5h | 0.5h | 1h | 0h | 5h |
| Saturday | Mini-project: ship the full growth system | 0h | 0h | 0h | 0h | 0h | 5.5h | 5.5h |
| Sunday | Quiz + final review | 0h | 0h | 0h | 1.5h | 0h | 0h | 1.5h |
| **Total** | | **6h** | **4.5h** | **3.5h** | **4h** | **4.5h** | **5.5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-capstone-scope-and-warehouse.md](./lecture-notes/01-capstone-scope-and-warehouse.md) | Scope the capstone, stand up raw → staging → marts on Crunch Boards | 2h |
| 2 | [lecture-notes/02-assembling-the-growth-system.md](./lecture-notes/02-assembling-the-growth-system.md) | Wire funnel, retention, unit economics, the experiment into one system | 2h |
| 3 | [lecture-notes/03-presenting-growth-decisions.md](./lecture-notes/03-presenting-growth-decisions.md) | Forecast, reconcile methods, turn the system into a decision narrative | 2h |
| 4 | [exercises/exercise-01-stand-up-the-warehouse.md](./exercises/exercise-01-stand-up-the-warehouse.md) | Build the staging layer + `dim_user`, `dim_date` | 1.5h |
| 5 | [exercises/exercise-02-build-the-metric-suite.md](./exercises/exercise-02-build-the-metric-suite.md) | Build `fct_acquisition`, `fct_mrr_monthly`, `fct_unit_economics` | 1.5h |
| 6 | [exercises/exercise-03-run-the-capstone-experiment.md](./exercises/exercise-03-run-the-capstone-experiment.md) | Analyze `onboarding_checklist_v2` and decide ship/hold/extend | 1.5h |
| 7 | [challenges/challenge-01-end-to-end-growth-system.md](./challenges/challenge-01-end-to-end-growth-system.md) | Wire every mart into one build script + a retention-cohort view | 2h |
| 8 | [challenges/challenge-02-defend-the-growth-plan.md](./challenges/challenge-02-defend-the-growth-plan.md) | Forecast with 2+ methods, reconcile, defend a channel budget call | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Ship the full growth system as a defended decision deck | 5.5h |
| 10 | [homework.md](./homework.md) | Extra reps: new channel, new experiment, sensitivity checks | 4.5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Stand up a warehouse from raw events to business-ready marts, alone, for a growth question you scoped yourself.
- Read one system and answer "how are we acquiring, activating, keeping, and monetizing customers" without switching tools.
- Analyze an experiment and say plainly whether the evidence is strong enough to act on — not just whether the lift looks good.
- Model unit economics per channel and turn LTV:CAC into an actual budget recommendation.
- Forecast with more than one method, show where they disagree, and defend the one you'd bet the quarter on.
- Present a growth system as a decision, with every number a query away from being re-checked live.

## Course complete

This is the last week of **C38 · Crunch Scale**. When you ship the mini-project and pass the quiz, you've run — start to finish, in SQL and pandas, never a spreadsheet — the same system a RevOps or growth-analytics lead runs in production: instrument, warehouse, measure, experiment, price, and forecast. That's the job.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
