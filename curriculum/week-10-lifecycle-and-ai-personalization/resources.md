# Week 10 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up. This week deliberately uses **plain scikit-learn and plain SQL/pandas** — no proprietary ML platform, no paid feature store, nothing this week's deliverables depend on beyond an open-source Python install.

## Required reading (this week's core)

- **scikit-learn — `train_test_split` and cross-validation:** <https://scikit-learn.org/stable/modules/cross_validation.html>
  *Why: the exact split discipline (`stratify`, `random_state`) Lecture 1 and Exercise 2 depend on.*
- **scikit-learn — Logistic Regression:** <https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression>
  *Why: this week's primary, winning model — understand `class_weight="balanced"` and what standardized coefficients mean.*
- **scikit-learn — Random Forest Classifier:** <https://scikit-learn.org/stable/modules/ensemble.html#random-forests>
  *Why: the comparison model, and the source of `feature_importances_` used in Lecture 1, Section 7.*
- **scikit-learn — Model evaluation guide (ROC-AUC, PR-AUC, precision/recall, confusion matrix):** <https://scikit-learn.org/stable/modules/model_evaluation.html>
  *Why: the single most important reference this week — read the imbalanced-classification sections closely.*
- **statsmodels — Two-proportion z-test:** <https://www.statsmodels.org/stable/generated/statsmodels.stats.proportion.proportions_ztest.html>
  *Why: exactly the test Lecture 3 and Challenge 2 run to check whether the win-back lift is real.*
- **statsmodels — Power and sample-size calculations:** <https://www.statsmodels.org/stable/stats.html#power-and-sample-size-calculations>
  *Why: the `NormalIndPower`/`proportion_effectsize` machinery behind the "you need ~205 per group" result.*

## Concept references (the ideas behind the code)

- **Google Cloud — "What is a machine learning model?" and the classification crash course:** <https://developers.google.com/machine-learning/crash-course>
  *Why: free, thorough treatment of the classification/evaluation concepts this week assumes — a good primer if scikit-learn's API feels unfamiliar.*
- **Google — "ROC and AUC" (Machine Learning Crash Course):** <https://developers.google.com/machine-learning/crash-course/classification/roc-and-auc>
  *Why: the clearest free visual explanation of what ROC-AUC actually measures.*
- **Evidently AI — "12 Important Model Evaluation Metrics for Machine Learning":** <https://www.evidentlyai.com/classification-metrics>
  *Why: a practical, tool-agnostic walkthrough of precision/recall/PR-AUC tradeoffs on imbalanced data, matching this week's churn-modeling context directly.*

## Growth/RevOps background on churn, lifecycle, and holdouts

- **Amplitude — "What Is Churn Rate? How to Calculate and Reduce It":** <https://amplitude.com/blog/churn-rate>
  *Why: industry vocabulary for churn definitions (voluntary/involuntary, logo vs. revenue) referenced in Lecture 1, Section 1.*
- **Reforge / OpenView — lifecycle marketing and next-best-action frameworks (search "next best action SaaS"):** general search — no single canonical free link, but useful as background reading once you've built this week's version yourself.
- **Optimizely — "A/B Testing Statistics: An Intuitive Guide":** <https://www.optimizely.com/optimization-glossary/statistical-significance/>
  *Why: reinforces Week 8's significance concepts in the specific context of measuring a live campaign, exactly what Lecture 3/Challenge 2 do.*

## Practice beyond the seed database

- **Kaggle — Telco Customer Churn dataset:** <https://www.kaggle.com/datasets/blastchar/telco-customer-churn>
  *Why: a well-known, free, real-shaped churn dataset — good for practicing the same feature-engineering-then-model workflow on data you didn't generate yourself.*
- **scikit-learn — User Guide, "Choosing the right estimator" map:** <https://scikit-learn.org/stable/machine_learning_map.html>
  *Why: once you're comfortable with logistic regression and random forests, this map is the standard reference for what else to reach for and when.*

## Glossary

| Term | Definition |
|------|------------|
| **Cutoff date** | The "as of" moment features are computed from — everything after it is off-limits while building features. |
| **Label window** | The future period a model predicts over (e.g., "will they churn in the next 4 months"). |
| **Leakage** | A feature (or the label itself, disguised) containing information that wouldn't actually be available at prediction time. |
| **Base rate** | The proportion of positive labels in a dataset — the floor a model has to beat to be worth anything. |
| **ROC-AUC** | The probability a model ranks a random positive example above a random negative one; 0.5 = random, 1.0 = perfect. |
| **PR-AUC (average precision)** | Area under the precision-recall curve; more informative than ROC-AUC on imbalanced data. |
| **`class_weight="balanced"`** | A training option that upweights the minority class so the model doesn't just learn to predict the majority class. |
| **Risk band** | A discretized bucket (e.g., high/medium/low) collapsing a continuous score into an actionable group. |
| **Value tier** | A customer-value bucket (e.g., by MRR) crossed with risk to decide the right intervention. |
| **Next-best-action (NBA)** | A rules-engine lookup mapping a customer's state (score × value, typically) to exactly one action. |
| **Trigger-based journey** | A lifecycle campaign fired by a state change (e.g., entering a risk band), not a calendar date. |
| **Randomized holdout** | A group deliberately excluded from an intervention, assigned *before* any outcome is known, used to measure the intervention's true causal effect. |
| **Lift** | The difference in outcome rate between a treatment group and its holdout — absolute (percentage points) or relative (%). |
| **Statistical power** | The probability a test correctly detects a real effect of a given size, given its sample size. |
| **Model card** | A short, honest document stating what a model predicts, how it was validated, and its known limitations and non-use-cases. |

---

*Broken link? Open an issue or PR.*
