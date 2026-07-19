# Week 6 — RevOps & the Customer Data Stack

> **Goal:** by Sunday you can take three disagreeing sources of truth — a product database, a billing system, and a raw event stream — and turn them into one warehouse that everyone in the company trusts, with a documented, tested, idempotent SQL pipeline from **raw → staging → marts**. No spreadsheet touches a single number.

Every company eventually asks: "what's our MRR, actually?" And every company that hasn't built a warehouse gets three different answers — one from Stripe, one from the product database, one from whatever pivot table Finance keeps updating by hand. This week you fix that permanently. You'll model a small SaaS company (**Crunch Flow**, a project-management product) whose billing system and internal app database have quietly drifted apart, and you'll build the layered SQL pipeline — raw events, staging, marts — that reconciles them into a single number, with tests that catch the next drift before a human notices it in a board deck.

This is the RevOps week: not a growth-tactics lecture, but the plumbing that makes every other growth number in this course (funnels, retention, LTV, experiments, pricing) rest on data you can actually defend under questioning.

## Learning objectives

By the end of this week, you will be able to:

- **Explain** the modern customer data stack — raw events, staging models, analytics marts — and where ELT and a semantic layer fit, using plain SQL (views and tables), no proprietary tooling required.
- **Design** conformed dimension and fact tables for users, subscriptions, and revenue, so every downstream query (this course and beyond) joins to the same `dim_user` and reads the same `fct_mrr_monthly`.
- **Reconcile** two disagreeing source systems (a billing system and a product database) into one trusted MRR number, and document *why* one number won.
- **Write idempotent loads** — `TRUNCATE`+reload, `INSERT ... ON CONFLICT DO UPDATE` (Postgres) / `INSERT OR REPLACE` (SQLite) — so re-running a pipeline never doubles a number.
- **Write data tests** as plain SQL assertions (not-null, uniqueness, accepted-values, referential integrity, freshness) that fail loudly when the data is wrong.
- **Argue, with a straight face, in an interview or to a stakeholder,** why a warehouse beats a spreadsheet for auditability, reproducibility, and single-source-of-truth revenue reporting.

## Prerequisites

- **C33 Crunch SQL** (or equivalent): comfortable with `SELECT`, joins, `GROUP BY`/aggregates, window functions, and `CREATE TABLE`.
- Weeks 1–5 of this course (growth foundations, funnels, activation, retention, LTV/CAC) — this week assumes you know *why* MRR, activation, and churn matter; it teaches you where the numbers actually come from.
- PostgreSQL 16+ (primary) or SQLite 3.35+ (fallback). No spreadsheet software is used anywhere this week — **that is a hard rule of this course.** Spreadsheets are taught only in C41 Crunch Excel, and never as a data store.

## Set up the seed database (do this first)

This week's fictional company is **Crunch Flow**, a small B2B project-management SaaS. Its data lives in three disagreeing raw sources — exactly like a real startup:

1. **`raw_users`** — the product's own signup record (one source of truth for *who* the customer is).
2. **`raw_events`** — the product's usage event stream (signup, activation, feature use, plan changes, cancellations).
3. **`raw_stripe_subscriptions`** — an export from the billing system (Stripe-shaped). This is money's source of truth, keyed by **email** — Stripe has never heard of your internal `user_id`.
4. **`raw_app_subscriptions`** — the internal product database's *own* subscription record, keyed by `user_id`, using the product's **legacy plan names** (`Basic`/`Pro`/`Enterprise`) instead of the current public names (`Starter`/`Growth`/`Scale`) — and, as you'll discover, occasionally out of sync with what Stripe actually billed.

**PostgreSQL:**

```bash
createdb crunch_flow
psql crunch_flow
```

**SQLite:**

```bash
sqlite3 crunch_flow.db
```

Paste this into the shell (unchanged on both engines):

```sql
-- ============================================================
-- RAW LAYER — landed as-is from source systems, never modified
-- ============================================================

CREATE TABLE raw_users (
    user_id      INTEGER PRIMARY KEY,
    email        TEXT NOT NULL,
    full_name    TEXT NOT NULL,
    company_name TEXT NOT NULL,
    country      TEXT NOT NULL,
    signup_date  DATE NOT NULL,
    plan_at_signup TEXT NOT NULL
);

INSERT INTO raw_users VALUES
(1,'ada@northwind.io','Ada Byrne','Northwind Traders','USA','2025-01-08','Starter'),
(2,'ben@lumen.co','Ben Ortiz','Lumen Studio','USA','2025-01-15','Growth'),
(3,'cara@fjordtech.no','Cara Voss','Fjord Tech','Norway','2025-01-22','Starter'),
(4,'drew@brightpath.com','Drew Kim','BrightPath Consulting','USA','2025-02-03','Growth'),
(5,'elin@stavro.se','Elin Berg','Stavro AB','Sweden','2025-02-10','Starter'),
(6,'felix@marlowe.io','Felix Marlowe','Marlowe & Co','UK','2025-02-19','Scale'),
(7,'gina@rioverde.mx','Gina Ruiz','Rio Verde Software','Mexico','2025-03-01','Growth'),
(8,'hugo@baumgarten.de','Hugo Baumgarten','Baumgarten GmbH','Germany','2025-03-08','Starter'),
(9,'ines@delmar.es','Ines Del Mar','Del Mar Analytics','Spain','2025-03-14','Growth'),
(10,'jae@hanuri.kr','Jae Hanuri','Hanuri Labs','South Korea','2025-03-22','Starter'),
(11,'kira@novasys.ca','Kira Novak','NovaSys','Canada','2025-04-02','Scale'),
(12,'liam@westfold.ie','Liam Walsh','Westfold Digital','Ireland','2025-04-11','Growth'),
(13,'mira@suncrest.au','Mira Patel','Suncrest Farms','Australia','2025-04-19','Starter'),
(14,'noel@tidewater.us','Noel Ford','Tidewater Logistics','USA','2025-05-02','Growth'),
(15,'opal@brightloop.io','Opal Reyes','BrightLoop','USA','2025-05-10','Scale');

CREATE TABLE raw_events (
    event_id   INTEGER PRIMARY KEY,
    user_id    INTEGER NOT NULL,
    event_name TEXT NOT NULL,
    event_ts   TIMESTAMP NOT NULL,
    source     TEXT NOT NULL
);

INSERT INTO raw_events VALUES
(1,1,'signup','2025-01-08 09:00:00','web_app'),
(2,1,'activated','2025-01-09 14:22:00','web_app'),
(3,1,'feature_used','2025-01-12 10:00:00','web_app'),
(4,1,'feature_used','2025-02-02 11:00:00','web_app'),
(5,1,'invite_sent','2025-02-15 09:30:00','web_app'),
(6,2,'signup','2025-01-15 10:00:00','web_app'),
(7,2,'activated','2025-01-16 09:00:00','web_app'),
(8,2,'feature_used','2025-01-20 13:00:00','web_app'),
(9,2,'invite_sent','2025-02-01 10:00:00','web_app'),
(10,2,'upgraded','2025-03-10 14:05:00','billing_webhook'),
(11,2,'feature_used','2025-03-15 09:00:00','web_app'),
(12,3,'signup','2025-01-22 08:00:00','web_app'),
(13,3,'activated','2025-01-24 16:00:00','web_app'),
(14,3,'feature_used','2025-02-10 10:00:00','web_app'),
(15,4,'signup','2025-02-03 09:00:00','web_app'),
(16,4,'activated','2025-02-04 10:30:00','web_app'),
(17,4,'feature_used','2025-02-20 11:00:00','web_app'),
(18,4,'upgraded','2025-03-28 16:00:00','billing_webhook'),
(19,4,'feature_used','2025-04-02 09:00:00','web_app'),
(20,5,'signup','2025-02-10 09:00:00','web_app'),
(21,5,'payment_failed','2025-02-24 00:00:00','billing_webhook'),
(22,5,'canceled','2025-02-25 11:00:00','billing_webhook'),
(23,6,'signup','2025-02-19 10:00:00','web_app'),
(24,6,'activated','2025-02-19 15:00:00','web_app'),
(25,6,'feature_used','2025-02-25 09:00:00','web_app'),
(26,6,'invite_sent','2025-03-01 09:00:00','web_app'),
(27,6,'feature_used','2025-03-20 09:00:00','web_app'),
(28,7,'signup','2025-03-01 09:00:00','web_app'),
(29,7,'activated','2025-03-02 10:00:00','web_app'),
(30,7,'payment_failed','2025-04-01 00:00:00','billing_webhook'),
(31,7,'payment_succeeded','2025-04-03 08:00:00','billing_webhook'),
(32,7,'feature_used','2025-04-10 09:00:00','web_app'),
(33,8,'signup','2025-03-08 09:00:00','web_app'),
(34,8,'activated','2025-03-09 11:00:00','web_app'),
(35,8,'feature_used','2025-03-15 09:00:00','web_app'),
(36,9,'signup','2025-03-14 09:00:00','web_app'),
(37,9,'activated','2025-03-15 10:00:00','web_app'),
(38,9,'downgraded','2025-04-05 09:00:00','billing_webhook'),
(39,9,'feature_used','2025-04-15 09:00:00','web_app'),
(40,10,'signup','2025-03-22 09:00:00','web_app'),
(41,10,'payment_failed','2025-04-09 00:00:00','billing_webhook'),
(42,10,'canceled','2025-04-10 09:00:00','billing_webhook'),
(43,11,'signup','2025-04-02 09:00:00','web_app'),
(44,11,'activated','2025-04-03 10:00:00','web_app'),
(45,11,'feature_used','2025-04-10 09:00:00','web_app'),
(46,11,'invite_sent','2025-04-20 09:00:00','web_app'),
(47,12,'signup','2025-04-11 09:00:00','web_app'),
(48,12,'activated','2025-04-12 09:00:00','web_app'),
(49,12,'feature_used','2025-04-25 09:00:00','web_app'),
(50,13,'signup','2025-04-19 09:00:00','web_app'),
(51,13,'activated','2025-04-20 10:00:00','web_app'),
(52,13,'payment_failed','2025-05-18 00:00:00','billing_webhook'),
(53,13,'canceled','2025-05-19 11:00:00','billing_webhook'),
(54,14,'signup','2025-05-02 09:00:00','web_app'),
(55,14,'activated','2025-05-03 10:00:00','web_app'),
(56,15,'signup','2025-05-10 09:00:00','web_app'),
(57,15,'activated','2025-05-11 10:00:00','web_app'),
(58,15,'feature_used','2025-05-20 09:00:00','web_app');

-- Billing system export (Stripe-shaped). Money's source of truth.
-- Keyed by EMAIL — Stripe has no idea what your internal user_id is.
CREATE TABLE raw_stripe_subscriptions (
    subscription_id     TEXT PRIMARY KEY,
    customer_email       TEXT NOT NULL,
    plan_name            TEXT NOT NULL,   -- Starter / Growth / Scale
    status                TEXT NOT NULL,   -- active / canceled
    amount_cents          INTEGER NOT NULL,
    billing_interval      TEXT NOT NULL,   -- month / year
    created               DATE NOT NULL,
    current_period_end    DATE NOT NULL,
    canceled_at           DATE
);

INSERT INTO raw_stripe_subscriptions VALUES
('sub_1001','ada@northwind.io','Starter','active',2900,'month','2025-01-08','2025-06-08',NULL),
('sub_1002','ben@lumen.co','Growth','canceled',9900,'month','2025-01-15','2025-03-15','2025-03-10'),
('sub_1003','ben@lumen.co','Scale','active',29900,'month','2025-03-10','2025-06-10',NULL),
('sub_1004','cara@fjordtech.no','Starter','active',2900,'month','2025-01-22','2025-06-22',NULL),
('sub_1005','drew@brightpath.com','Growth','canceled',9900,'month','2025-02-03','2025-04-03','2025-03-28'),
('sub_1006','drew@brightpath.com','Scale','active',29900,'month','2025-03-28','2025-06-28',NULL),
('sub_1007','elin@stavro.se','Starter','canceled',2900,'month','2025-02-10','2025-03-10','2025-02-25'),
('sub_1008','felix@marlowe.io','Scale','active',29900,'month','2025-02-19','2025-06-19',NULL),
('sub_1009','gina@rioverde.mx','Growth','active',9900,'month','2025-03-01','2025-06-01',NULL),
('sub_1010','hugo@baumgarten.de','Starter','active',2900,'month','2025-03-08','2025-06-08',NULL),
('sub_1011','ines@delmar.es','Growth','canceled',9900,'month','2025-03-14','2025-04-05','2025-04-05'),
('sub_1012','ines@delmar.es','Starter','active',2900,'month','2025-04-05','2025-06-05',NULL),
('sub_1013','jae@hanuri.kr','Starter','canceled',2900,'month','2025-03-22','2025-04-22','2025-04-10'),
('sub_1014','kira@novasys.ca','Scale','active',29900,'month','2025-04-02','2025-06-02',NULL),
('sub_1015','liam@westfold.ie','Growth','active',9900,'month','2025-04-11','2025-06-11',NULL),
('sub_1016','mira@suncrest.au','Starter','canceled',2900,'month','2025-04-19','2025-05-19','2025-05-19'),
('sub_1017','noel@tidewater.us','Growth','active',9900,'month','2025-05-02','2025-06-02',NULL),
('sub_1018','opal@brightloop.io','Scale','active',299000,'year','2025-05-10','2026-05-10',NULL);

-- Internal product database's OWN subscription record — one row per user
-- (current-state table, not history). Uses LEGACY plan names, and has
-- drifted out of sync with Stripe in three places. Finding those three
-- rows is the point of Exercise 3.
CREATE TABLE raw_app_subscriptions (
    id             INTEGER PRIMARY KEY,
    user_id        INTEGER NOT NULL,
    plan           TEXT NOT NULL,     -- Basic / Pro / Enterprise (legacy names)
    price_usd      NUMERIC NOT NULL,
    billing_cycle  TEXT NOT NULL,     -- monthly / annual
    status         TEXT NOT NULL,     -- active / canceled
    start_date     DATE NOT NULL,
    end_date       DATE,
    updated_at     TIMESTAMP NOT NULL
);

INSERT INTO raw_app_subscriptions VALUES
(1,1,'Basic',29.00,'monthly','active','2025-01-08',NULL,'2025-01-08 09:02:00'),
(2,2,'Enterprise',299.00,'monthly','active','2025-03-10',NULL,'2025-03-10 14:11:00'),
(3,3,'Basic',29.00,'monthly','active','2025-01-22',NULL,'2025-01-22 08:40:00'),
(4,4,'Enterprise',299.00,'monthly','active','2025-03-28',NULL,'2025-03-28 16:05:00'),
(5,5,'Basic',29.00,'monthly','canceled','2025-02-10','2025-02-25','2025-02-25 11:00:00'),
(6,6,'Enterprise',299.00,'monthly','active','2025-02-19',NULL,'2025-02-19 10:15:00'),
(7,7,'Pro',99.00,'monthly','active','2025-03-01',NULL,'2025-03-01 09:30:00'),
(8,8,'Basic',29.00,'monthly','active','2025-03-08',NULL,'2025-03-08 12:00:00'),
(9,9,'Pro',99.00,'monthly','active','2025-03-14',NULL,'2025-03-14 13:20:00'),
(10,10,'Basic',29.00,'monthly','canceled','2025-03-22','2025-04-10','2025-04-10 09:00:00'),
(11,11,'Enterprise',299.00,'annual','active','2025-04-02',NULL,'2025-04-02 15:45:00'),
(12,12,'Pro',99.00,'monthly','active','2025-04-11',NULL,'2025-04-11 09:55:00'),
(13,13,'Basic',29.00,'monthly','active','2025-04-19',NULL,'2025-04-19 08:10:00'),
(14,14,'Pro',99.00,'monthly','active','2025-05-02',NULL,'2025-05-02 10:40:00'),
(15,15,'Enterprise',2990.00,'annual','active','2025-05-10',NULL,'2025-05-10 11:25:00');
```

Sanity checks — these should print `15`, `58`, `18`, `15`:

```sql
SELECT COUNT(*) FROM raw_users;
SELECT COUNT(*) FROM raw_events;
SELECT COUNT(*) FROM raw_stripe_subscriptions;
SELECT COUNT(*) FROM raw_app_subscriptions;
```

Two things are baked into this data **on purpose**, and you'll need both all week:

1. **Ben and Drew each have two Stripe subscription rows** — an old canceled one and a new active one, because upgrading a Stripe subscription in real life often creates a new subscription object rather than mutating the old one in place. Your staging layer has to pick the *current* row, not just any row.
2. **The internal app database and Stripe disagree on three customers** (Ines, Kira, Mira) — plan/price, billing cycle, and cancellation status respectively. That's not a typo; that's what real RevOps data looks like, and Exercise 3 makes you find and resolve all three.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed data; the modern customer data stack | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Modeling users & revenue; dims and facts | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Build the staging layer | 0h | 2h | 1h | 0.5h | 1h | 0h | 4.5h |
| Thursday | Model the MRR fact; reconcile two sources | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | RevOps hygiene: idempotent loads, data tests | 2h | 0h | 1h | 0.5h | 1h | 1.5h | 6h |
| Saturday | Mini-project (full raw → staging → marts) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **28.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it, and every SQL example targets the seed database you just created — the same one for the whole week.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-the-modern-data-stack.md](./lecture-notes/01-the-modern-data-stack.md) | Events → staging → marts, ELT vs. ETL, the semantic layer, where RevOps owns definitions | 2h |
| 2 | [lecture-notes/02-modeling-users-and-revenue.md](./lecture-notes/02-modeling-users-and-revenue.md) | Conformed `dim_user`/`dim_date`, `fct_mrr_monthly`, normalizing annual to monthly, one trusted MRR number | 2h |
| 3 | [lecture-notes/03-revops-hygiene.md](./lecture-notes/03-revops-hygiene.md) | Idempotent loads, data tests as SQL assertions, documented metric definitions, warehouse vs. spreadsheet | 2h |
| 4 | [exercises/exercise-01-build-a-staging-layer.md](./exercises/exercise-01-build-a-staging-layer.md) | Clean, type, dedupe, and conform all four raw tables into `stg_*` | 2h |
| 5 | [exercises/exercise-02-model-an-mrr-fact.md](./exercises/exercise-02-model-an-mrr-fact.md) | Build `fct_mrr_monthly` off a date spine and subscription ranges | 1.5h |
| 6 | [exercises/exercise-03-reconcile-two-sources.md](./exercises/exercise-03-reconcile-two-sources.md) | Find and resolve the three Stripe vs. app-DB discrepancies | 1.5h |
| 7 | [challenges/challenge-01-design-a-mart-schema.md](./challenges/challenge-01-design-a-mart-schema.md) | Design (in DDL) the full mart layer this course's later weeks will build on | 1.5h |
| 8 | [challenges/challenge-02-add-data-tests.md](./challenges/challenge-02-add-data-tests.md) | Write a SQL test suite that fails loudly on bad data | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Ship the full raw → staging → marts pipeline as one auditable source of truth | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice: ARR, logo churn, and a second reconciliation drill | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Explain, to a skeptical stakeholder, why the warehouse's MRR number is the right one — and defend it with a paper trail back to raw data.
- Build a staging layer that cleans, types, dedupes, and conforms disagreeing source systems into one naming standard.
- Model conformed `dim_user`/`dim_date` dimensions and an `fct_mrr_monthly` fact that every later week of this course (segmentation, experiments, pricing, forecasting) can join to without re-deriving revenue from scratch.
- Write idempotent SQL loads and data tests that catch drift before it reaches a dashboard.
- Say, with evidence, exactly why none of this belongs in a spreadsheet.

## Up next

[Week 7 — Segmentation & customer intelligence](../week-07-segmentation-and-customer-intelligence/) — now that revenue and users live in trustworthy marts, you'll segment them into RFM, behavioral, and clustering-based cohorts.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
