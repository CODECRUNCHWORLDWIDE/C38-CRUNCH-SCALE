# Mini-Project — Score, Personalize, Prove It

> Ship the complete lifecycle-AI pipeline for Crunch Flow: engineer churn features from the warehouse, train and evaluate a model, turn its scores into a next-best-action table, launch a win-back segment with a randomized holdout, and report the measured lift — honestly, including if it isn't statistically significant.

**Estimated time:** 2.5 hours, best done Saturday after the exercises and challenges.

Like Week 6's mini-project, the deliverable here isn't a set of answered questions — it's **a pipeline someone else could run next month with a new cutoff date and get a fresh set of scores and actions out the other end.** That's the difference between "I built a churn model once" and "Crunch Flow has a lifecycle program."

---

## Deliverable

A directory in your portfolio `c38-week-10/mini-project/` containing:

1. **`01_features.sql`** — the full feature-engineering query from Exercise 1, run and confirmed against the `2025-09-01` cutoff (236 rows).
2. **`02_train_model.py`** — the train/test split, both models (logistic regression, random forest), and the full metrics table from Exercise 2, with your own run's numbers printed as comments.
3. **`03_score_and_route.py`** — the production refit on the full population, risk bands, value tiers, and the six-cell next-best-action table from Exercise 3, saved as `churn_scored.csv`.
4. **`04_winback_journey.sql`** — the `winback_segment` query, at least two exclusion rules, and the hash-based holdout split from Challenge 1.
5. **`05_measure_lift.py`** — the outcome simulation, lift calculation, significance test, and power calculation from Challenge 2.
6. **`model_card.md`** — a one-page model card (see Milestone 2) documenting what the model does, doesn't do, and shouldn't be trusted for.
7. **`report.md`** — the full narrative: feature table → model choice and why → NBA table → win-back result → recommendation.
8. **`notes.md`** — a short reflection (see the end).

Everything runs against `crunch_flow_scale.db` (or the equivalent PostgreSQL database) from the [week README](../README.md). Note which engine you used for the SQL portions; the Python/scikit-learn portions are engine-independent.

---

## What you're building, end to end

```
raw_events,              feature table          churn_score,          winback_segment
raw_subscriptions   ──▶  (236 rows,        ──▶   risk_band,      ──▶  (58 rows) ──▶  measured
                         leakage-free)            value_tier,                        lift vs.
                                                   action                             holdout
```

If a future teammate re-ran this pipeline with `CUTOFF = 2025-10-01` instead, every script should work unchanged — the cutoff is a parameter, not a value baked into six different places. Grep your own SQL and Python for the literal string `2025-09-01` before you submit; if it appears more than once or twice as a single named constant, refactor it.

---

## Milestones

Pace yourself; don't try to do this in one sitting.

- **Milestone 1 (35 min):** Re-run the feature query (`01_features.sql`), confirm 236 rows and a 36.9% base rate, then train both models (`02_train_model.py`). Confirm your ROC-AUC values land within ±0.02 of the lecture's (LR 0.669, RF 0.575).
- **Milestone 2 (30 min):** Write `model_card.md` — a compact document covering: what the model predicts (the exact cutoff/window definition from Lecture 1, Section 1), what data it was trained on, its validated performance (ROC-AUC, PR-AUC, precision/recall at 0.5), at least one known limitation (e.g., "trained on 236 customers — expect wider variance on any single new customer than these aggregate metrics suggest"), and one thing it should explicitly **not** be used for (e.g., "not validated for enterprise/Scale-tier customers specifically — only 33 of 236 training examples are Scale").
- **Milestone 3 (30 min):** Score the full population and build the NBA table (`03_score_and_route.py`). Confirm the exact action counts (12/58/45/51/52/18).
- **Milestone 4 (35 min):** Build the win-back segment and holdout split (`04_winback_journey.sql`), then run the outcome simulation and lift measurement (`05_measure_lift.py`). Confirm the 30/28 split and the 57.1%/43.3% churn rates.
- **Milestone 5 (30 min):** Write `report.md` and `notes.md`.

---

## Rules

- **Everything is SQL and pandas/scikit-learn.** No spreadsheet, at any point — not for the feature table, not for "eyeballing" the model's scores, not for tracking the NBA action counts. That's this course's hard rule.
- **The cutoff is a named parameter**, used consistently across every SQL query and every Python script — not a magic string repeated six different ways.
- **Report the model card honestly.** A model card that only lists strengths isn't a model card — Milestone 2 explicitly requires at least one real limitation and one explicit non-use-case.
- **Report the lift honestly, including if it's not significant.** `report.md`'s recommendation section must state the p-value and the power-calculation result, not just the headline "13.8-point lift" number in isolation.
- **Every raw table stays untouched.** No `UPDATE`/`DELETE` against `raw_*` — corrections and derived work happen downstream.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Feature engineering correctness | 20% | 236 eligible rows, 36.9% base rate, zero leakage (no `event_ts >= cutoff` rows anywhere in feature SQL) |
| Model training + evaluation | 20% | Both models trained correctly; ROC-AUC/PR-AUC/precision/recall all reported and within tolerance of the lecture's numbers |
| Model card | 15% | Names the exact prediction task, reports real metrics, states a genuine limitation and a genuine non-use-case |
| NBA table | 15% | Exact action counts (12/58/45/51/52/18); decision logic matches Lecture 2's table |
| Win-back segment + holdout | 15% | 58-row segment, correct 30/28 split, at least 2 documented exclusion rules |
| Lift measurement + honesty | 15% | Correct lift/p-value/power numbers; `report.md` doesn't overstate a non-significant result |

---

## Reflection (`notes.md`, ~200 words)

1. Where in this pipeline would leakage be easiest to introduce by accident, and what specifically would you check to catch it before shipping?
2. The random forest lost to logistic regression this week. If Crunch Flow had 50,000 customers instead of 236, would you expect that result to hold? Why or why not?
3. The win-back test came back directionally positive but not statistically significant. If your manager asked you to "just say it worked" in a slide for the board, what would you say instead, and why does it matter?
4. Name one feature this week's model *doesn't* have access to (something not in `raw_events`/`raw_subscriptions`) that you'd want for a better model, and where in a real company's stack it would likely come from.

---

## Why this matters

A model that never leaves a notebook has never saved a single customer. This mini-project is the whole chain — features, model, decision table, triggered campaign, honest measurement — that turns "we have data" into "we have a lifecycle program that provably moves a number." Every later week of this course (forecasting, the capstone) assumes you can build this chain again, for a different question, without being walked through it.

When done: push, then take the [quiz](../quiz.md) and start Week 11 — Forecasting growth.
