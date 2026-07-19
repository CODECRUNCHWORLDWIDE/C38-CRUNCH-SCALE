# Mini-Project — Instrument an Event Schema and Ship a Baseline Metrics Query Set

> Design and load a `users` + `events` schema for a product **of your own choosing**, pick its north star and input metrics using this week's five-question test, and ship the SQL query set that computes all of them straight from raw events in PostgreSQL. This is the week's capstone — proof you can do the entire Week 1 workflow from a blank page, not just follow along against StreakLab.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

Every lecture, exercise, and challenge this week handed you StreakLab already instrumented. Real growth work starts before that — someone has to design the schema, decide what counts as an event, and pick the north star in the first place. This mini-project is that first mile, done once, completely, by you.

---

## Deliverable

A directory in your portfolio `c38-week-01/mini-project/` containing:

1. `schema.sql` — your `CREATE TABLE` statements for `users` and `events` (same two-table shape as StreakLab, adapted to your product).
2. `seed.sql` — your `INSERT` statements: **at least 8 users** and **at least 50 events**, following the archetype-pattern approach from the week README (a few different, deliberately distinct user behavior patterns — don't make every user identical, or your metrics will have nothing interesting to say).
3. `metrics.sql` — your baseline query set: **one query for the north star, and one query for each of at least 4 input metrics**, all commented with what they compute.
4. `report.md` — for each metric: the actual result (a real number, run against your own seed data) and one sentence of interpretation.
5. `design-notes.md` — see "Design notes" below.

Everything must run against your own `users`/`events` tables on PostgreSQL (SQLite acceptable as a fallback, noted in `report.md`).

---

## Step 1 — Pick a product (30 min)

Pick **one**:

- **Your own idea** — anything with users who do repeated things (a game, a marketplace, a media app, a B2B tool). Prefer something you understand well enough to invent realistic behavior for.
- **Suggested option A — "PocketLedger"**: a personal budgeting app. Free tier tracks manual transactions; a $6/month Plus tier adds bank-account auto-sync.
- **Suggested option B — "TrailMate"**: a hiking-route-sharing app. Free to browse; users can post routes and follow other users; no paid tier, monetized by affiliate gear links (an `affiliate_click` event with a `event_value` payout).
- **Suggested option C — "DeskPing"**: a B2B async-standup tool for small teams. Team-based signup (like the B2B example in Exercise 1), priced per active seat.

Write two sentences in `design-notes.md`: what the product does, and who its user is.

## Step 2 — Design the schema (30 min)

Design `users` (dimension) and `events` (fact), following Lecture 3, section 2's rules exactly:

- `users`: primary key, a `signup_at` timestamp, a `signup_channel`, and at least one more attribute relevant to your product (StreakLab used `country`; yours might be `plan_at_signup`, `team_size`, whatever fits).
- `events`: primary key, foreign key to `users`, `event_name` (a **fixed vocabulary of 5–7 values** — write them all out in `design-notes.md` before writing a single `INSERT`), `event_time`, and an `event_value` column for the one or two event types that carry a number (a payment, a payout, a score).

In `design-notes.md`, list your `event_name` vocabulary as a table: name, meaning, which AARRR stage it belongs to (Acquisition/Activation/Retention/Revenue/Referral).

## Step 3 — Pick the north star (30 min)

Run **two candidates** through Lecture 1's five-question test in `design-notes.md`, exactly like the worked StreakLab example. Choose one and write its precise definition — precise enough that the query in Step 5 is the *only possible* correct implementation of your sentence.

## Step 4 — Seed realistic data (45 min)

Write `seed.sql`. Use the archetype approach from this week's seed data: instead of randomizing every user independently, design **3–5 distinct behavior patterns** (a power user, a user who never activates, a user who churns, etc.), and apply each pattern to a few users. This keeps your data internally consistent enough that you can sanity-check your own metric queries by reasoning about them, the same way you checked StreakLab's numbers all week.

Minimum bar: **8 users, 50 events, at least 3 distinct `event_name` values represented, at least one revenue or referral event.**

## Step 5 — Write the metric query set (45 min)

In `metrics.sql`: the north-star query, plus queries for at least 4 input metrics that plausibly roll up into it (mirror the structure from Lecture 3, section 4 — CTEs are encouraged). Run each, and copy the real result into `report.md` next to a one-sentence interpretation.

---

## Rules

- **No spreadsheets, anywhere, at any point.** If you found yourself wanting to sketch your seed data in Excel first — don't; sketch it as a markdown table in `design-notes.md` instead, then translate straight to `INSERT` statements.
- **Every metric must be a runnable SQL query**, not a hand-computed number typed into `report.md`.
- **State your assumptions.** Any judgment call (a threshold, a time window, what counts as "active") gets one sentence in `design-notes.md` explaining the choice, the same discipline Challenge 2 demanded of StreakLab's WEU.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Schema design | 20% | Clean dimension/fact split; fixed, sensible `event_name` vocabulary; correct types |
| North-star selection | 20% | Both candidates genuinely tested against all five questions; final choice is defensible, not just convenient |
| Seed data realism | 20% | Distinct, deliberate behavior patterns — not uniform random noise |
| Metric query set | 25% | North star + 4+ input metrics, all correct, all runnable, all commented |
| Documentation & honesty | 15% | `design-notes.md` states real assumptions; `report.md` interprets numbers in plain English |

---

## Reflection (part of `design-notes.md`, ~200 words)

1. What was harder — designing the schema, or picking the north star? Why?
2. Where did you catch yourself wanting to reach for a spreadsheet, and what stopped you?
3. If you had 10x the users and events, which of your metric queries would you worry might get slow, and why? (You don't need to fix it yet — Week 7 is query performance. Just notice it.)
4. One thing about your chosen product's growth you genuinely can't answer with a `users`/`events` two-table schema alone — what table would you need to add?

---

## Why this matters

Every later week in C38 assumes you can do exactly this: take a business, without anyone handing you a pre-built dataset, and turn "how do we know if we're growing" into a schema, a north star, and a query set. StreakLab's seed data was training wheels. This project is the first mile without them — and Week 2 starts by taking whichever product you pick here (or StreakLab, your choice) and building real multi-touch attribution on top of its acquisition events.

When done: push, then take the [quiz](../quiz.md) and start [Week 2 — Acquisition funnels & attribution](../../week-02-acquisition-funnels-and-attribution/).
