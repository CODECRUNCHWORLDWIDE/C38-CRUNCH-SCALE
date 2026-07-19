# Lecture 2 — Personalization and Next-Best-Action

> **Duration:** ~2 hours. **Outcome:** You can turn a churn score into a small, defensible decision table — risk band × customer value → one action per customer — with guardrails that keep expensive human attention off customers a script can save, and stop everyone from getting the same generic nudge.

A score sitting in a dataframe has never saved a single customer. **Next-best-action (NBA)** is the layer that turns "this customer is 71% likely to churn" into "send Priya a calendar invite" or "queue the automated win-back email" — a rule, run against every scored customer, that a support team, a marketing platform, or a script can execute without a human re-deriving the logic every time.

## 1. Score a "production" model on the full population

Lecture 1 validated the model on a held-out test set — that's how you know it's trustworthy. To actually run a lifecycle program, you score **everyone currently active**, so refit the same model (same features, same hyperparameters) on the **full eligible population**, not just the training split:

```python
scaler_full = StandardScaler().fit(X)                 # X = all 236 eligible customers
lr_full = LogisticRegression(max_iter=1000, random_state=42,
                              class_weight="balanced").fit(scaler_full.transform(X), y)
df["churn_score"] = lr_full.predict_proba(scaler_full.transform(X))[:, 1]
```

This is standard practice: validate on a split to estimate real-world performance (Lecture 1's 0.669 ROC-AUC is your honest expectation), then retrain on everything you have before deploying, because more training data almost always helps and you've already measured how good the *method* is. On this seed, `churn_score` across the full 236-customer population comes out:

| Statistic | Value |
|---|---:|
| mean | 0.477 |
| std | 0.203 |
| min | 0.051 |
| 25th pct | 0.313 |
| median | 0.484 |
| 75th pct | 0.640 |
| max | 1.000 |

## 2. From a continuous score to risk bands

A raw probability like `0.612` is precise but not actionable at scale — no team can hand-review 236 individual decimal scores every week. **Risk bands** collapse the score into a small number of buckets a rules engine (or a human) can act on directly:

```python
def risk_band(score):
    if score >= 0.60: return "high"
    if score >= 0.35: return "medium"
    return "low"

df["risk_band"] = df["churn_score"].apply(risk_band)
```

```
high      70
medium    96
low       70
```

Where do `0.60` and `0.35` come from? Not a formula — a **business conversation**. Set the "high" cutoff where the team's intervention capacity runs out (if a CSM team can meaningfully follow up with ~70 accounts a month, `0.60` is the cutoff that produces roughly that many) and the "low" cutoff where you're confident enough to spend zero incremental effort. **Thresholds are a capacity and cost decision dressed up as a modeling decision** — treat them that way, and revisit them whenever team capacity changes, not just when the model retrains.

## 3. Risk alone isn't enough — cross it with value

Two customers can have the identical churn score and deserve completely different treatment. A $299/month Scale customer at 65% risk justifies a CSM's time; a $29/month Starter customer at the same 65% risk does not — the unit economics don't support a phone call, but they easily support an automated email with a modest discount. **Value tier** is the second axis:

```python
def value_tier(mrr):
    return "high_value" if mrr >= 99 else "low_value"

df["value_tier"] = df["mrr_usd"].apply(value_tier)
```

Crossing the two axes:

```python
pd.crosstab(df["risk_band"], df["value_tier"])
```

```
value_tier  high_value  low_value
risk_band
high                12         58
low                 52         10
medium              45         51
```

Notice the shape: only **12 of the 70 high-risk customers are high-value** — most of the urgent-looking risk is concentrated in the cheapest plan, exactly where a human-touch intervention doesn't pencil out. This crosstab, on its own, is often the most useful artifact in the whole exercise: it tells a RevOps leader in one glance how much of "our churn problem" is actually "a handful of expensive accounts" versus "a long tail of cheap ones," which changes the entire intervention strategy.

## 4. The decision table

Six cells, six actions, one per customer — no customer gets more than one action, no cell is left undefined:

| Risk band | Value tier | Action | n (this week) |
|---|---|---|---:|
| High | High-value | **CSM call** — human outreach, understand the specific problem | 12 |
| High | Low-value | **Automated win-back email + discount offer** — can't justify a human, can justify a script | 58 |
| Medium | High-value | **In-app check-in nudge** — surface unused features, no discount (nothing's broken yet) | 45 |
| Medium | Low-value | **Lifecycle nurture email** — low-cost, low-urgency re-engagement drip | 51 |
| Low | High-value | **Expansion/upsell nudge** — healthy and valuable; this is a growth opportunity, not a retention one | 52 |
| Low | Low-value | **No action — standard lifecycle** — don't manufacture urgency where there's none | 18 |

```python
def action(row):
    if row.risk_band == "high" and row.value_tier == "high_value": return "csm_call"
    if row.risk_band == "high" and row.value_tier == "low_value":  return "winback_email_discount"
    if row.risk_band == "medium" and row.value_tier == "high_value": return "inapp_checkin_nudge"
    if row.risk_band == "medium" and row.value_tier == "low_value":  return "lifecycle_nurture_email"
    if row.risk_band == "low" and row.value_tier == "high_value":   return "expansion_upsell_nudge"
    return "no_action_standard_lifecycle"

df["action"] = df.apply(action, axis=1)
```

This is a **rules engine**, and it's deliberately dumb — a lookup table, not a second model. That's a feature, not a limitation: a support lead can read this table in thirty seconds, argue with a specific cell ("should medium-risk, high-value really get a nudge instead of a call?"), and change it without retraining anything. Keep the *scoring* (statistical, opaque, needs a model) and the *decisioning* (business logic, needs to be legible) in separate layers. Conflating them — burying "and therefore call them" inside the model itself — is how you end up with a system nobody can audit or adjust.

## 5. Guardrails: what the decision table doesn't do

A next-best-action table this simple has failure modes you have to design against explicitly, not hope away:

- **One action per customer, per period.** The table above already enforces this structurally (each customer falls in exactly one cell), but if you later add *more* triggers — a product-led "you're close to a milestone" nudge, a billing reminder — you need an explicit priority order or a suppression rule (e.g., "no more than one lifecycle email per customer per 14 days") so customers don't get three uncoordinated messages in the same week. Nothing erodes trust in a "personalized" system faster than obvious incoherence.
- **Don't discount your way into a bad deal.** The `winback_email_discount` action is only cheap because it's automated and targets low-value accounts. If the same discount logic ever got applied to the `csm_call` cell too, you'd be handing a discount to your *most* valuable accounts specifically because they're already the least likely to need one to stay — check every discount rule against "am I paying people who were going to renew anyway?"
- **A wrong low-risk score is a silent failure.** The customers in `no_action_standard_lifecycle` get nothing, by design — but if the model is wrong about one of them, no one finds out until they've already left. This is why Lecture 1's PR-AUC and threshold choice matter: a model that misses too many real churners in the "low" band is quietly costing revenue that never shows up as an obvious bug.
- **Propensity-to-expand deserves the same rigor as propensity-to-churn.** The `expansion_upsell_nudge` cell in this lecture used a placeholder rule (low risk + high value); a real deployment would train a second model — same feature-engineering discipline, same train/test evaluation — specifically for expansion propensity, rather than assuming "not at risk" is the same thing as "ready to grow." Homework extends exactly this idea.

## 6. Check yourself

- Why refit the model on the full 236-customer population instead of just using the test-set predictions for scoring?
- The high-risk × high-value cell has only 12 customers. If your CSM team has capacity for 40 calls a month, would you lower the `0.60` threshold, change the value cutoff, or neither — and why?
- Give a concrete example (not in the lecture) of a rule that would "manufacture urgency where there's none," and why it would backfire.
- Why does the decision table stay a simple lookup instead of becoming a second model that predicts "best action" directly?
- A stakeholder asks why the cheapest, highest-risk customers get an email instead of a phone call — that sounds like giving up on them. How do you answer, using the unit-economics language from Week 5?

If those are automatic, Lecture 3 turns these actions into an actual triggered campaign — and proves, with a holdout, whether it worked.

## Further reading

- **scikit-learn — `predict_proba` and probability calibration:** <https://scikit-learn.org/stable/modules/calibration.html>
- **pandas — `cut`/`qcut` for bucketing continuous scores into bands:** <https://pandas.pydata.org/docs/reference/api/pandas.cut.html>
- **pandas — `crosstab` for two-way segment tables:** <https://pandas.pydata.org/docs/reference/api/pandas.crosstab.html>
