# Lecture 1 — Churn and Propensity Models

> **Duration:** ~2 hours. **Outcome:** You can define a churn label precisely (cutoff + label window), engineer leakage-free features from a warehouse in SQL, train a scikit-learn classifier, and evaluate it with the metrics that actually matter when the output drives an action.

A churn model earns its keep the moment someone acts on its output — a CSM calls the right account, an email fires to the right inbox, a discount lands on the right invoice. Everything before that moment is plumbing. This lecture builds the plumbing correctly: a precise label, features that don't cheat, a model you can explain, and metrics that tell the truth about whether it's worth deploying.

## 1. Churn is a definition, not a fact

"Did this customer churn?" sounds like a yes/no fact sitting in the database. It isn't — it's a **modeling decision** with at least three parts you have to nail down before you write a single line of feature-engineering SQL:

1. **What counts as churn.** For a subscription business, the obvious answer is "`raw_subscriptions.status = 'canceled'`." But real churn definitions get more specific: voluntary vs. involuntary (payment failure vs. a deliberate cancel), full cancellation vs. downgrade, and whether a 3-day free trial abandonment counts the same as a 2-year customer leaving. This week, **churn = the subscription's `status` becomes `'canceled'`** — the simplest honest definition, and the one Crunch Flow's warehouse can answer without ambiguity.

2. **As of when.** A model doesn't predict "will this customer ever churn" — it predicts "will this customer churn **in some specific future window**, given what we know about them **right now**." "Right now" is the **cutoff date**. This week, `CUTOFF = 2025-09-01` — Crunch Flow's "today."

3. **Over what window.** The **label window** is how far into the future you're predicting. Too short (7 days) and you'll barely catch anyone; too long (2 years) and the prediction is nearly useless for *this month's* campaign planning, and unrelated future events (a new competitor, a re-org) dilute the signal. This week, the label window is **`CUTOFF` to `LABEL_END` — four months, September through December 2025.**

Put together: *"Will this customer, active as of September 1st, cancel their subscription by December 31st?"* That's a precise, answerable, SQL-expressible question. "Will this customer churn?" is not.

## 2. The cardinal sin: leakage

**Leakage** is when a feature contains information that wouldn't have been available at the moment you'd actually need the prediction — in practice, information from *after* the cutoff, or information that's really just a disguised copy of the label.

Two leakage traps that catch almost everyone the first time:

- **Time leakage.** If you compute "events in the last 30 days" using `raw_events` rows with no date filter, you'll pull in October and November activity for a customer you're supposedly scoring on September 1st — activity that includes the very engagement drop-off caused *by* their decision to leave. The model will look great in backtesting and be useless in production, because in production you'll never have next month's data when you need this month's score. **Rule: every feature query filters on `event_ts < CUTOFF`. No exceptions, ever.**
- **Target leakage.** A feature like "days until cancellation" or "customer received a win-back email" (if win-back emails are triggered *by* risk) is really the label wearing a disguise. If a feature is suspiciously perfect, ask: *could I have computed this without already knowing the answer?*

Crunch Flow's warehouse makes this easy to get right, because `raw_events` is an immutable, timestamped log — every feature query below filters `WHERE event_ts < DATE('2025-09-01')`, full stop.

## 3. Engineering features from the warehouse, in SQL

Five feature families cover most of what a churn model needs, and all five come straight out of the event log and subscription table you seeded:

| Family | Example features | What it captures |
|---|---|---|
| **Tenure** | `tenure_days` (signup → cutoff) | Newer customers haven't found their footing; very old customers have proven stickiness |
| **Recency** | `days_since_last_event` | Engagement gone quiet is the single strongest churn signal in most SaaS products |
| **Frequency / trend** | `events_30d`, `events_31_60d`, `events_90d`, `engagement_trend` | Not just "how active" but "active and *rising* or *falling*" |
| **Friction** | `support_tickets_90d`, `had_payment_failure` | Support load and billing pain are leading indicators, not just symptoms |
| **Account shape** | `plan`, `mrr_usd`, `is_annual` | Commitment level and price point predict churn independently of behavior |

Here's the full feature query, built entirely from `raw_events` and `raw_subscriptions`, strictly filtered to `event_ts < CUTOFF`:

```sql
WITH cutoff AS (SELECT DATE('2025-09-01') AS d),

-- population: customers still active as of the cutoff. Anyone already
-- canceled before the cutoff is excluded — they're not "at risk," they're gone.
eligible AS (
    SELECT s.user_id, s.plan, s.mrr_usd, s.billing_interval, s.start_date, s.canceled_at
    FROM raw_subscriptions s, cutoff
    WHERE s.canceled_at IS NULL OR s.canceled_at > cutoff.d
),

-- events strictly BEFORE the cutoff — this WHERE clause is the leakage guard
ev AS (
    SELECT e.*,
           CAST(julianday((SELECT d FROM cutoff)) - julianday(e.event_ts) AS INTEGER) AS days_before_cutoff
    FROM raw_events e, cutoff
    WHERE e.event_ts < cutoff.d
)

SELECT
    el.user_id,
    el.plan,
    el.mrr_usd,
    CASE WHEN el.billing_interval = 'year' THEN 1 ELSE 0 END AS is_annual,
    CAST(julianday((SELECT d FROM cutoff)) - julianday(el.start_date) AS INTEGER) AS tenure_days,
    COALESCE(SUM(CASE WHEN ev.event_name = 'feature_used' AND ev.days_before_cutoff <= 30
                       THEN 1 ELSE 0 END), 0) AS events_30d,
    COALESCE(SUM(CASE WHEN ev.event_name = 'feature_used' AND ev.days_before_cutoff > 30
                       AND ev.days_before_cutoff <= 60 THEN 1 ELSE 0 END), 0) AS events_31_60d,
    COALESCE(SUM(CASE WHEN ev.event_name = 'feature_used' AND ev.days_before_cutoff <= 90
                       THEN 1 ELSE 0 END), 0) AS events_90d,
    COALESCE(SUM(CASE WHEN ev.event_name = 'support_ticket_opened' AND ev.days_before_cutoff <= 90
                       THEN 1 ELSE 0 END), 0) AS support_tickets_90d,
    COALESCE(MAX(CASE WHEN ev.event_name = 'payment_failed' THEN 1 ELSE 0 END), 0) AS had_payment_failure,
    COALESCE(MIN(ev.days_before_cutoff), 999) AS days_since_last_event
FROM eligible el
LEFT JOIN ev ON ev.user_id = el.user_id
GROUP BY el.user_id, el.plan, el.mrr_usd, el.billing_interval, el.start_date, el.canceled_at;
```

Run it — you should get **236 rows**, one per eligible customer. Two things worth noticing in the shape of this query:

- The `LEFT JOIN` from `eligible` to `ev` means a customer with **zero** pre-cutoff events still gets a row, with `events_30d = 0` etc. — `COALESCE` turns `SUM`'s `NULL` (nothing to sum) into `0`. Two brand-new customers in this dataset (signed up 31–46 days before cutoff) have genuinely no events yet, and `days_since_last_event` falls back to a sentinel `999` — read that as "no activity on record," not "active 999 days ago." Left uninterpreted, a raw `999` would badly mislead a distance-based model; a tree-based model handles it fine as "an extreme, distinct value," but you should always know *why* a sentinel exists before you ship it.
- `engagement_trend` isn't in the SQL — it's `events_30d - events_31_60d`, a derived feature it's just as easy to compute in pandas after loading the query result. Some features belong in SQL (anything aggregating raw rows); some belong in pandas (arithmetic on already-aggregated columns). Neither is "more correct" — pick whichever keeps the SQL readable.

## 4. The label, joined separately

The label lives in a **separate query**, on purpose — it's tempting to fold it into the feature query, but keeping them apart forces you to think about the label window as its own decision, not an afterthought:

```sql
SELECT user_id,
       CASE WHEN canceled_at IS NOT NULL
                 AND canceled_at > '2025-09-01'
                 AND canceled_at <= '2025-12-31'
            THEN 1 ELSE 0 END AS churned_in_window
FROM raw_subscriptions
WHERE canceled_at IS NULL OR canceled_at > '2025-09-01';
```

Join features to label on `user_id` in pandas:

```python
import pandas as pd, sqlite3
conn = sqlite3.connect("crunch_flow_scale.db")
feat = pd.read_sql(FEATURE_SQL, conn)
label = pd.read_sql(LABEL_SQL, conn)
df = feat.merge(label, on="user_id")
df["engagement_trend"] = df["events_30d"] - df["events_31_60d"]
```

Against this week's seed: **236 eligible customers, 87 churn during the label window — a 36.9% base rate.** That's your baseline: a model that predicts "everyone churns" gets 36.9% precision and 100% recall on the positive class; a model that predicts "no one churns" gets 100% accuracy and is worthless. Neither number alone tells you if a model is good — which is exactly why Section 6 doesn't lead with accuracy.

## 5. Training the model

One-hot encode `plan` (drop one category as the reference level), split train/test, and fit two models — a linear baseline and a tree ensemble — so you have something to compare:

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier

df["plan_starter"] = (df["plan"] == "Starter").astype(int)
df["plan_growth"] = (df["plan"] == "Growth").astype(int)   # Scale is the reference level

feature_cols = ["mrr_usd", "is_annual", "tenure_days", "events_30d", "events_31_60d",
                 "events_90d", "engagement_trend", "support_tickets_90d",
                 "had_payment_failure", "days_since_last_event", "plan_starter", "plan_growth"]
X, y = df[feature_cols], df["churned_in_window"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.30, random_state=42, stratify=y
)
```

`stratify=y` matters here: with a 36.9% positive rate and only 236 rows, an unstratified split can easily land a test set with 25% or 48% positives by chance, making your evaluation noisy for no reason. `random_state=42` makes the split — and therefore every metric below — exactly reproducible.

```python
scaler = StandardScaler().fit(X_train)
logreg = LogisticRegression(max_iter=1000, random_state=42, class_weight="balanced")
logreg.fit(scaler.transform(X_train), y_train)
proba_lr = logreg.predict_proba(scaler.transform(X_test))[:, 1]

rf = RandomForestClassifier(n_estimators=300, max_depth=5, min_samples_leaf=4,
                             random_state=42, class_weight="balanced")
rf.fit(X_train, y_train)
proba_rf = rf.predict_proba(X_test)[:, 1]
```

`class_weight="balanced"` upweights the minority (churned) class during training so the model doesn't just learn "predict 0, you'll be right most of the time" — worth doing by default on any imbalanced churn/fraud/default problem. `StandardScaler` matters for logistic regression (its coefficients are only comparable across features on the same scale) and is harmless-but-unnecessary for the random forest (tree splits don't care about scale) — fit it anyway for consistency, or skip it for the forest; either is fine.

## 6. Evaluating with the metrics that matter

Split sizes on this seed: **165 training rows (61 positive), 71 test rows (26 positive)**. Run both models through the same evaluation:

```python
from sklearn.metrics import (roc_auc_score, average_precision_score,
                              precision_score, recall_score, f1_score, confusion_matrix)

for name, proba in [("Logistic Regression", proba_lr), ("Random Forest", proba_rf)]:
    pred = (proba >= 0.5).astype(int)
    print(name,
          "ROC-AUC:", round(roc_auc_score(y_test, proba), 3),
          "PR-AUC:", round(average_precision_score(y_test, proba), 3),
          "precision:", round(precision_score(y_test, pred), 3),
          "recall:", round(recall_score(y_test, pred), 3))
```

Actual results on this seed:

| Model | ROC-AUC | PR-AUC | Precision@0.5 | Recall@0.5 | Confusion matrix (TN, FP / FN, TP) |
|---|---:|---:|---:|---:|---|
| Logistic Regression | **0.669** | **0.540** | 0.435 | 0.385 | 32, 13 / 16, 10 |
| Random Forest | 0.575 | 0.425 | 0.391 | 0.346 | 31, 14 / 17, 9 |

Four things to take from this table, in order of importance:

1. **The simpler model won.** Logistic regression beat the random forest on every metric here. This is a genuine, common outcome on small tabular datasets (236 rows is *small* for a tree ensemble to find real structure in) — "more complex model" is not a strategy, it's a hyperparameter to test like any other. Ship the one that validates better, and default to the more interpretable one when they're close.
2. **Why ROC-AUC and PR-AUC, not accuracy.** ROC-AUC (0.5 = random guessing, 1.0 = perfect ranking) measures how well the model *ranks* churners above non-churners across every possible threshold — exactly what you need when the next step is "sort by risk and act on the top N." **PR-AUC matters more on imbalanced data**: with a 36.5% positive test rate, a PR-AUC of 0.540 (vs. a 0.365 baseline-if-random) tells you precision holds up reasonably well as you push recall up — accuracy would have stayed near 60–65% for a model that predicts almost nothing, and told you nothing useful.
3. **Precision/recall at 0.5 is a business choice, not a law.** At the default 0.5 cutoff, logistic regression flags 23 of 71 customers and gets 10 of them right (43.5% precision) while catching 10 of 26 actual churners (38.5% recall). Moving the threshold trades one for the other — Lecture 2 sets the threshold based on what an intervention *costs*, not on 0.5 by default.
4. **A 0.669 AUC is not "good" in the abstract — it's good *relative to what a next-best-action system needs*.** You are not trying to predict the future perfectly; you're trying to rank 236 customers well enough that the top of the list is meaningfully riskier than the bottom, so a limited pool of CSM time and discount budget goes to the right accounts. Section 4 of Lecture 2 shows exactly how much better than random this ranking is where it counts.

## 7. Reading feature importance (and being skeptical of it)

```python
import pandas as pd
pd.Series(rf.feature_importances_, index=feature_cols).sort_values(ascending=False).round(3)
```

On this seed, the random forest ranks `events_90d`, `tenure_days`, `days_since_last_event`, `mrr_usd`, and `engagement_trend` as the top five drivers — recency and frequency dominate, exactly matching the "engagement gone quiet" intuition from Section 3. Two cautions before you present feature importance to a stakeholder as causal fact:

- **Importance is not causation.** `mrr_usd` ranking highly could mean "cheap plans churn more" (true here, by design) or could be a proxy for something else entirely in a real dataset. Correlational features are still useful for *scoring* — you don't need to know *why* a customer is at risk to act on the fact that they are — but be careful before you tell a product team "raise prices to cut churn" off a feature-importance chart alone.
- **Logistic regression coefficients are more interpretable, and worth checking too.** On standardized features, `days_since_last_event` (+0.90), `plan_starter` (+0.41), and `had_payment_failure` (+0.30) push risk up; `events_90d` (−0.61) and `mrr_usd` (−0.38) pull it down. That sign pattern is a sanity check as much as an insight: if `tenure_days` had come out with a strongly *positive* coefficient (longer-tenured customers riskier), that would be a signal to go looking for a bug before trusting the model at all.

## 8. Check yourself

- What are the two things a churn label needs beyond "did they cancel" — and why does a model trained without them fail silently in production?
- Give an example of time leakage in this week's dataset that isn't already in the lecture, and the SQL fix.
- Why does `stratify=y` matter more here than it would on a dataset of 100,000 rows with a 50/50 class balance?
- The random forest lost to logistic regression on every metric. Does that mean random forests are bad for churn modeling? What would make you reach for one anyway?
- A colleague reports "our churn model gets 91% accuracy." Base rate is 8% churn. Is that impressive? What would you ask for instead?

If those are automatic, Lecture 2 turns this score into something a team can act on.

## Further reading

- **scikit-learn — Logistic Regression:** <https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression>
- **scikit-learn — Random Forest Classifier:** <https://scikit-learn.org/stable/modules/ensemble.html#random-forests>
- **scikit-learn — Model evaluation (ROC-AUC, PR-AUC, precision/recall):** <https://scikit-learn.org/stable/modules/model_evaluation.html>
- **PostgreSQL — Date/time functions (for `julianday`-equivalent logic, e.g. `EXTRACT(EPOCH FROM ...)`):** <https://www.postgresql.org/docs/current/functions-datetime.html>
