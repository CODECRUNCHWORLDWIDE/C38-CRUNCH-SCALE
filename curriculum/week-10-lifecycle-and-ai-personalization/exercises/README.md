# Week 10 — Exercises

Three guided exercises, ~60–120 min each. **Type every query and every line of Python yourself** — don't copy-paste out of the lectures, even when a task builds directly on a worked example there. A model you typed once sticks; a model you pasted doesn't.

1. **[Exercise 1 — Engineer churn features from events](exercise-01-engineer-churn-features.md)** — build the leakage-free feature table from `raw_events`/`raw_subscriptions` as of the cutoff.
2. **[Exercise 2 — Train and evaluate a churn model](exercise-02-train-a-churn-model.md)** — logistic regression + random forest, real train/test evaluation.
3. **[Exercise 3 — Map scores to next-best-actions](exercise-03-map-scores-to-actions.md)** — score the full population and build the risk × value decision table.

## Before you start

- You've completed all three lectures.
- You ran `seed_crunch_flow_scale.py` from the [week README](../README.md) and it printed `260 users, 9095 events.`
- You have a shell open (`psql crunch_flow_scale` or `sqlite3 crunch_flow_scale.db`) **and** a Python environment with `pandas` and `scikit-learn` installed (`pip install pandas scikit-learn`).

## Suggested workflow

- Open the exercise file beside your terminal/editor.
- Save every query and every model-training step into a single script per exercise — `solutions_01.sql`, `solutions_02.py`, `solutions_03.py` — so later exercises and the mini-project can reload your work instead of re-deriving it.
- For each task, **write the query/code before running it**, then check the output against the "Expected" note. This week's expected numbers come from the *exact* seed script in the README — if your numbers don't match, the bug is almost always an off-by-one in a date filter, not a modeling mistake.
- If a metric surprises you (a suspiciously perfect score, a wildly different AUC), stop and figure out why before moving on — in churn modeling, the surprising result is usually leakage.

## A note on reproducibility

Every `random_state=42` and every `np.random.seed(42)` in this week's materials is there for a reason: with them, your numbers match this document's exactly. Without them (or with a different scikit-learn/NumPy version doing something unusual under the hood), you may see small deviations — a percentage point or two on a metric is normal; a wildly different result means something upstream diverged.
