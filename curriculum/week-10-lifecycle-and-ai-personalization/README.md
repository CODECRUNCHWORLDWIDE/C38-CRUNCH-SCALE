# Week 10 — Lifecycle & AI Personalization

> **Goal:** by Sunday you can take a warehouse full of customer events, turn it into a churn model that actually beats guessing, translate that model's scores into next-best-action rules a support and marketing team can run on, and prove — against a randomized holdout, not a hunch — that the resulting lifecycle campaign moved the number.

Nine weeks of this course have been about **measuring** the business: funnels, retention, unit economics, a warehouse you can trust, segments, experiments, pricing. This week you start **acting** on it at scale. A human can personally check in on 20 at-risk accounts. Crunch Flow now has hundreds of them, growing every week. The only way to give every one of them the right nudge — not a blast, not a guess, not a spreadsheet a CSM half-remembers to update — is to score them with a model, route the score through a rules engine, and fire a lifecycle journey that a randomized holdout proves actually works.

This week's fictional company is still **Crunch Flow**, the B2B project-management SaaS from Week 6's warehouse build. It's grown: 260 customers on the books, eight months of usage and billing history. Some of them are quietly disengaging right now, and you don't yet know which ones. That's the job this week.

**Every table this week lives in SQL (SQLite or PostgreSQL) and every model lives in pandas + scikit-learn. No spreadsheet touches a customer record — that is a hard rule of this course.** Spreadsheets are taught only in C41 Crunch Excel, and never as a system of record.

## Learning objectives

By the end of this week, you will be able to:

- **Frame** churn and propensity as supervised prediction problems: define the label precisely (what counts as "churned," over what window), separate the **observation window** from the **label window**, and explain why getting this boundary wrong silently leaks the future into your features.
- **Engineer features** from warehouse events in SQL — recency, frequency, tenure, support friction, payment health, engagement trend — computed strictly *as of* a cutoff date, with no peeking past it.
- **Train and evaluate** a churn model in scikit-learn (logistic regression and random forest), and judge it with the metrics that matter for an imbalanced, action-driving problem: ROC-AUC, PR-AUC, precision/recall at a chosen threshold — not bare accuracy.
- **Build next-best-action logic** that crosses a churn score with customer value into a small decision table, so every customer gets exactly one action, and low-value customers don't get an expensive human touch a script could handle.
- **Design and reason about lifecycle campaigns** — trigger-based journeys, not calendar blasts — and **measure their lift** against a randomized holdout, including recognizing when a test is too small to trust its own result.

## Prerequisites

- **Weeks 1–9 of this course**, especially Week 6 (the warehouse: raw → staging → marts, idempotent loads) and Week 8 (experiment design, holdouts, significance) — this week is where those two threads merge into one system.
- Comfortable with SQL joins, aggregation, `CASE WHEN`, and window functions ([C33 Crunch SQL](../../../C33-CRUNCH-SQL/)).
- Basic Python and pandas (`DataFrame`, `groupby`, `merge`). No prior machine-learning experience assumed — scikit-learn's model-fitting API is taught from first principles this week.
- PostgreSQL 16+ (primary) or SQLite 3.35+ (fallback), plus Python 3.10+ with `pandas` and `scikit-learn` installed (`pip install pandas scikit-learn`). **No spreadsheet software is used anywhere this week.**

## Set up the seed database (do this first)

Pasting nine thousand `INSERT` rows into a README doesn't teach anyone anything — so this week the seed data comes from a **deterministic generator script** instead of a giant SQL literal. Same idea as every other week (one seed dataset, used all week), different delivery: you run a small, seeded Python script once, it writes the raw tables, and because every random draw uses a fixed seed, your database is byte-for-byte the same as the one every number in this week's materials was computed from.

Save this as `seed_crunch_flow_scale.py`:

```python
import sqlite3, random, math, datetime as dt
import numpy as np
import pandas as pd

random.seed(42)
np.random.seed(42)

N = 260
START = dt.date(2025, 1, 1)
CUTOFF = dt.date(2025, 9, 1)     # today, for scoring purposes
LABEL_END = dt.date(2025, 12, 31)  # end of the 4-month label window

plans = [("Starter", 29.00), ("Growth", 99.00), ("Scale", 299.00)]
plan_weights = [0.5, 0.35, 0.15]
countries = ["USA","Canada","UK","Germany","Ireland","Spain","Sweden","Norway",
             "Mexico","South Korea","Australia","France"]
first_names = ["Ada","Ben","Cara","Drew","Elin","Felix","Gina","Hugo","Ines","Jae",
    "Kira","Liam","Mira","Noel","Opal","Priya","Quentin","Rosa","Sam","Talia","Uma",
    "Victor","Wren","Xander","Yara","Zane","Owen","Nina","Marco","Lena","Ivo","Hana",
    "Grant","Freya","Ezra","Dana","Cole","Bea","Aria","Theo"]
last_names = ["Byrne","Ortiz","Voss","Kim","Berg","Marlowe","Ruiz","Baumgarten",
    "Del Mar","Hanuri","Novak","Walsh","Patel","Ford","Reyes","Shah","Diaz","Okafor",
    "Lindgren","Park","Costa","Bianchi","Weber","Nilsson","Haddad","Fischer","Moreau",
    "Nakamura","Ibrahim","Sorensen"]
companies = ["Northwind Traders","Lumen Studio","Fjord Tech","BrightPath Consulting",
    "Stavro AB","Marlowe & Co","Rio Verde Software","Baumgarten GmbH","Del Mar Analytics",
    "Hanuri Labs","NovaSys","Westfold Digital","Suncrest Farms","Tidewater Logistics",
    "BrightLoop","Parallax Media","Coastline Freight","Underline Studio","Foundry Robotics",
    "Lattice Legal","Meridian Health","Cobalt Interiors","Redwood Realty","Sable & Finch",
    "Amber Analytics"]

def sigmoid(x): return 1 / (1 + math.exp(-x))

rows_users, rows_subs, rows_events = [], [], []
event_id = 1
def add_event(uid, name, ts):
    global event_id
    rows_events.append((event_id, uid, name, ts.isoformat(" ", timespec="seconds")))
    event_id += 1

for uid in range(1, N + 1):
    plan, price = random.choices(plans, weights=plan_weights, k=1)[0]
    is_annual = random.random() < 0.22
    signup_offset = random.randint(0, 212)          # Jan 1 – Jul 31 2025
    signup_date = START + dt.timedelta(days=signup_offset)
    fname, lname = random.choice(first_names), random.choice(last_names)
    company = random.choice(companies)
    country = random.choice(countries)
    email = f"{fname.lower()}.{lname.lower().replace(' ','')}{uid}@{company.split()[0].lower()}.com"
    rows_users.append((uid, email, f"{fname} {lname}", company, country,
                        signup_date.isoformat(), plan))

    mrr = price if not is_annual else round(price * 10 / 12, 2)   # annual billed 10x monthly, amortized
    billing_interval = "year" if is_annual else "month"

    # ---- latent customer-health traits (not visible to the model directly —
    #      only through the events and support tickets they generate) ----
    engagement = np.clip(np.random.beta(2.3, 2.3), 0.03, 0.98)
    support_friction = np.clip(np.random.beta(1.3, 5.0), 0.0, 1.0)
    payment_trouble = random.random() < 0.15

    # ---- early abandon: trial fizzle, independent of the label-window drivers ----
    early_abandon = random.random() < 0.09
    canceled_at, status = None, "active"
    if early_abandon:
        candidate = signup_date + dt.timedelta(days=random.randint(15, 50))
        if candidate < CUTOFF:
            canceled_at, status = candidate, "canceled"

    eligible_for_label = canceled_at is None
    churned_in_window = False
    if eligible_for_label:
        plan_risk = {"Starter": 1.15, "Growth": 0.15, "Scale": -1.05}[plan]
        annual_risk = -1.35 if is_annual else 0.0
        tenure_at_cutoff = (CUTOFF - signup_date).days
        short_tenure_risk = 1.0 if tenure_at_cutoff < 60 else (0.35 if tenure_at_cutoff < 120 else 0.0)
        logit = (-2.35 + plan_risk + annual_risk + short_tenure_risk
                 + 2.9 * support_friction - 2.6 * (engagement - 0.5)
                 + (1.55 if payment_trouble else 0.0)
                 + np.random.normal(0, 0.55))
        churned_in_window = np.random.random() < sigmoid(logit)
        if churned_in_window:
            canceled_at = CUTOFF + dt.timedelta(days=random.randint(3, (LABEL_END - CUTOFF).days - 1))
            status = "canceled"

    # ---- observable events, Jan–Aug (pre-cutoff), driven by the SAME latent traits ----
    for m in pd.date_range("2025-01-01", "2025-08-01", freq="MS"):
        m_date = m.date()
        if m_date < signup_date or (canceled_at is not None and m_date > canceled_at):
            continue
        for _ in range(np.random.poisson(lam=max(0.3, engagement * 10.5))):
            day = random.randint(0, 27)
            ts = dt.datetime.combine(m_date, dt.time(random.randint(8, 19), random.randint(0, 59))) + dt.timedelta(days=day)
            if ts.date() < CUTOFF:
                add_event(uid, "feature_used", ts)
        for _ in range(np.random.poisson(lam=support_friction * 2.1)):
            day = random.randint(0, 27)
            ts = dt.datetime.combine(m_date, dt.time(random.randint(8, 19), random.randint(0, 59))) + dt.timedelta(days=day)
            if ts.date() < CUTOFF:
                add_event(uid, "support_ticket_opened", ts)
    if payment_trouble:
        elig = [m for m in pd.date_range("2025-01-01", "2025-08-01", freq="MS")
                if m.date() >= signup_date + dt.timedelta(days=20) and m.date() < CUTOFF
                and (canceled_at is None or m.date() <= canceled_at)]
        if elig and random.random() < 0.75:
            m_date = random.choice(elig).date()
            ts = dt.datetime.combine(m_date, dt.time(10, 0)) + dt.timedelta(days=random.randint(0, 20))
            if ts.date() < CUTOFF:
                add_event(uid, "payment_failed", ts)

    # events during the label window too (Sep–Dec) — realism for the raw log only
    if canceled_at is None or canceled_at > CUTOFF:
        for m in pd.date_range("2025-09-01", "2025-12-01", freq="MS"):
            m_date = m.date()
            if canceled_at is not None and m_date > canceled_at:
                continue
            for _ in range(np.random.poisson(lam=max(0.2, engagement * 9))):
                day = random.randint(0, 27)
                ts = dt.datetime.combine(m_date, dt.time(random.randint(8, 19), random.randint(0, 59))) + dt.timedelta(days=day)
                if ts.date() <= LABEL_END:
                    add_event(uid, "feature_used", ts)
    if canceled_at is not None:
        add_event(uid, "canceled", dt.datetime.combine(canceled_at, dt.time(9, 0)))

    rows_subs.append((uid, plan, mrr, billing_interval, status, signup_date.isoformat(),
                       canceled_at.isoformat() if canceled_at else None))

conn = sqlite3.connect("crunch_flow_scale.db")
cur = conn.cursor()
cur.executescript("""
DROP TABLE IF EXISTS raw_users;
DROP TABLE IF EXISTS raw_subscriptions;
DROP TABLE IF EXISTS raw_events;
CREATE TABLE raw_users (
    user_id INTEGER PRIMARY KEY, email TEXT NOT NULL, full_name TEXT NOT NULL,
    company_name TEXT NOT NULL, country TEXT NOT NULL, signup_date DATE NOT NULL,
    plan_at_signup TEXT NOT NULL
);
CREATE TABLE raw_subscriptions (
    user_id INTEGER PRIMARY KEY, plan TEXT NOT NULL, mrr_usd NUMERIC NOT NULL,
    billing_interval TEXT NOT NULL, status TEXT NOT NULL, start_date DATE NOT NULL,
    canceled_at DATE
);
CREATE TABLE raw_events (
    event_id INTEGER PRIMARY KEY, user_id INTEGER NOT NULL, event_name TEXT NOT NULL,
    event_ts TIMESTAMP NOT NULL
);
""")
cur.executemany("INSERT INTO raw_users VALUES (?,?,?,?,?,?,?)", rows_users)
cur.executemany("INSERT INTO raw_subscriptions VALUES (?,?,?,?,?,?,?)", rows_subs)
cur.executemany("INSERT INTO raw_events VALUES (?,?,?,?)", rows_events)
conn.commit()
conn.close()
print("Seeded crunch_flow_scale.db —", len(rows_users), "users,", len(rows_events), "events.")
```

Run it once:

```bash
python3 seed_crunch_flow_scale.py
```

Sanity check — this should print `260 users, 9095 events.` If you get different numbers, your Python/NumPy version is drawing random numbers in a different order than expected; re-check you copied the script exactly, in order (every `random.*`/`np.random.*` call must fire in the same sequence to reproduce the same database).

**PostgreSQL users:** load the same three tables via `psycopg2` with the same DDL (types translate directly — `TEXT`, `NUMERIC`, `DATE`, `TIMESTAMP` are valid in both engines) or export/import via `pgloader`/CSV. Every SQL example this week runs unchanged on both engines.

Two facts baked into this data **on purpose**, load-bearing for the whole week:

1. **`CUTOFF = 2025-09-01`** is "today." Everything you compute for modeling uses only `raw_events` rows with `event_ts < CUTOFF` and `raw_subscriptions` state as of that date. `LABEL_END = 2025-12-31` is the end of a 4-month **label window** — what you're trying to predict, and data you are **not allowed to touch** while building features. This observation/label split is the single most important discipline in this week's materials.
2. **24 customers already canceled before the cutoff** (an independent "early abandon" trait, unrelated to the label-window drivers) — they're excluded from the modeling population entirely, leaving **236 eligible customers** active as of September 1st. Of those, **87 churn during the label window** (a 36.9% base rate) — high, but realistic for an "at the moment we're scoring" population that specifically excludes already-departed customers.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed data; frame churn as prediction | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Feature engineering from events in SQL | 0h | 2h | 0h | 0.5h | 1h | 0h | 3.5h |
| Wednesday | Train + evaluate the churn model | 2h | 2h | 0h | 0.5h | 1h | 0h | 5.5h |
| Thursday | Personalization, next-best-action | 2h | 1h | 1.5h | 0.5h | 1h | 0.5h | 6.5h |
| Friday | Lifecycle campaigns, holdouts, lift | 2h | 0h | 1.5h | 0.5h | 1h | 1.5h | 6.5h |
| Saturday | Mini-project (score → NBA → win-back → lift) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **6h** | **3h** | **3.5h** | **5h** | **4.5h** | **30h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it, and every example targets the `crunch_flow_scale.db` you just seeded.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-churn-and-propensity-models.md](./lecture-notes/01-churn-and-propensity-models.md) | Framing churn as supervised prediction, observation/label windows, leakage, SQL feature engineering, training + evaluating in scikit-learn | 2h |
| 2 | [lecture-notes/02-personalization-and-nba.md](./lecture-notes/02-personalization-and-nba.md) | Risk bands, value tiers, the next-best-action decision table, guardrails against over-messaging | 2h |
| 3 | [lecture-notes/03-lifecycle-campaigns.md](./lecture-notes/03-lifecycle-campaigns.md) | Lifecycle stages, trigger-based journeys, randomized holdouts, measuring lift, statistical power | 2h |
| 4 | [exercises/exercise-01-engineer-churn-features.md](./exercises/exercise-01-engineer-churn-features.md) | Build the SQL feature table from `raw_events`/`raw_subscriptions` as of the cutoff | 2h |
| 5 | [exercises/exercise-02-train-a-churn-model.md](./exercises/exercise-02-train-a-churn-model.md) | Train logistic regression + random forest, evaluate with ROC-AUC/PR-AUC | 2h |
| 6 | [exercises/exercise-03-map-scores-to-actions.md](./exercises/exercise-03-map-scores-to-actions.md) | Score the full population, build risk × value next-best-action table | 1h |
| 7 | [challenges/challenge-01-build-a-winback-journey.md](./challenges/challenge-01-build-a-winback-journey.md) | Design the trigger, journey steps, and randomized holdout split for a win-back segment | 1.5h |
| 8 | [challenges/challenge-02-measure-campaign-lift.md](./challenges/challenge-02-measure-campaign-lift.md) | Compute observed lift, run a significance check, size the test that would actually detect it | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Ship the full pipeline: warehouse → features → model → scores → NBA → win-back segment → lift | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice: an expansion-propensity model, a different label window, expected-value sizing | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Explain, precisely, what "churn" means for a given model — the exact cutoff, the exact label window — and why that boundary is where leakage hides.
- Write SQL that turns a raw event log into a leakage-free feature table, and Python that turns that feature table into a scored population.
- Defend a model's evaluation with ROC-AUC, PR-AUC, and a precision/recall tradeoff appropriate to the cost of a false positive vs. a false negative — not a bare accuracy number.
- Build a next-best-action table that a support and marketing team could run from tomorrow, with guardrails against wasting an expensive human touch on a customer a script could save.
- Design a trigger-based lifecycle journey with a real randomized holdout, measure its lift honestly, and say out loud when a test is too small to trust.

## Up next

**Week 11 — Forecasting growth** — with a warehouse, a churn model, and a lifecycle program now running, you turn the same event and revenue data into a time-series forecast and a scenario model for where the business is headed.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
