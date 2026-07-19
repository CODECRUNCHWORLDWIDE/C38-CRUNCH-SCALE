# Lecture 1 — The Modern Customer Data Stack

> **Duration:** ~2 hours. **Outcome:** You can explain, and start building, a layered warehouse — raw, staging, marts — and say precisely why each layer exists, what ELT means in practice, what a "semantic layer" is, and where RevOps owns metric definitions instead of leaving them to whoever wrote the last dashboard.

Run every example in this lecture against the `crunch_flow` database you seeded in the [week README](../README.md). If you haven't set that up yet, stop and do it now — nothing here works without it.

## 1. The question this whole week answers

Ask three people at a growing SaaS company "what's our MRR?" and you'll often get three numbers:

- **Finance** says $1,900 — they pulled it from Stripe's dashboard this morning.
- **Product** says $2,100 — they pulled it from the internal app database, which still thinks a customer who downgraded three weeks ago is on the old plan.
- **The board deck** says $1,850 — someone updated a spreadsheet by hand two Fridays ago and nobody's touched it since.

None of them are lying. Each numbers is *locally* correct for the system it came from and *globally* wrong because there's no single place where "MRR" is defined and computed once. That's the problem this week solves — not with a smarter dashboard, but with a **warehouse**: one place where raw facts land unmodified, get cleaned and conformed, and get assembled into numbers that every team queries instead of re-deriving.

## 2. The three-layer shape: raw → staging → marts

Nearly every modern data stack — whether it's built with a heavyweight tool like dbt or, as we do in this course, with **plain SQL views and tables** — uses the same three layers. The tooling is optional; the shape is not.

```
   SOURCE SYSTEMS                RAW              STAGING              MARTS
┌──────────────────┐      ┌──────────────┐  ┌──────────────────┐  ┌───────────────────┐
│ product database  │ ──▶  │ raw_users     │  │ stg_users         │  │ dim_user            │
│ event stream       │ ──▶  │ raw_events    │→ │ stg_events        │→ │ dim_date             │
│ billing (Stripe)   │ ──▶  │ raw_stripe_*  │  │ stg_subscriptions │  │ fct_mrr_monthly      │
│ internal app DB    │ ──▶  │ raw_app_*     │  │  (unified)        │  │ fct_events           │
└──────────────────┘      └──────────────┘  └──────────────────┘  └───────────────────┘
   "the truth, as each        "landed          "cleaned, typed,      "business-ready,
    system understands it"     as-is"           deduped, conformed"   documented, joinable"
```

### Raw layer — land it, don't touch it

The raw layer is a **faithful, untouched copy** of whatever the source system gave you. In `crunch_flow`, that's `raw_users`, `raw_events`, `raw_stripe_subscriptions`, and `raw_app_subscriptions` — exactly as you inserted them. Two rules:

1. **Never overwrite raw data with a "corrected" value.** If Stripe says a subscription is `canceled`, the raw table says `canceled` forever, even after you decide in staging that the app database's status is actually the one you'll trust for a specific report. The raw layer is your audit trail — if a number downstream looks wrong six months from now, you replay it from here.
2. **Land it with the source's own vocabulary.** `raw_app_subscriptions.plan` uses the legacy names `Basic`/`Pro`/`Enterprise` because that's genuinely what the app database calls them. Translating that to `Starter`/`Growth`/`Scale` is *staging's* job, not raw's.

### Staging layer — clean it, type it, dedupe it, conform it

Staging is where the mess gets tamed, one source table at a time, with **no business logic and no joins across sources yet**. A staging model:

- casts types (`TEXT` dates → real `DATE`, `NUMERIC` cents → dollars, or vice versa — pick one unit and stick to it),
- trims/lowercases things like email so joins don't silently fail on `Ada@Northwind.io` vs. `ada@northwind.io`,
- **dedupes** — Ben has two Stripe subscription rows; staging picks the current one,
- **conforms vocabulary** — maps `Basic`/`Pro`/`Enterprise` to `Starter`/`Growth`/`Scale` so every downstream query uses one plan taxonomy, and
- keeps a 1:1 (or clearly documented 1:many) relationship to its raw source — `stg_users` is still "one row per user," just cleaner.

```sql
-- A minimal staging model: one raw source in, one clean source out.
CREATE VIEW stg_users AS
SELECT
    user_id,
    LOWER(TRIM(email))      AS email,
    full_name,
    company_name,
    country,
    signup_date,
    CASE plan_at_signup
        WHEN 'Basic'      THEN 'Starter'
        WHEN 'Pro'        THEN 'Growth'
        WHEN 'Enterprise' THEN 'Scale'
        ELSE plan_at_signup
    END AS signup_plan
FROM raw_users;
```

Note this is built as a `VIEW`, not a table with an `INSERT`. That's a real, common choice: a view always reflects the current raw data with zero load step, which is perfect for staging in a small warehouse. (Marts, by contrast, are usually materialized as real tables — Lecture 3 explains why and how to keep them idempotent.)

### Marts layer — business-ready, documented, joinable

Marts are where staging models become **dimensions** (who/what/when — `dim_user`, `dim_date`, `dim_plan`) and **facts** (measurable events/snapshots — `fct_mrr_monthly`, `fct_events`, `fct_subscription_changes`). This is the layer a growth analyst, a RevOps lead, or a Python notebook actually queries. It should never require knowing which raw system a number originally came from. Lecture 2 builds this layer in depth.

## 3. ETL vs. ELT — and why this course does ELT

**ETL** (Extract, Transform, Load) transforms data *before* it lands in the warehouse — you write a script that reads from Stripe's API, reshapes it, and only then inserts clean rows. **ELT** (Extract, Load, Transform) loads the raw data first, exactly as the source gave it, and does all transformation *inside the warehouse* with SQL — which is exactly the raw → staging → marts pattern above.

| | ETL | ELT (this course) |
|---|---|---|
| Where transformation happens | Outside the warehouse, in a script | Inside the warehouse, in SQL |
| Raw data preserved? | Often not — you only keep the transformed result | Always — raw tables are untouched |
| Re-deriving a metric after a bug fix | Re-run the extraction script | Re-run a `SELECT`; raw data never moved |
| Auditability | Depends on the script | High — every layer is a queryable SQL object |

ELT wins for a RevOps warehouse because the whole point is **auditability**: when Finance asks "why does this month's MRR differ from last week's report," you want the answer to be a SQL query you can run right now against unmodified raw data, not "let me find the old script and its logs."

## 4. The semantic layer — where "MRR" gets defined once

A **semantic layer** is the (often informal, and in this course, deliberately simple) place where a business metric's definition lives exactly once, in one form everyone agrees to use. Without one, "MRR" silently means five different things across five dashboards: does it include a customer whose card just failed? Does an annual plan count as its full price or divided by 12? Does a $0 free-tier account count as a "customer" for logo-churn purposes?

In this course, the semantic layer is not a separate tool — it's:

1. **The mart's SQL definition itself** — `fct_mrr_monthly` computes MRR exactly one way, and every query reads from that fact table instead of re-deriving MRR from raw subscriptions.
2. **A written metric-definitions document** — a markdown or SQL-comment glossary that says, in plain language, "MRR = normalized monthly recurring revenue from *active* subscriptions in `stg_subscriptions`, annual plans divided by 12, as of the last day of the month." You'll write exactly this in Lecture 3 and use it again in the mini-project.

The rule of thumb: **if two people could read the metric's name and reasonably disagree about how to compute it, it needs a written definition** — and that definition should point at exactly one mart table.

## 5. Where RevOps sits in this picture

RevOps (Revenue Operations) is the function — sometimes a whole team, sometimes one person wearing that hat — that owns the handoffs between Sales, Marketing, Customer Success, and Finance, and specifically owns **the definitions**: what counts as an "active customer," how a downgrade is treated in a churn calculation, which system wins when Stripe and the product database disagree. RevOps doesn't necessarily write every line of ELT SQL, but they are accountable for:

- **Definitions:** approving what "MRR," "active," "churned," and "upgraded" mean, in writing, in the semantic layer.
- **Source-of-truth calls:** deciding, per field, which raw system wins when two disagree (you'll make exactly this call in Exercise 3 — money comes from billing, product state comes from the app database, and you document why).
- **Hygiene:** insisting the pipeline is idempotent and tested (Lecture 3) so a rerun or a late-arriving webhook doesn't silently double revenue.

This week you *are* RevOps for Crunch Flow. Every choice you make in staging and marts is a definition someone downstream will trust without re-checking it — that's the job.

## 6. A first look at all three layers, end to end

Before the deeper lectures, run this to see the shape with your own eyes. This is not the final staging/mart design (Lectures 2–3 build that properly) — it's a five-minute preview so the diagram above stops being abstract.

```sql
-- RAW: exactly what the app database says, legacy names and all
SELECT plan, status, price_usd FROM raw_app_subscriptions WHERE user_id = 9;
-- → Pro, active, 99.00   (this is STALE — see below)

-- RAW: exactly what Stripe says for the same customer
SELECT plan_name, status, amount_cents
FROM raw_stripe_subscriptions
WHERE customer_email = 'ines@delmar.es'
ORDER BY created;
-- → Growth, canceled, 9900   (the old subscription)
-- → Starter, active, 2900    (the current one — Ines downgraded)

-- A hint of staging: conform the app DB's legacy name so it's comparable at all
SELECT
    CASE plan WHEN 'Basic' THEN 'Starter' WHEN 'Pro' THEN 'Growth' WHEN 'Enterprise' THEN 'Scale' END AS plan_conformed,
    status,
    price_usd
FROM raw_app_subscriptions
WHERE user_id = 9;
-- → Growth, active, 99.00   -- now directly comparable to Stripe... and clearly wrong
```

Ines's internal record says she's still on Growth; Stripe says she downgraded to Starter three weeks ago. Nobody lied — the webhook that should have updated the app database just never fired. This is precisely the kind of drift a warehouse with a tested pipeline catches automatically, and a spreadsheet never does, because nobody re-checks a spreadsheet cell against Stripe every morning.

## 7. Check yourself

- In your own words, what's the difference between what belongs in **raw** vs. what belongs in **staging**?
- Why should raw tables never be overwritten with "corrected" values?
- What's the practical difference between ETL and ELT, and which one does a raw → staging → marts warehouse use?
- What is a "semantic layer" in this course's plain-SQL sense — is it a tool, or something else?
- Give one concrete example of a decision RevOps owns that a database schema alone can't make for you.
- Why does `stg_users` conform `Basic`/`Pro`/`Enterprise` to `Starter`/`Growth`/`Scale`, and why is that staging's job and not raw's?

If those are automatic, Lecture 2 builds the actual dimension and fact tables — `dim_user`, `dim_date`, and `fct_mrr_monthly` — that turn this raw mess into one trusted revenue number.

## Further reading

- **PostgreSQL — Views:** <https://www.postgresql.org/docs/current/sql-createview.html>
- **PostgreSQL — `CASE` expressions:** <https://www.postgresql.org/docs/current/functions-conditional.html>
- **SQLite — `CREATE VIEW`:** <https://www.sqlite.org/lang_createview.html>
- **Stripe — Subscriptions object (for a sense of real billing-system shape):** <https://stripe.com/docs/api/subscriptions>
