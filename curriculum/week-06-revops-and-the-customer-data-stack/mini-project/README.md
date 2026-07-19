# Mini-Project — One Auditable Source of Truth

> Build the complete raw → staging → marts pipeline for Crunch Flow — every table this week's lectures and exercises walked you through, assembled into one idempotent, tested, documented SQL build — so that every later week of this course (segmentation, experiments, pricing, forecasting) has exactly one place to get MRR, users, and events from.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and unlike a typical mini-project, the deliverable isn't a set of answered questions — it's **a running pipeline**. A stakeholder doesn't want twenty numbers today; they want a warehouse they can query for the next twenty numbers themselves, tomorrow, without asking you. That's what you're shipping.

---

## Deliverable

A directory in your portfolio `c38-week-06/mini-project/` containing:

1. **`01_raw_check.sql`** — the four sanity-check row counts from the week README, run and confirmed (paste the actual output as a comment).
2. **`02_staging.sql`** — all four staging models (`stg_users`, `stg_events`, `stg_subscriptions`, `stg_app_subscriptions`) as `CREATE VIEW` statements, plus `stg_subscriptions_current`.
3. **`03_marts.sql`** — `dim_date`, `dim_user`, and `fct_mrr_monthly`, built with the **idempotent truncate-and-reload pattern** from Lecture 3, Section 1, wrapped in `BEGIN`/`COMMIT`.
4. **`04_tests.sql`** — at least **6** data tests (reuse and trim down Challenge 2's suite if you did it, or write fresh ones): one uniqueness test on `fct_mrr_monthly`, one referential-integrity test, one accepted-values test, one range/sanity test, and both cross-source reconciliation tests from Exercise 3.
5. **`metrics.md`** — a written metric-definitions glossary, following the exact shape from Lecture 3, Section 3, covering at minimum **MRR**, **ARR**, and **active customer**.
6. **`report.md`** — the six-month MRR trend table, the three named reconciliation discrepancies with your chosen resolution, and the idempotency proof (see Milestone 4).
7. **`notes.md`** — a short reflection (see the end).

Everything runs against the `crunch_flow` seed database from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used and call out every place your SQL diverges between the two engines.

---

## What you're building, end to end

```
raw_users, raw_events,                stg_users, stg_events,          dim_date, dim_user,
raw_stripe_subscriptions,      ──▶     stg_subscriptions,        ──▶  fct_mrr_monthly
raw_app_subscriptions                 stg_subscriptions_current,
                                       stg_app_subscriptions
```

If a future teammate — or you, in six months — runs `03_marts.sql` twice in a row, `fct_mrr_monthly` must contain **exactly the same 90 rows** both times. If a future teammate runs `04_tests.sql`, every test except the two reconciliation tests must return **zero rows**; the two reconciliation tests must return exactly the three known discrepancies, proving the suite actually works rather than trivially passing.

---

## Milestones

Pace yourself; don't try to do this in one sitting.

- **Milestone 1 (30 min):** Re-verify the raw layer, then build all four `stg_*` views (`02_staging.sql`). Confirm `stg_subscriptions` has 18 rows, not 15 — if you deduped, go back to Lecture 2, Section 4.
- **Milestone 2 (45 min):** Build `dim_date`, `dim_user`, and `fct_mrr_monthly` (`03_marts.sql`) using the idempotent pattern. **Run the whole script twice** and confirm `SELECT COUNT(*) FROM fct_mrr_monthly;` returns 90 both times — this is your idempotency proof, and it goes in `report.md`.
- **Milestone 3 (45 min):** Write and run `04_tests.sql`. Confirm the uniqueness, referential-integrity, accepted-values, and range tests all return 0 rows, and the two reconciliation tests correctly surface the 3 known discrepancies (Ines, Kira, Mira) — not fewer, not more.
- **Milestone 4 (30 min):** Write `metrics.md` (MRR, ARR, active-customer definitions, each naming its source-of-truth table and any known limitation — e.g., the month-end-snapshot gap from Exercise 2, Task 7).
- **Milestone 5 (30 min):** Write `report.md` — the trend table, the reconciliation summary with your Stripe-wins resolution, and the idempotency proof — then `notes.md`.

---

## Rules

- **Everything is SQL.** No spreadsheet, at any point, for any intermediate calculation, cross-check, or "scratch work" — that's this course's hard rule, and this project is exactly the kind of task a spreadsheet tempts you toward. If you catch yourself wanting one, write a `SELECT` instead.
- **The pipeline must be idempotent.** A script that only works correctly the first time it's run is not done. Prove it in `report.md` with the twice-run row count.
- **Stripe is money's source of truth.** Every dollar figure in `fct_mrr_monthly` and `metrics.md` traces back to `raw_stripe_subscriptions`, never to `raw_app_subscriptions` — and `metrics.md` says so explicitly.
- **Every raw table stays untouched.** No `UPDATE`/`DELETE` against `raw_*` tables anywhere in your submission — corrections happen in staging or later, never by editing raw data (Lecture 1, Section 2).
- **Name your columns.** No naked `SELECT *` in anything that ships as part of the pipeline.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Pipeline correctness | 30% | All row counts match exactly (18/15/24/15/90); the May/June MRR figures match ($1,858.17, 12 active) |
| Idempotency | 20% | Script genuinely re-run twice with identical results, proven in `report.md`, not just asserted |
| Data tests | 20% | All 6+ tests present, correctly categorized, and the 2 reconciliation tests genuinely surface the 3 known discrepancies |
| Metric documentation | 15% | `metrics.md` names a source-of-truth table for each metric and states at least one real limitation |
| Reconciliation resolution | 10% | The 3 discrepancies are named, resolved (Stripe wins), and the dollar blast-radius is quantified |
| Readability | 5% | Consistent formatting, meaningful view/table names, comments explaining non-obvious `WHERE` conditions |

---

## Reflection (`notes.md`, ~200 words)

1. Where did the raw data disagree with itself in a way that surprised you, even after Exercise 3 already walked you through three discrepancies?
2. What's one data test you added beyond the required six, and what real mistake would it have caught?
3. If Crunch Flow's engineering team asked you tomorrow to add a fourth source system (say, a CRM), what would change about your staging layer, and what would stay exactly the same?
4. One metric this course will need in a later week (Week 7's segments, Week 8's experiment significance, Week 9's price sensitivity, Week 10's churn score, or Week 11's forecast) that `fct_mrr_monthly` alone can't answer — what additional fact table would you need, and roughly what grain would it have?

---

## Why this matters

Every later week of this course — segmentation, experimentation, pricing, lifecycle AI, forecasting — is going to ask you for a trustworthy number about users and revenue. This mini-project is the moment you stop re-deriving that number from scratch every time and start pointing at a warehouse you built, tested, and can defend. That's the actual job title "RevOps" describes, and it's the foundation everything from Week 7 onward assumes you already have.

When done: push, then take the [quiz](../quiz.md) and start [Week 7 — Segmentation & customer intelligence](../../week-07-segmentation-and-customer-intelligence/).
