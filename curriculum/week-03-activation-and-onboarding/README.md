# Week 3 — Activation & Onboarding

> **Goal:** by Sunday you can look at a raw events table for any product, define a defensible activation event, empirically find the "aha moment" that separates users who stick from users who churn, turn onboarding into a step funnel, and locate — in SQL, with numbers — exactly where new users stall.

Acquisition gets people in the door. Activation is what happens in the first few minutes and days that decides whether they ever come back. It is the highest-leverage stage in the growth funnel: fix a leaky activation step and every acquisition dollar you already spent gets more valuable; leave it broken and no amount of new signups will save you. This week you stop guessing at "the aha moment" and start finding it the way a growth analyst actually does — by correlating early behavior with real retention, in SQL, against real data.

We work against one running example all week: **Crunch Boards**, a small Kanban-style project tool. You have its full event stream — 400 signups, ~5,000 events — and you'll use it for every lecture, exercise, challenge, and the mini-project.

## Learning objectives

By the end of this week, you will be able to:

- **Define** an activation event that is specific, early, observable, and — critically — grounded in a measured link to downstream retention, not a guess.
- **Find** the aha moment empirically: correlate early user actions against long-run retention in SQL and read the resulting lift table without fooling yourself on reach, causality, or reverse causation.
- **Measure** time-to-value (TTV) — the gap between signup and the activation event — as a full distribution (percentiles, buckets), not a single average.
- **Break** an onboarding flow into an ordered step funnel, compute step-over-step conversion, and pinpoint the single biggest drop with SQL.
- **Propose** onboarding changes tied to a quantified, numbers-backed activation and retention lift — and know the difference between a *projection* and a *proof* (that's Week 8).

## Prerequisites

- Comfortable with SQL through joins, aggregation (`GROUP BY`/`HAVING`), CTEs, and basic window functions — the level of [C33 Crunch SQL](../../../C33-CRUNCH-SQL/), Weeks 1–4.
- Weeks 1–2 of this course (north-star metrics, AARRR, funnels) — this week is "Activation," the second letter of AARRR.
- Basic Python (you'll run one seed-generation script; you don't need to understand every line, just run it).
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.9+ with no extra packages (the generator uses only the standard library).

## Set up the seed data (do this first)

Everything this week runs against two tables: `users` (who signed up, when, from where) and `events` (everything they did afterward — onboarding steps and ongoing product usage). Rather than hand-typing thousands of rows, we **generate** them with a small, seeded (deterministic) Python script — this is also exactly how real growth teams build synthetic seed data for testing pipelines.

**1. Save this as `generate_seed.py` and run it (`python3 generate_seed.py`).** It writes `users.csv` and `events.csv` in the current directory.

```python
import random, csv, datetime as dt

random.seed(38)

N_USERS = 400
START = dt.date(2025, 1, 1)
SIGNUP_WINDOW_DAYS = 70   # users sign up over 10 weeks
TRACK_WEEKS = 12          # each user tracked for 12 weeks post-signup

CHANNELS = ['Organic Search', 'Paid Search', 'Referral', 'Content', 'Social']
CHANNEL_W = [35, 20, 15, 20, 10]
COUNTRIES = ['USA', 'UK', 'Canada', 'Germany', 'India', 'Brazil', 'Australia', 'Spain']
COUNTRY_W = [30, 15, 10, 15, 10, 8, 6, 6]
PLANS = ['Free', 'Pro']
PLAN_W = [70, 30]

users = []
for uid in range(1, N_USERS + 1):
    offset = random.randint(0, SIGNUP_WINDOW_DAYS - 1)
    signup_date = START + dt.timedelta(days=offset)
    signup_time = dt.datetime.combine(signup_date, dt.time(hour=random.randint(6, 22), minute=random.randint(0, 59)))
    channel = random.choices(CHANNELS, weights=CHANNEL_W)[0]
    country = random.choices(COUNTRIES, weights=COUNTRY_W)[0]
    plan = random.choices(PLANS, weights=PLAN_W)[0]
    users.append({'user_id': uid, 'signup_at': signup_time, 'channel': channel, 'country': country, 'plan': plan})

events = []
event_id = 1

def add_event(uid, name, t):
    global event_id
    events.append({'event_id': event_id, 'user_id': uid, 'event_name': name, 'event_at': t})
    event_id += 1

for u in users:
    uid = u['user_id']
    signup_time = u['signup_at']
    add_event(uid, 'signed_up', signup_time)

    verified = random.random() < 0.78
    t_verified = None
    if verified:
        t_verified = signup_time + dt.timedelta(minutes=random.randint(2, 600))
        add_event(uid, 'verified_email', t_verified)

    created_board = verified and random.random() < 0.83
    t_board = None
    if created_board:
        t_board = signup_time + dt.timedelta(hours=random.uniform(0.2, 48))
        add_event(uid, 'created_first_board', t_board)

    created_card = created_board and random.random() < 0.80
    t_card = None
    if created_card:
        t_card = t_board + dt.timedelta(hours=random.uniform(0.1, 72))
        add_event(uid, 'created_first_card', t_card)

    invited = created_card and random.random() < 0.60
    t_invited = None
    if invited:
        t_invited = t_card + dt.timedelta(hours=random.uniform(0.5, 96))
        add_event(uid, 'invited_teammate', t_invited)

    completed_task = invited and random.random() < 0.85
    if completed_task:
        t_completed = t_invited + dt.timedelta(hours=random.uniform(0.5, 120))
        add_event(uid, 'completed_first_task', t_completed)

    connect_p = 0.20 if invited else 0.03
    if random.random() < connect_p:
        t_conn = signup_time + dt.timedelta(days=random.uniform(3, 30))
        add_event(uid, 'connected_integration', t_conn)

    base_p = 0.10
    if verified: base_p += 0.05
    if created_board: base_p += 0.05
    if created_card: base_p += 0.05
    if invited: base_p += 0.35
    if completed_task: base_p += 0.15
    base_p = min(base_p, 0.92)

    for week in range(0, TRACK_WEEKS):
        p = base_p * (0.94 ** week)
        if random.random() < p:
            n_opens = random.randint(1, 5)
            for _ in range(n_opens):
                day_offset = week * 7 + random.randint(0, 6)
                t = signup_time + dt.timedelta(days=day_offset, hours=random.uniform(0, 20))
                add_event(uid, 'app_open', t)

with open('users.csv', 'w', newline='') as f:
    w = csv.DictWriter(f, fieldnames=['user_id', 'signup_at', 'channel', 'country', 'plan'])
    w.writeheader()
    for u in users:
        w.writerow(u)

with open('events.csv', 'w', newline='') as f:
    w = csv.DictWriter(f, fieldnames=['event_id', 'user_id', 'event_name', 'event_at'])
    w.writeheader()
    for e in events:
        w.writerow(e)

print(f"users.csv: {len(users)} rows | events.csv: {len(events)} rows")
```

Running it should print: `users.csv: 400 rows | events.csv: 5017 rows`. The seed (`38`) is fixed, so your CSVs are byte-identical to everyone else's — every number quoted in this week's materials was computed against this exact dataset.

**2. Load it — PostgreSQL:**

```bash
createdb crunch_scale
psql crunch_scale
```

```sql
CREATE TABLE users (
    user_id    INTEGER PRIMARY KEY,
    signup_at  TIMESTAMP NOT NULL,
    channel    TEXT NOT NULL,
    country    TEXT NOT NULL,
    plan       TEXT NOT NULL
);

CREATE TABLE events (
    event_id   INTEGER PRIMARY KEY,
    user_id    INTEGER NOT NULL REFERENCES users(user_id),
    event_name TEXT NOT NULL,
    event_at   TIMESTAMP NOT NULL
);

\copy users FROM 'users.csv' WITH (FORMAT csv, HEADER true);
\copy events FROM 'events.csv' WITH (FORMAT csv, HEADER true);
```

**Load it — SQLite:**

```bash
sqlite3 crunch_scale.db
```

```sql
CREATE TABLE users (
    user_id    INTEGER PRIMARY KEY,
    signup_at  TEXT NOT NULL,
    channel    TEXT NOT NULL,
    country    TEXT NOT NULL,
    plan       TEXT NOT NULL
);

CREATE TABLE events (
    event_id   INTEGER PRIMARY KEY,
    user_id    INTEGER NOT NULL REFERENCES users(user_id),
    event_name TEXT NOT NULL,
    event_at   TEXT NOT NULL
);
```

```
.mode csv
.import --skip 1 users.csv users
.import --skip 1 events.csv events
```

**Sanity check** — this should print `400` and `5017`:

```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM events;
```

The event vocabulary you'll see in `event_name`: `signed_up`, `verified_email`, `created_first_board`, `created_first_card`, `invited_teammate`, `completed_first_task`, `connected_integration`, and the recurring `app_open` (product usage, used to measure retention). Not every user reaches every onboarding event — that's the funnel you're about to diagnose.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed setup; what activation really is | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Finding the aha moment empirically | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Onboarding funnel diagnosis; TTV | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Exercises wrap; Challenge 1 | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Challenge 2; homework | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (diagnose the funnel) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-what-is-activation.md](./lecture-notes/01-what-is-activation.md) | Why activation is the highest-leverage funnel stage; criteria for a good activation event | 2h |
| 2 | [lecture-notes/02-finding-the-aha-moment.md](./lecture-notes/02-finding-the-aha-moment.md) | Correlating early actions with retention in SQL; reach, lift, and the causality trap | 2h |
| 3 | [lecture-notes/03-onboarding-funnel-diagnosis.md](./lecture-notes/03-onboarding-funnel-diagnosis.md) | Step funnels, step-over-step conversion, time-to-value distributions | 2h |
| 4 | [exercises/exercise-01-define-an-activation-event.md](./exercises/exercise-01-define-an-activation-event.md) | Measure reach for each candidate event and write a justified activation-event definition | 1h |
| 5 | [exercises/exercise-02-time-to-value-distribution.md](./exercises/exercise-02-time-to-value-distribution.md) | Compute the full TTV distribution (percentiles + buckets) on both engines | 1h |
| 6 | [exercises/exercise-03-onboarding-step-dropoff.md](./exercises/exercise-03-onboarding-step-dropoff.md) | Build the funnel table and find the single biggest step-over-step drop | 1h |
| 7 | [challenges/challenge-01-find-the-aha-empirically.md](./challenges/challenge-01-find-the-aha-empirically.md) | Full lift analysis across every candidate event; defend a pick against reach + causality objections | 1.5h |
| 8 | [challenges/challenge-02-redesign-onboarding-for-lift.md](./challenges/challenge-02-redesign-onboarding-for-lift.md) | Propose an onboarding fix and quantify its projected activation/retention lift | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Diagnose a leaky onboarding funnel end to end | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the reading worth your time | — |

## By the end of this week you can…

- Read a raw event stream and propose (and defend) a specific activation event.
- Write the SQL to correlate early actions with retention and rank candidates by lift, reach, and plausibility — not just the biggest number.
- Compute a time-to-value distribution and describe its shape, not just its average.
- Build a step funnel from an event table, find the worst drop, and explain *why* it's probably happening.
- Turn a diagnosis into a quantified — and honestly labeled as *projected, not proven* — improvement estimate.

## Up next

[Week 4 — Retention & cohort analysis](../week-04-retention-and-cohort-analysis/) — once users are activated, the next question is whether they *stay*.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
