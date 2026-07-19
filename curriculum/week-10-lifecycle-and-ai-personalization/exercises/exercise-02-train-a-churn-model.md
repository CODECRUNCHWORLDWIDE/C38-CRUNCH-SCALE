# Exercise 2 — Train and Evaluate a Churn Model

**Goal:** Train logistic regression and random forest classifiers on Exercise 1's feature table, evaluate both honestly on a held-out test set, and reproduce the lecture's numbers exactly.

**Estimated time:** 90 minutes.

## Setup

`churn_features.csv` from Exercise 1 exists (236 rows). Continue in `solutions_02.py`.

## Tasks

1. **One-hot encode `plan`.** Add `plan_starter` and `plan_growth` binary columns (`Scale` stays the implicit reference level — don't add a third column for it). *(Expected: `df["plan_starter"].sum()` = **116**, `df["plan_growth"].sum()` = **87** — check them against `df["plan"].value_counts()`, which should also show **33** `Scale`.)*

2. **Build `X` and `y`.** Use exactly this feature list, in this order:
   `["mrr_usd", "is_annual", "tenure_days", "events_30d", "events_31_60d", "events_90d", "engagement_trend", "support_tickets_90d", "had_payment_failure", "days_since_last_event", "plan_starter", "plan_growth"]`

3. **Stratified train/test split.** `train_test_split(X, y, test_size=0.30, random_state=42, stratify=y)`. *(Expected: **165** train rows with **61** positives, **71** test rows with **26** positives.)* If your counts differ, check you built `X`/`y` from the exact same dataframe row order as Exercise 1 produced.

4. **Scale and fit logistic regression.** `StandardScaler` fit on **train only**, then `LogisticRegression(max_iter=1000, random_state=42, class_weight="balanced")`. *(Expected test-set ROC-AUC: **0.669** ± 0.01.)*

5. **Fit a random forest.** `RandomForestClassifier(n_estimators=300, max_depth=5, min_samples_leaf=4, random_state=42, class_weight="balanced")` — unscaled features. *(Expected test-set ROC-AUC: **0.575** ± 0.02.)*

6. **Full metrics table.** For both models, report ROC-AUC, PR-AUC (`average_precision_score`), and precision/recall at the default 0.5 threshold. Reproduce this table:

   | Model | ROC-AUC | PR-AUC | Precision@0.5 | Recall@0.5 |
   |---|---:|---:|---:|---:|
   | Logistic Regression | 0.669 | 0.540 | 0.435 | 0.385 |
   | Random Forest | 0.575 | 0.425 | 0.391 | 0.346 |

7. **Confusion matrices.** Print `confusion_matrix(y_test, pred)` for both models at the 0.5 threshold. *(Expected LR: `[[32, 13], [16, 10]]`. Expected RF: `[[31, 14], [17, 9]]`.)* In a one-sentence comment, translate the logistic regression's `FN=16` into a business sentence — what does it mean that this cell is nonzero, and why can't a churn model realistically drive it to zero?

8. **Feature importance / coefficients.** Print the random forest's `feature_importances_` and the logistic regression's `coef_`, both sorted descending. *(Expected top-3 RF importances: `events_90d`, `tenure_days`, `days_since_last_event` — order among the top 3 may vary slightly by scikit-learn version, but these three should dominate.)*

## Done when…

- [ ] Tasks 1–3 match the exact row/positive counts given.
- [ ] Task 4 and 5's ROC-AUC values match within ±0.01–0.02.
- [ ] Task 6's table is fully reproduced with your own numbers alongside the expected ones.
- [ ] Task 7's confusion matrices match, and the `FN=16` sentence is written.

## Stretch

- Try three other classification thresholds (`0.3`, `0.4`, `0.6`) on the logistic regression's test-set probabilities and plot (or tabulate) how precision and recall trade off. At what threshold does recall cross 60%? What happens to precision there?
- Retrain the random forest with `max_depth=None` (unconstrained) and compare test-set ROC-AUC against the `max_depth=5` version. Which is higher, and does that surprise you given Section 6's point about model complexity on small datasets?
- Combine both models into a simple ensemble (`(proba_lr + proba_rf) / 2`) and check whether the blended ROC-AUC beats either model alone.

## Submission

Commit `solutions_02.py` to your portfolio under `c38-week-10/exercise-02/`.
