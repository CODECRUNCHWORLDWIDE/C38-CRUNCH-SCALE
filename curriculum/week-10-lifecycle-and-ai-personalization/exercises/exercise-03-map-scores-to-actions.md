# Exercise 3 — Map Scores to Next-Best-Actions

**Goal:** Retrain the winning model on the full eligible population, score every customer, and build the risk × value next-best-action table from Lecture 2 — reproducing its exact counts.

**Estimated time:** 60 minutes.

## Setup

Exercise 2's `X`, `y`, and `feature_cols` exist. Continue in `solutions_03.py`.

## Tasks

1. **Pick the production model.** Exercise 2 showed logistic regression beating the random forest on every metric. Refit a `StandardScaler` and `LogisticRegression(max_iter=1000, random_state=42, class_weight="balanced")` on the **full** `X`/`y` (all 236 rows, no train/test split this time) and store `df["churn_score"]`. *(Expected: mean **0.477**, std **0.203**, min **0.051**, max **1.000**.)*

2. **Risk bands.** Apply the `risk_band` function from Lecture 2, Section 2 (`>= 0.60` high, `>= 0.35` medium, else low). *(Expected counts: high **70**, medium **96**, low **70**.)*

3. **Value tiers.** Apply `value_tier` (`>= 99` high-value, else low-value) to `mrr_usd`. *(Expected counts: run `df["value_tier"].value_counts()` and confirm the totals reconcile with the crosstab in Task 4.)*

4. **The crosstab.** Run `pd.crosstab(df["risk_band"], df["value_tier"])`. *(Expected exactly:)*

   | risk_band | high_value | low_value |
   |---|---:|---:|
   | high | 12 | 58 |
   | medium | 45 | 51 |
   | low | 52 | 18 |

5. **The action table.** Apply the `action` function from Lecture 2, Section 4. *(Expected counts: `csm_call` **12**, `winback_email_discount` **58**, `inapp_checkin_nudge` **45**, `lifecycle_nurture_email` **51**, `expansion_upsell_nudge` **52**, `no_action_standard_lifecycle` **18**. These six numbers must sum to **236**.)*

6. **Sanity-check one customer by hand.** Pick any single `user_id` from your dataframe. Manually trace its `churn_score` → `risk_band` → `value_tier` → `action` and confirm your code's output matches your by-hand trace. Write the trace as a one-line comment.

7. **Where's the risk concentrated?** Using the crosstab from Task 4, compute what fraction of `mrr_usd`-weighted revenue sits in the `high` risk band vs. the `low` risk band (i.e., `SUM(mrr_usd)` for each band, not just headcount). Compare that to the headcount split (70/96/70) — does revenue-at-risk look proportionally worse, better, or about the same as headcount-at-risk?

## Done when…

- [ ] Task 1's score distribution matches within ±0.01 on each statistic.
- [ ] Task 2's band counts are exactly 70/96/70.
- [ ] Task 4's crosstab matches exactly, cell for cell.
- [ ] Task 5's six action counts match exactly and sum to 236.
- [ ] Task 6's hand-trace is written and correct.
- [ ] Task 7 reports a concrete revenue-at-risk number, with one sentence of interpretation.

## Stretch

- Build a second next-best-action table using `0.50`/`0.30` band cutoffs instead of `0.60`/`0.35` and compare how many customers move cells — specifically, how many move from `no_action_standard_lifecycle` into an active-outreach action, and is your intervention capacity ready for that many more?
- For the 12 `csm_call` customers, pull their `support_tickets_90d` and `had_payment_failure` values and draft (in a comment, not code) one sentence a CSM could use to open each call — grounded in that specific customer's data, not a generic template.
- Persist the final scored table as `scored_customers` in your database (`df.to_sql(...)` or a `CREATE TABLE` + bulk insert) — Challenge 1 queries it directly.

## Submission

Commit `solutions_03.py` and the resulting `churn_scored.csv` to your portfolio under `c38-week-10/exercise-03/`.
