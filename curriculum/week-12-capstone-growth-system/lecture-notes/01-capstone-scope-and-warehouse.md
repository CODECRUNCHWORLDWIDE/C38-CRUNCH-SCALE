# Lecture 1 — Capstone Scope and Warehouse

> **Duration:** ~2 hours. **Outcome:** You can scope a capstone growth question in one sentence, explain why that sentence controls every table you build afterward, and stand up the staging layer and two core dimensions (`dim_user`, `dim_date`) on top of the `crunch_boards` raw tables from the [week README](../README.md).

Run every query in this lecture against `crunch_boards`. If you haven't seeded it yet, stop and do that now — nothing here works without it.

## 1. Scope before you build anything

Every capstone that goes sideways goes sideways the same way: someone opens a SQL editor before they've written down the one sentence that says what the whole project is *for*. Twelve weeks of this course gave you enough tools to answer a hundred different questions about Crunch Boards. You don't have time to answer a hundred questions this week, and neither does a real stakeholder. You have time to answer **one**, defended completely, end to end.

That one sentence is your **scope statement**. For this capstone, it's fixed for you so everyone builds toward the same target:

> **"Which acquisition channel should Crunch Boards put next quarter's growth budget behind, and what does the onboarding-checklist experiment tell us about whether we can improve conversion on the channels we keep?"**

Notice what that sentence does and doesn't ask for. It asks for a **channel recommendation** backed by unit economics, an **experiment read**, and enough of a **revenue forecast** to know what's at stake. It does *not* ask you to build a full self-serve BI tool, model every possible metric Crunch Boards could ever want, or predict churn for every individual customer. If you find yourself building a mart that doesn't serve one of the three questions in that sentence, you've scope-crept — stop, and ask whether it belongs in `homework.md` instead of the mini-project.

**Write your own scope statement before you touch SQL**, even though this week's is provided. Practicing the one-sentence discipline here is what makes it a reflex for the next growth question you're handed at a real job, where nobody hands you the sentence — you write it, or you drown in requests.

## 2. The warehouse shape, once more, on your own data

Week 6 taught the three-layer shape — **raw → staging → marts** — on `crunch_flow`. The shape doesn't change because the company changed. `crunch_boards` gets exactly the same treatment:

```
   RAW (given)              STAGING (this lecture)         MARTS (Lecture 2)
┌───────────────────┐    ┌────────────────────────┐    ┌──────────────────────┐
│ raw_users           │    │ stg_users                │    │ dim_user                │
│ raw_events           │ →  │ stg_events                │ →  │ dim_date                │
│ raw_marketing_spend  │    │ stg_subscriptions         │    │ fct_acquisition          │
│ raw_subscriptions    │    │ stg_marketing_spend       │    │ fct_mrr_monthly           │
│ raw_experiment_...   │    │ stg_experiment_assignments│    │ fct_unit_economics        │
└───────────────────┘    └────────────────────────┘    │ fct_experiment_results    │
                                                          └──────────────────────┘
```

Raw stays untouched — you already loaded it in the README, exactly as given, and nothing in this week ever runs `UPDATE`/`DELETE` against a `raw_*` table. Staging cleans and types each source, one at a time, no cross-source joins yet. Marts assemble the business-ready dimensions and facts that Lecture 2 builds and the mini-project ships.

## 3. Staging — five views, no business logic yet

Staging's job, per source table, is narrow: cast types correctly, standardize vocabulary, and — critically for this dataset — **decide what "as of" means**. Build all five as views:

```sql
-- 1. Users: no cleanup needed beyond consistent casing on channel
CREATE VIEW stg_users AS
SELECT
    user_id,
    LOWER(TRIM(email))       AS email,
    company_name,
    signup_date,
    LOWER(signup_channel)    AS signup_channel,
    plan_at_signup
FROM raw_users;

-- 2. Events: rename the one event this dataset tracks to something self-documenting
CREATE VIEW stg_events AS
SELECT
    event_id,
    user_id,
    event_name,
    event_ts AS event_date
FROM raw_events
WHERE event_name = 'created_first_project';   -- the only activation signal in this dataset

-- 3. Marketing spend: no cleanup needed, but name it for what it'll join to
CREATE VIEW stg_marketing_spend AS
SELECT
    spend_month,
    LOWER(channel) AS channel,
    spend_usd
FROM raw_marketing_spend;

-- 4. Subscriptions: THIS is where "as of" gets decided
CREATE VIEW stg_subscriptions AS
SELECT
    subscription_id,
    user_id,
    plan_name,
    status,
    amount_usd,
    created,
    canceled_at,
    CASE WHEN status = 'active' THEN 1 ELSE 0 END AS is_active_now
FROM raw_subscriptions
WHERE created <= DATE '2025-06-30';   -- the analysis cutoff — see Section 4

-- 5. Experiment assignments: pass through, typed
CREATE VIEW stg_experiment_assignments AS
SELECT user_id, experiment_name, LOWER(variant) AS variant, assigned_at
FROM raw_experiment_assignments;
```

**SQLite note:** all five views run unchanged — no Postgres-only syntax used here.

## 4. The cutoff filter is not a detail — it's the whole lesson

Look again at `stg_subscriptions`'s `WHERE created <= DATE '2025-06-30'`. Run the raw count against the staged count:

```sql
SELECT COUNT(*) FROM raw_subscriptions;   -- 35
SELECT COUNT(*) FROM stg_subscriptions;   -- 30
```

Five rows disappear. They aren't wrong, deleted, or bad data — they're five June signups (Crunch Boards users 64, 66, 70, 71, 72) who converted to paying customers in **early July**, a few days *after* this capstone's analysis cutoff. This is Week 5's right-censoring lesson, wearing a different hat: **"as of 2025-06-30" is a business decision baked into a `WHERE` clause, not a fact the data hands you for free.** Every mart you build downstream of `stg_subscriptions` inherits this cutoff automatically, which is exactly the point of putting it here instead of re-deriving it in every mart query. If a stakeholder later asks "what about the customers who signed up in June but paid in July," you now have a one-sentence answer: "excluded by the analysis cutoff — here's the query that shows you exactly who, and why."

**This is also why the raw layer never gets this filter.** `raw_subscriptions` still has all 35 rows. If Crunch Boards moves the cutoff to `2025-07-15` next month, you change one line in `stg_subscriptions` and every mart downstream recomputes correctly — you don't go hunting for every place someone hardcoded "as of June."

## 5. `dim_date` — the calendar spine

Every fact table this week grains to a month (`fct_mrr_monthly`) or joins to one (`fct_acquisition`'s `signup_month`). Build one calendar dimension once, covering the analysis window plus the forecast horizon (Lecture 3 needs July–September):

```sql
CREATE TABLE dim_date (
    month_start  DATE    PRIMARY KEY,
    month_name   TEXT    NOT NULL,
    month_number INTEGER NOT NULL,
    year         INTEGER NOT NULL,
    is_forecast  BOOLEAN NOT NULL   -- FALSE for Jan-Jun (observed), TRUE for Jul-Sep (projected)
);

INSERT INTO dim_date VALUES
('2025-01-01','January',1,2025,FALSE),
('2025-02-01','February',2,2025,FALSE),
('2025-03-01','March',3,2025,FALSE),
('2025-04-01','April',4,2025,FALSE),
('2025-05-01','May',5,2025,FALSE),
('2025-06-01','June',6,2025,FALSE),
('2025-07-01','July',7,2025,TRUE),
('2025-08-01','August',8,2025,TRUE),
('2025-09-01','September',9,2025,TRUE);
```

Nine rows, hand-written, is completely fine — this is a small, fixed calendar, not a general-purpose one. Don't reach for `generate_series` machinery to build nine rows you can type in fifteen seconds; save that tool for when the range is actually large or dynamic.

## 6. `dim_user` — one row per customer, decorated with cohort

`dim_user` is `stg_users` with the two derived attributes every downstream mart will want: `signup_month` (for cohorting) and a flag for whether the user is in the capstone experiment.

```sql
CREATE TABLE dim_user AS
SELECT
    u.user_id,
    u.email,
    u.company_name,
    u.signup_date,
    DATE(u.signup_date, 'start of month')   AS signup_month,   -- SQLite
    -- DATE_TRUNC('month', u.signup_date)::date AS signup_month,  -- Postgres
    u.signup_channel,
    u.plan_at_signup,
    CASE WHEN e.user_id IS NOT NULL THEN 1 ELSE 0 END AS in_experiment
FROM stg_users u
LEFT JOIN stg_experiment_assignments e ON e.user_id = u.user_id;
```

Verify it landed correctly:

```sql
SELECT signup_channel, COUNT(*) FROM dim_user GROUP BY signup_channel;
-- paid_search  30
-- organic      24
-- referral     18

SELECT in_experiment, COUNT(*) FROM dim_user GROUP BY in_experiment;
-- 0   48
-- 1   24
```

Twenty-four users — the May and June cohorts — are in `onboarding_checklist_v2`. Forty-eight are not. Hold onto that `in_experiment` flag; Lecture 2 uses it to keep the experiment's control/treatment comparison from accidentally leaking into your channel-level funnel numbers (an experiment cell is not a customer segment, and mixing them is a classic capstone mistake — Lecture 2, Section 5 shows exactly how it goes wrong).

## 7. Start the metric-definitions glossary now, not later

Week 6 taught that a semantic layer, in this course, is two things: the mart's SQL definition, and a written glossary. Start the glossary **today**, as `metrics.md`, even though most of the metrics it will define don't have mart tables yet. Write down what you already know for certain:

```markdown
# Crunch Boards — Metric Definitions

**Analysis cutoff:** 2025-06-30. Any subscription created after this date is excluded
from every mart in this capstone (see stg_subscriptions). Raw data is not affected.

**Activated user:** a user with at least one `created_first_project` event
(the only activation signal this dataset tracks) — source: stg_events.

**Paying customer:** a user with a row in stg_subscriptions where status = 'active'
as of the cutoff — source: stg_subscriptions.

**Signup channel:** dim_user.signup_channel, fixed at signup, never re-attributed later.
```

You will add MRR, CAC, LTV, and the experiment's success metric to this file across Lectures 2 and 3. Writing definitions *as you build*, instead of reconstructing them from memory the night before the mini-project is due, is the single highest-leverage habit this capstone can teach you — every real growth team you'll ever work on either has this document or desperately needs one.

## 8. Check yourself

- Why does a capstone scope statement belong in one sentence, and what's the risk of skipping it?
- What does `stg_subscriptions`'s `WHERE created <= DATE '2025-06-30'` actually decide, and why does that decision belong in staging rather than raw or every downstream mart query?
- Five rows exist in `raw_subscriptions` but not in `stg_subscriptions`. Are they bad data? What are they?
- Why is `dim_date` hand-written here instead of built with `generate_series` or a recursive CTE?
- What does `dim_user.in_experiment` protect you from getting wrong in Lecture 2?
- Why start `metrics.md` in Lecture 1 instead of waiting until the mini-project?

If those are automatic, Lecture 2 builds the marts — `fct_acquisition`, `fct_mrr_monthly`, `fct_unit_economics`, and `fct_experiment_results` — that turn this staged, dated, cohorted data into the growth system the capstone actually ships.

## Further reading

- **PostgreSQL — `DATE_TRUNC`:** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **SQLite — Date and Time Functions:** <https://www.sqlite.org/lang_datefunc.html>
- **PostgreSQL — `CREATE VIEW`:** <https://www.postgresql.org/docs/current/sql-createview.html>
- **PostgreSQL — `CREATE TABLE AS`:** <https://www.postgresql.org/docs/current/sql-createtableas.html>
