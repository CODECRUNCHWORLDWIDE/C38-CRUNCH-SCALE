# Challenge 1 — Design an Analytics Mart Schema

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Flow's leadership is happy with this week's `fct_mrr_monthly` and wants you to plan the **rest** of the mart layer — the tables Weeks 2–5 of this course (funnels, activation, retention, LTV/CAC) and Weeks 7–11 (segmentation, experiments, pricing, lifecycle, forecasting) will all eventually need. You are not required to populate every table with real data this week — this challenge is a **schema design**, in real, runnable DDL, with each table's grain, keys, and purpose written down so the next person (a future you, or a teammate) can build against it without guessing.

This is exactly the kind of task a RevOps or analytics-engineering interview asks: "design the warehouse," not "write one query."

## Your task

Produce a `challenge-01.md` containing:

1. **An entity list** — every dimension and fact table you're proposing, one line each: name, one-sentence purpose, and **grain** (the single most important sentence in any table's design — "one row per what?").
2. **Real `CREATE TABLE` DDL** for at least 3 dimensions and 4 facts, including this week's `dim_user`, `dim_date`, and `fct_mrr_monthly` for continuity, plus **new** tables you design, covering at minimum:
   - a `dim_plan` dimension (Starter/Growth/Scale, with list price, so `fct_mrr_monthly` doesn't need to hardcode plan economics in every query),
   - an event-grain fact (`fct_events` or similar) built from `stg_events`, suitable for the funnel/activation work of Weeks 2–3,
   - a **subscription-change** fact (`fct_subscription_changes`) — one row per upgrade/downgrade/cancellation — that a churn or expansion-revenue query could use directly instead of diffing `fct_mrr_monthly` month to month,
   - one fact or dimension of your own choosing that a later week (segmentation, experimentation, pricing, or forecasting — pick one) will plausibly need, with a sentence explaining which week and why.
3. **A grain justification** for each fact table — why that grain, and what question it *cannot* answer that a finer or coarser grain could.
4. **Explicit foreign keys** from every fact to `dim_user` and (where time-based) `dim_date` — this is what "conformed" (Lecture 2) means in practice; a schema that doesn't share dimensions across facts has failed the assignment even if every individual table is well-formed.
5. **One paragraph** on what you deliberately left out of scope for this challenge and why (e.g., "no `fct_support_tickets` yet — no raw source for it this week").

## Constraints

- Every table must have a real primary key (or, for SQLite, an explicit `UNIQUE` constraint standing in for one) and, where applicable, `REFERENCES` foreign keys to `dim_user`/`dim_date`.
- Use the conformed-dimension principle from Lecture 2 — do not invent a second `dim_user`-shaped table under a different name.
- Money is always stored as integer cents, never `FLOAT`/`REAL`, matching this week's convention (a hard rule of this course's data tooling — floating-point currency is a real, well-documented source of production bugs).
- SQL only. No spreadsheet, no diagramming tool required — the DDL itself is the diagram.

## Hints

<details>
<summary>On grain (the hardest part of this challenge)</summary>

"One row per subscription change" (`fct_subscription_changes`) is a genuinely different, and often more useful, grain than "one row per user per month" (`fct_mrr_monthly`) for answering a question like "how much expansion revenue did upgrades generate in Q2?" — that question is a `SUM` over a handful of change-event rows, not a fragile diff between two monthly snapshots. Naming *why* you'd reach for one grain over the other in a specific query is the actual skill this challenge is testing.

</details>

<details>
<summary>On `dim_plan`</summary>

Without it, "what's the list price of Growth" is a fact buried inside every query that touches plans (`CASE plan_name WHEN 'Growth' THEN 9900 ...`, repeated everywhere). With a `dim_plan` table (`plan_name`, `list_price_cents`, `tier_rank`, maybe `feature_limits`), that logic lives in one place, and a pricing change (Week 9!) becomes one `UPDATE`, not a hunt through every query in the codebase.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Grain clarity | "one row per subscription" (ambiguous — per subscription *ever*, or per current state?) | "one row per subscription **change event** — a new row is inserted on every upgrade, downgrade, or cancellation, never updated in place" |
| Conformance | A second, slightly different user table for one fact | Every fact's `user_id` foreign key points at the single `dim_user` |
| Currency handling | `NUMERIC`/`FLOAT` dollars | Integer cents throughout, consistent with this week's convention |
| Forward-thinking | Only builds what's asked | The "your own choosing" table shows real thought about a specific later week's actual query needs |
| Honesty about scope | Pretends the design is complete | Names what's deliberately left out and why |

## Submission

Commit `challenge-01.md` and the DDL (either inline in the markdown or a companion `.sql` file) to your portfolio under `c38-week-06/challenge-01/`.
