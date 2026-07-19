# Mini-Project — Cohort-Retention Dashboard + the One-Page Read

> Build the query set behind a real retention dashboard — the triangle, the growth-accounting split, the survival curve, and stickiness — over the seeded `users`/`events` dataset, then write the one-page memo every growth team eventually has to write: **is this product's growth compounding, or is it leaking?**

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's structured the way retention work actually happens on the job: nobody asks you for "a query." They ask you for an answer they can act on, backed by queries they can audit. You'll build four SQL views that could genuinely sit behind a live dashboard, then step back from the SQL and write the plain-English read a VP would actually get.

---

## Deliverable

A directory in your portfolio `c38-week-04/mini-project/` containing:

1. `dashboard.sql` — four labeled sections of SQL (see below), each producing one dashboard panel.
2. `dashboard_results.md` — the actual output of each query, pasted in as a table, with one sentence of interpretation under each.
3. `growth-read.md` — the one-page (300–500 word) written verdict: does Crunch's growth compound or leak? (See §3 below for what this must cover.)

Everything runs against the seed tables from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## Part 1 — The four dashboard panels (`dashboard.sql`)

Build these as four separate, cleanly-commented queries (or views, if your engine supports them — `CREATE VIEW` is fine and arguably more realistic).

### Panel A — Cohort retention triangle

The full triangle from Exercise 1: `cohort_month`, `cohort_size`, `month_number`, `active_users`, `pct_retained`, for every populated cell.

### Panel B — Growth-accounting trend

The month-by-month new/retained/resurrected/churned/active_total table from Exercise 2, for every calendar month in the dataset. Add one more column: `net_new = active_total − LAG(active_total) OVER (ORDER BY month)`, so the panel also shows whether the *total* active base is growing or shrinking month over month, not just the composition of who's active.

### Panel C — Eligible-adjusted survival curve, by segment

The segment-level survival table from Lecture 3 §2 — `segment`, `month_number`, `eligible_users`, `active_users`, `survival_pct` — correctly excluding cohorts too young to have reached a given `month_number`.

### Panel D — Stickiness trend

The DAU/MAU stickiness table from Exercise 3, for April/May/June, both blended and split by segment (two result sets, or one query with `segment` as a column including a `'TOTAL'` row — your choice, state which).

---

## Part 2 — Interpret each panel (`dashboard_results.md`)

For each of the four panels: paste the actual query output as a markdown table, then write **one sentence** stating what it shows in plain English (not "here's the data" — the actual finding, e.g. "month-1 retention roughly quadrupled from the Feb to the May cohort, consistent with the April onboarding change").

---

## Part 3 — The one-page read (`growth-read.md`)

This is the deliverable that actually gets read by a non-technical stakeholder. In 300–500 words, answer directly: **does Crunch's growth compound, or does it leak?** A strong memo covers:

1. **The headline verdict**, stated in the first two sentences — don't bury it.
2. **The evidence**, drawn from at least three of your four panels, each cited with a specific number (not "retention improved" — "month-1 retention rose from 16.7% to 66.7% across four cohorts").
3. **The caveat that matters most** — where is your evidence weakest? (Small cohort sizes? Cohorts too young to judge? A confound between segment and outcome?) A memo with no caveats is less trustworthy than one that names its own limits.
4. **One concrete recommendation** for what Crunch's growth team should do next, grounded in something you actually found (not a generic "run more experiments" — tie it to the specific segment, cohort era, or metric that stood out).

---

## Milestones

- **Milestone 1 (60 min):** Panels A and B. You've already built both in the exercises — the work here is turning them into clean, presentable, reusable SQL.
- **Milestone 2 (45 min):** Panels C and D. Same idea, pulling from Lecture 3 and Exercise 3.
- **Milestone 3 (30 min):** `dashboard_results.md` — run everything, paste real output, one sentence each.
- **Milestone 4 (45 min):** `growth-read.md` — step away from SQL entirely and write the memo like you're handing it to someone who will never open a terminal.

---

## Rules

- **All four panels must run against the live seed tables** — no hardcoded numbers copied from the lecture notes. If your panel's output doesn't match the lecture's worked examples exactly, you have a bug; find it before moving on.
- **Every number in `growth-read.md` must be traceable to a panel.** If you can't point to which query produced a number in your memo, don't put it in the memo.
- **State your engine.** PostgreSQL and SQLite date syntax differs in a few places (Lecture 1–3 give both); note which you used in `dashboard_results.md`.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|---|---:|---|
| SQL correctness | 35% | All four panels run clean and match the lecture-verified numbers |
| Interpretation (per-panel) | 15% | Each one-sentence read states an actual finding, not a data description |
| The one-page memo | 35% | Clear verdict, specific evidence, honest caveats, one concrete recommendation |
| Caveats and confidence | 15% | Explicitly names where the evidence is weak (small n, censoring, confounds) |

---

## Why this matters

This is the shape of real growth-analytics work: build the queries once, reuse them as a living dashboard, and periodically step back and write the plain-English verdict a non-technical leader can act on. The SQL is necessary but not sufficient — a dashboard nobody can read past is worthless, and a memo with no queries behind it is just an opinion. Keep `dashboard.sql`; Week 5 builds unit economics directly on top of the retention numbers you produce here.

When done: take the [quiz](../quiz.md), then start [Week 5 — LTV, CAC & unit economics](../../week-05-ltv-cac-and-unit-economics/).
